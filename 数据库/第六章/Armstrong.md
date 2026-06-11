

# Armstrong 公理系统

本节主线是：用 **Armstrong 公理系统** 从已知函数依赖 $F$ 推导新的函数依赖，并进一步解决三个考试核心问题：

1. 如何判断 $F$ 是否蕴涵某个函数依赖 $X \rightarrow Y$；
2. 如何计算属性集闭包 $X_F^+$；
3. 如何判断函数依赖集等价、覆盖，并求极小函数依赖集。

---

## 逻辑蕴涵

> 若关系模式 $R(U,F)$ 中，任何满足 $F$ 的关系 $r$ 都满足函数依赖 $X \rightarrow Y$，则称 $F$ 逻辑蕴涵 $X \rightarrow Y$。

$$
F \models X \rightarrow Y
$$

> 通俗理解：只要 $F$ 成立，$X \rightarrow Y$ 就必然成立。

> 考试注意：逻辑蕴涵强调的是“所有满足 $F$ 的关系实例”都满足该依赖，不是某一个具体表偶然满足。

---

## Armstrong 公理系统

> Armstrong 公理系统用于从已知函数依赖推出新的函数依赖，是函数依赖推理、码计算、模式分解算法的基础。

Armstrong 三条基本规则：

1. **自反律**；
2. **增广律**；
3. **传递律**。

> 通俗理解：这三条规则是函数依赖推理的“基本推理公理”。

> 考试注意：Armstrong 公理系统是有效且完备的，后续闭包、覆盖、极小依赖集都依赖它。

---

## 自反律

> 若 $Y \subseteq X \subseteq U$，则 $X \rightarrow Y$ 被 $F$ 蕴涵。

$$
Y \subseteq X \subseteq U \Rightarrow X \rightarrow Y
$$

> 通俗理解：一个属性集一定能决定自己的任意子集。

> 易错点：自反律推出的都是**平凡函数依赖**，不依赖于 $F$ 中具体有哪些函数依赖。

---

## 增广律

> 若 $X \rightarrow Y$ 被 $F$ 蕴涵，且 $Z \subseteq U$，则 $XZ \rightarrow YZ$ 被 $F$ 蕴涵。

$$
X \rightarrow Y \Rightarrow XZ \rightarrow YZ
$$

> 通俗理解：函数依赖两边同时加上同一组属性，依赖仍然成立。

> 易错点：增广律是“两边同时加”，不能只给右部加属性。

---

## 传递律

> 若 $X \rightarrow Y$ 且 $Y \rightarrow Z$ 被 $F$ 蕴涵，则 $X \rightarrow Z$ 被 $F$ 蕴涵。

$$
X \rightarrow Y,\ Y \rightarrow Z \Rightarrow X \rightarrow Z
$$

> 通俗理解：如果 $X$ 决定 $Y$，$Y$ 又决定 $Z$，那么 $X$ 可以间接决定 $Z$。

> 易错点：用 Armstrong 传递律推出的函数依赖，不一定就是规范化理论中的“传递函数依赖”。

---

## 合并规则

> 若 $X \rightarrow Y$ 且 $X \rightarrow Z$，则 $X \rightarrow YZ$。

$$
X \rightarrow Y,\ X \rightarrow Z \Rightarrow X \rightarrow YZ
$$

> 通俗理解：同一个决定因素能分别决定多个属性，就能一次性决定它们的并集。

> 易错点：合并规则要求左部相同；若左部不同，不能直接合并。

---

## 分解规则

> 若 $X \rightarrow Y$ 且 $Z \subseteq Y$，则 $X \rightarrow Z$。

$$
X \rightarrow Y,\ Z \subseteq Y \Rightarrow X \rightarrow Z
$$

> 通俗理解：能决定一组属性，就能决定这组属性中的任意一部分。

> 易错点：分解的是右部，不是随意删左部属性。

---

## 伪传递规则

> 若 $X \rightarrow Y$ 且 $WY \rightarrow Z$，则 $WX \rightarrow Z$。

$$
X \rightarrow Y,\ WY \rightarrow Z \Rightarrow WX \rightarrow Z
$$

> 通俗理解：如果 $X$ 能提供 $Y$，而 $WY$ 能决定 $Z$，那么 $WX$ 也能决定 $Z$。

> 易错点：伪传递不是简单的 $X \rightarrow Y,\ Y \rightarrow Z$，它允许在第二个依赖左部附加公共属性 $W$。

---

## 引理 6.1：右部分解与合并

> $X \rightarrow Y_1Y_2\cdots Y_n$ 成立，当且仅当每个 $X \rightarrow Y_i$ 都成立。

$$
X \rightarrow Y_1Y_2\cdots Y_n \Leftrightarrow X \rightarrow Y_i,\ i=1,2,\cdots,n
$$

