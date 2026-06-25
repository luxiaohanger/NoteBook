## 总览

| 考点     | 核心作用                 | 关键内容                                     |
| ------ | -------------------- | ---------------------------------------- |
| inode  | 保存文件元数据              | 权限、大小、时间、链接计数、数据块指针等                     |
| 目录项    | 把文件名映射到 inode        | Unix 目录项通常是 $\langle 文件名, inode号\rangle$ |
| 文件读写定位 | 由 inode 把文件偏移量映射到磁盘块 | $offset \rightarrow block$               |
| 多级索引   | 支持不同大小文件             | 直接指针、一级间接、二级间接、三级间接                      |
| 容量计算   | 计算 inode 能索引多大文件     | 块大小、指针大小、每块可存指针数                         |

---

## inode

inode，即 index node，是 Unix 类文件系统中保存文件元数据的结构。文件名本身通常不保存在 inode 中，而是保存在目录项中；目录负责把文件名映射到 inode 号。inode 负责记录文件属性以及文件数据块的位置。

inode 中常见信息包括：

| 字段                  | 含义               |
| ------------------- | ---------------- |
| `mode`              | 文件类型与访问权限        |
| `uid/gid`           | 文件所有者和所属组        |
| `size`              | 文件大小             |
| `atime/mtime/ctime` | 访问、修改、状态改变时间     |
| `links_count`       | 硬链接计数            |
| `blocks`            | 已分配的数据块数量        |
| `block pointers`    | 指向文件数据块或间接索引块的指针 |

文件访问的核心映射是：

$$
\text{path name} \rightarrow \text{inode number} \rightarrow \text{inode} \rightarrow \text{data blocks}
$$

> inode 是文件的“身份证和索引表”，记录文件是谁、能不能访问、大小多少、数据放在哪。

> 文件名不等于文件本体；硬链接之所以成立，就是因为多个目录项可以指向同一个 inode。

---

## 目录项与 inode 的关系

Unix 中目录本身也是一种文件，它的数据内容是一组目录项。每个目录项通常保存一个文件名和对应 inode 号：

$$
\langle \text{file name},\ \text{inode number} \rangle
$$

例如：

```text
.         1952
..        6253
tmp       224
test.txt  1976
main.c    4594
```

当访问 `/usr/ast/mbox` 时，系统会逐级查目录：

```text
/        中找到 usr  -> inode 6
/usr     中找到 ast  -> inode 26
/usr/ast 中找到 mbox -> inode 60
```

得到目标 inode 后，才能根据 inode 中的数据块索引读取文件内容。

> 目录负责“按名字找 inode”，inode 负责“按偏移找数据块”。

> inode 通常不保存文件名；同一个 inode 可以有多个文件名，这就是硬链接的基础。

---

## inode 如何支持文件读写定位

用户程序读写时给的是文件描述符和字节数，内核实际需要把“文件内偏移量”转换成“磁盘块号”。

假设块大小为 $B$，当前文件偏移量为 $offset$，则文件逻辑块号为：

$$
logical_block = \left\lfloor \frac{offset}{B} \right\rfloor
$$

块内偏移为：

$$
inner_offset = offset \bmod B
$$

然后通过 inode 中的索引结构找到该逻辑块对应的物理磁盘块：

$$
logical_block \rightarrow physical_block
$$

读写流程可以简化为：

```text
fd
 │
 ▼
打开文件表项：取得 offset 和 inode 指针
 │
 ▼
inode：根据 offset 找到对应数据块
 │
 ▼
读取 / 写入磁盘块
 │
 ▼
更新 offset
```

> 文件偏移量是“文件里的位置”，inode 索引把它翻译成“磁盘上的位置”。

> `lseek()` 只改 offset，不直接访问磁盘；真正读写发生在 `read()` / `write()`。

---

## 多级索引

多级索引是 inode 中用于记录文件数据块位置的结构。它的目标是同时兼顾小文件和大文件：小文件直接用 inode 内的直接指针即可，大文件再通过间接指针扩展索引能力。

典型 inode 索引结构包括：

| 指针类型                   | 指向内容              | 作用      |
| ---------------------- | ----------------- | ------- |
| 直接指针 Direct Pointer    | 数据块               | 快速访问小文件 |
| 一级间接指针 Single Indirect | 一个“指针块”，其中每项指向数据块 | 支持较大文件  |
| 二级间接指针 Double Indirect | 指向一级指针块的指针块       | 支持更大文件  |
| 三级间接指针 Triple Indirect | 指向二级指针块的指针块       | 支持超大文件  |

结构示意：

```text
inode
 ├── direct[0]  ─────────────► data block
 ├── direct[1]  ─────────────► data block
 ├── ...
 ├── single indirect ────────► pointer block ───► data blocks
 ├── double indirect ────────► pointer block ───► pointer blocks ───► data blocks
 └── triple indirect ────────► pointer block ───► pointer blocks ───► pointer blocks ───► data blocks
```

> 直接指针像“目录第一页直接写地址”，间接指针像“先翻到地址目录，再找真正的数据块”。

> 多级索引不是为了让所有访问都更快，而是为了在 inode 大小固定的情况下支持大小差异很大的文件。

---

## 多级索引容量计算

假设：

$$
\text{数据块大小} = 4KB = 2^{12}B
$$

$$
\text{一个指针大小} = 4B
$$

则一个索引块能保存的指针数为：

$$
\frac{4KB}{4B} = \frac{2^{12}}{2^2} = 2^{10} = 1024
$$

若 inode 中有 $15$ 个指针，其中前 $12$ 个是直接指针，后 $3$ 个分别是一级、二级、三级间接指针，则可索引容量如下。

| 索引方式     |                             可指向的数据块数 |                     可索引容量 |
| -------- | -----------------------------------: | ------------------------: |
| 12 个直接指针 |                                 $12$ |    $12 \times 4KB = 48KB$ |
| 一级间接指针   |                             $2^{10}$ | $2^{10} \times 4KB = 4MB$ |
| 二级间接指针   |               $2^{10} \times 2^{10}$ | $2^{20} \times 4KB = 4GB$ |
| 三级间接指针   | $2^{10} \times 2^{10} \times 2^{10}$ | $2^{30} \times 4KB = 4TB$ |

总容量近似为：

$$
12 \times 4KB + 2^{10} \times 4KB + 2^{20} \times 4KB + 2^{30} \times 4KB
$$

其中最大部分由三级间接索引决定，所以最大文件大小约为：

$$
4TB
$$

> 计算题关键先算“一个索引块能装多少个指针”。

> 不要把“索引块大小”和“数据块大小”混淆；索引块里存的是指针，数据块里存的是文件内容。

---

## 访问代价对比

不同索引层级访问数据块时，可能需要额外读取索引块。

| 访问位置   | 额外读取的索引块数 | 说明               |
| ------ | --------: | ---------------- |
| 直接指针范围 |       $0$ | inode 中直接得到数据块地址 |
| 一级间接范围 |       $1$ | 先读一级索引块，再读数据块    |
| 二级间接范围 |       $2$ | 先读二级索引块，再读一级索引块  |
| 三级间接范围 |       $3$ | 需要逐层读索引块         |

例如访问一个由二级间接指针管理的数据块：

```text
inode
 │
 ▼
double indirect block
 │
 ▼
single indirect block
 │
 ▼
data block
```

> 索引层级越深，随机访问时可能需要更多 I/O。

> 实际系统会缓存 inode 和索引块，所以不是每次都真的访问磁盘。
