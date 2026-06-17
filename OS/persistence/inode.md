## 目录 inode
> 目录就是文件夹，也是一种特殊的文件
> **目录中保存的是“文件名 → inode 号”的映射；真正的 inode 存在于文件系统专门的 inode 区 / inode 表中。**

可以这样理解：

```text
目录文件的数据块
├── a.txt  → inode 100
├── b.txt  → inode 205
└── dir1   → inode 300
```

而真正的 inode 结构在别的地方：

```text
inode 表
├── inode 100：类型、权限、大小、时间戳、数据块指针...
├── inode 205：类型、权限、大小、时间戳、数据块指针...
└── inode 300：类型=目录、权限、大小、数据块指针...
```

---

## 目录里有什么？

目录文件里主要存的是**目录项 directory entry**，也就是：

```text
文件名 + inode 号
```

例如：

```text
a.txt → 100
```

这里的 `100` 是 inode 编号，不是完整 inode 本身。

---

## inode 本身在哪里？

在类 Unix 文件系统中，inode 通常存放在文件系统的 inode 表中。

磁盘结构可以粗略理解为：

```text
文件系统
├── 超级块 superblock
├── inode 位图
├── 数据块位图
├── inode 表
│    ├── inode 100
│    ├── inode 101
│    └── inode 102
└── 数据块区
     ├── 普通文件内容
     └── 目录文件内容
```

所以：

```text
目录项 只保存 inode 号
inode 表 保存 inode 具体内容
数据块 保存文件内容或目录项内容
```

---

## 路径访问过程

例如访问：

```text
/home/user/a.txt
```

大致过程是：

```text
根目录 inode
  ↓
读取根目录的数据块，找到 home → inode 号
  ↓
读取 home 的 inode
  ↓
读取 home 目录的数据块，找到 user → inode 号
  ↓
读取 user 的 inode
  ↓
读取 user 目录的数据块，找到 a.txt → inode 号
  ↓
读取 a.txt 的 inode
  ↓
根据 a.txt 的 inode 找到数据块
```

目录负责“根据名字找到 inode 号”，inode 负责“描述文件并定位数据块”。

---

## 硬链接也能说明这一点

假设：

```text
a.txt → inode 100
b.txt → inode 100
```

这两个文件名都在目录项中，但它们指向同一个 inode。

这说明：

> 文件名可以有多个，inode 只有一个；目录里只是保存指向 inode 的编号。

---

## 考试记忆版

> inode 不存在于目录中，而是存放在文件系统的 inode 表中。目录文件的数据块保存目录项，即“文件名到 inode 号”的映射。根据目录项得到 inode 号后，系统再到 inode 表中读取对应 inode，进而获得文件元数据和数据块地址。