> 通俗理解：右部多个属性的函数依赖，可以拆成多个右部单属性依赖；反过来也可以合并。

> 考试注意：求极小函数依赖集时，第一步通常就是把右部拆成单属性。

---

## 函数依赖集的闭包

> 在关系模式 $R(U,F)$ 中，被 $F$ 逻辑蕴涵的所有函数依赖的集合，称为 $F$ 的闭包，记为 $F^+$。

$$
F^+=\lbrace X \rightarrow Y \mid F \models X \rightarrow Y\rbrace
$$

> 通俗理解：$F^+$ 是从 $F$ 出发能推出的全部函数依赖。

> 易错点：$F^+$ 通常非常大，直接求 $F^+$ 是 NP 问题，考试中更常用属性集闭包来判断某个依赖是否成立。

---

## Armstrong 公理系统的有效性与完备性

> Armstrong 公理系统是有效且完备的。

有效性：

$$
F \vdash X \rightarrow Y \Rightarrow F \models X \rightarrow Y
$$

完备性：

$$
F \models X \rightarrow Y \Rightarrow F \vdash X \rightarrow Y
$$

> 通俗理解：“能用公理推出来”与“被逻辑蕴涵”是等价的。

> 考试注意：有效性保证推理不会推出错误结论；完备性保证所有正确结论都能被 Armstrong 公理推出。

---

## 属性集闭包

> 设 $F$ 是属性集 $U$ 上的一组函数依赖，$X \subseteq U$，属性集 $X$ 关于 $F$ 的闭包 $X_F^+$ 是所有能由 $X$ 推出的属性组成的集合。

$$
X_F^+=\lbrace A \mid A \in U \land F \models X \rightarrow A\rbrace
$$

> 通俗理解：$X_F^+$ 表示从属性集 $X$ 出发，在 $F$ 的作用下最多能推出哪些属性。

> 考试注意：判断 $F \models X \rightarrow Y$，不必求 $F^+$，只需要判断：

$$
Y \subseteq X_F^+
$$

---

## 引理 6.2：用属性集闭包判定函数依赖

> $X \rightarrow Y$ 能由 $F$ 根据 Armstrong 公理导出的充分必要条件是 $Y \subseteq X_F^+$。

$$
F \models X \rightarrow Y \Leftrightarrow Y \subseteq X_F^+
$$

> 通俗理解：只要 $Y$ 中的所有属性都能从 $X$ 推出来，$X \rightarrow Y$ 就成立。

> 考试注意：这是判断函数依赖是否成立的核心方法，尤其常用于证明两个函数依赖集是否等价。

---

## 算法 6.1：属性集闭包的计算

> 输入函数依赖集 $F$ 和属性集 $X$，输出属性集闭包 $X_F^+$。

1. **初始化**：令 $X^{(0)}=X$，$i=0$；
2. **扫描依赖集**：在 $F$ 中寻找满足 $A \rightarrow B$ 且 $A \subseteq X^{(i)}$ 的函数依赖；
3. **扩展闭包**：把所有可推出的右部属性加入当前集合，得到 $X^{(i+1)}$；
4. **判断是否停止**：若 $X^{(i+1)}=X^{(i)}$，或 $X^{(i)}=U$，则停止；
5. **继续迭代**：否则令 $i=i+1$，返回扫描步骤；
6. **输出结果**：最终得到的 $X^{(i)}$ 即为 $X_F^+$。

> 一句话概括：不断用“左部已包含于当前闭包”的函数依赖扩展属性集，直到不能再增加新属性。

> 易错点：每一轮都要扫描整个 $F$，不是只用一次函数依赖；只要新属性加入闭包，后续可能触发更多依赖。

---

## 函数依赖集的覆盖

> 设 $F$ 和 $G$ 是关系模式 $R$ 上的两个函数依赖集，如果 $F$ 逻辑蕴涵 $G$ 中所有函数依赖，则称 $F$ 覆盖 $G$。

$$
G \subseteq F^+
$$

> 通俗理解：$F$ 能推出 $G$ 中的所有依赖，就说 $F$ 覆盖 $G$。

> 考试注意：证明 $F$ 覆盖 $G$ 时，通常逐个检查 $G$ 中的每个依赖是否能由 $F$ 推出。

---

## 函数依赖集的等价

> 若两个函数依赖集 $F$ 和 $G$ 的闭包相等，则称 $F$ 与 $G$ 等价。

$$
F^+=G^+
$$

> 通俗理解：虽然 $F$ 和 $G$ 表面上写法不同，但它们能推出的全部函数依赖完全相同。

> 考试注意：等价不是 $F=G$，而是 $F^+=G^+$。

---

## 引理 6.3：函数依赖集等价判定

> $F$ 与 $G$ 等价的充分必要条件是二者相互覆盖。

$$
F^+=G^+ \Leftrightarrow F \subseteq G^+ \land G \subseteq F^+
$$

> 通俗理解：$F$ 能推出 $G$，$G$ 也能推出 $F$，二者就等价。

> 考试注意：证明等价时一般分两步：
>
> 1. 证明 $F$ 覆盖 $G$；
> 2. 证明 $G$ 覆盖 $F$。

---

## 极小函数依赖集

> 若函数依赖集 $F$ 满足以下三个条件，则称 $F$ 为极小函数依赖集，也称最小依赖集或最小覆盖。

1. $F$ 中每个函数依赖的右部只含一个属性；
2. $F$ 中不存在冗余函数依赖；
3. $F$ 中不存在左部冗余属性。

形式化理解：

$$
X \rightarrow A
$$

其中右部 $A$ 必须是单属性。

> 通俗理解：极小函数依赖集就是在保持等价的前提下，把函数依赖集化简到“不能再删、不能再缩”的形式。

> 易错点：极小函数依赖集不一定唯一，处理依赖和属性的顺序不同，可能得到不同但等价的最小覆盖。

---

## 极小函数依赖集的三个判定条件

> 判断 $F$ 是否为极小函数依赖集，需要检查右部、冗余依赖和左部冗余属性。

### 条件一：右部单属性

$$
X \rightarrow A
$$

> 每个函数依赖右部只能有一个属性。

### 条件二：无冗余函数依赖

> 不存在某个 $X \rightarrow A \in F$，使得删除它后仍与原函数依赖集等价。

$$
F-\lbrace X \rightarrow A\rbrace \equiv F
$$

### 条件三：无左部冗余属性

> 不存在 $X \rightarrow A$ 中的某个属性 $B \in X$，使得用 $(X-B)\rightarrow A$ 替换后仍与原函数依赖集等价。

$$
F-\lbrace X \rightarrow A\rbrace \cup \lbrace (X-B)\rightarrow A\rbrace \equiv F
$$

> 通俗理解：右部要拆单，整条依赖不能多余，左部属性也不能多余。

---

## 定理 6.3：最小依赖集存在性

> 每一个函数依赖集 $F$ 都等价于某个极小函数依赖集 $F_m$。

$$
F \equiv F_m
$$

> 通俗理解：任何函数依赖集都可以通过等价变换化简成一个最小覆盖。

> 考试注意：最小覆盖一定存在，但不一定唯一。

---

## 极小函数依赖集计算算法

> 输入函数依赖集 $F$，输出与 $F$ 等价的极小函数依赖集 $G$。

1. **初始化**：令 $G=F$；

2. **右部单属性化**：将每个形如 $X \rightarrow A_1A_2\cdots A_n$ 的依赖替换为：

   $$
   X \rightarrow A_1,\ X \rightarrow A_2,\ \cdots,\ X \rightarrow A_n
   $$

3. **消除左部冗余属性**：对每个 $X \rightarrow A$，检查 $X$ 中每个属性 $B$；

   若：

   $$
   A \in (X-B)_G^+
   $$

   则用：

   $$
   (X-B)\rightarrow A
   $$

   替换原来的：

   $$
   X \rightarrow A
   $$

4. **消除冗余函数依赖**：对每个 $X \rightarrow A$，令：

   $$
   H=G-\lbrace X \rightarrow A\rbrace
   $$

   若：

   $$
   A \in X_H^+
   $$

   则从 $G$ 中删除 $X \rightarrow A$；

5. **合并同左部依赖**：若存在：

   $$
   X \rightarrow A_1,\ X \rightarrow A_2,\ \cdots,\ X \rightarrow A_n
   $$

   可合并为：

   $$
   X \rightarrow A_1A_2\cdots A_n
   $$

6. **输出结果**：得到极小函数依赖集 $G$。

> 一句话概括：先拆右部，再删左部多余属性，再删整条多余依赖，最后可合并同左部依赖。

> 易错点：第 3 步和第 4 步的检查顺序可以调整，但最终必须保证没有左部冗余属性，也没有冗余函数依赖。

---


## 本章高频考试题型总结

> 本课件最后的复习思考题集中指向以下题型。

1. **写出 Armstrong 三条基本规则**：自反律、增广律、传递律；
2. **证明扩充规则**：合并规则、分解规则、伪传递规则；
3. **判断函数依赖推导是否成立**：用 Armstrong 公理或属性集闭包；
4. **计算属性集闭包**：按算法反复扩展；
5. **判断函数依赖集等价**：证明相互覆盖；
6. **求极小函数依赖集**：拆右部、删左部冗余、删冗余依赖；
7. **说明极小函数依赖集不唯一**：不同处理顺序可能得到不同最小覆盖。

> 复习重点：属性集闭包 $X_F^+$ 是本节最核心的工具，既能判断函数依赖是否成立，也能判断覆盖、等价和极小覆盖。
