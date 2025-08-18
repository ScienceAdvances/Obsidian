结构数据格式（SDF）由分子设计有限公司（MDL）开发。SDF 文件是简单的 ASCII 文本文件，遵循严格的格式标准，用于表示多个化学结构记录及相关数据字段。我们从PubChem下载的化合物结构一般是sdf格式，Gnina的分子对接结果也是sdf格式。可以使用pymol等工具可视化。

下边以阿司匹林的sdf文件为例，介绍下sdf格式
![image.png](https://s2.loli.net/2025/08/15/Xh8K59tnaDpl4Zz.png)


从包含多个分子对接结果的结构sdf文件中提取能量最低的构象。下边的代码Ls.sdf是分子对接结果，-O L.sdf输出文件
结构数据格式（SDF）由分子设计有限公司（MDL）开发。SDF 文件（也称为 SD 文件）是简单的 ASCII 文本文件，遵循严格的格式标准，用于表示多个化学结构记录及相关数据字段。
[三行代码搞定AutoDock Vina批量分子对接](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/MRXdxKEOU9E4F9E__fWkXg)

- `-f 26`：从第 26 个分子开始处理（`f` = first）。
- `-l 26`：到第 26 个分子结束处理（`l` = last）。  
    两者结合表示：**仅处理输入文件中序号为 26 的单个分子**（SDF 文件中分子按顺序编号，从 1 开始）
```shell
obabel Ls.sdf -f 26 -l 26 -O L.sdf
```

## **一、标题块（Header Block）**

SDF文件开头的前两行通常为标题信息，用于快速标识化合物或文件来源：

```C
2244
-OEChem-08142521593D

```

- 第一行“2244”：通常是化合物的唯一标识符（如本例中对应`PUBCHEM_COMPOUND_CID`，即PubChem数据库的化合物ID），方便后续检索和关联。
- 第二行“-OEChem-08142521593D”：生成该文件的软件信息（OEChem是常用的化学信息学工具包）和时间戳（08142521593D表示生成时间），用于追溯文件来源和生成环境。

### **二、分子结构数据块（Molecular Structure Block）**

这部分是SDF文件的核心，包含分子的原子、键的具体信息，格式严格遵循V2000或V3000标准（本例为V2000）。
### **1. 分子结构头部行**

```C
21 21 0 0 0 0 0 0 0999 V2000
```

这一行是分子结构的总览，各数字含义如下：

- `21`：分子中原子的总数（后续会有21行原子信息）。
- `21`：分子中化学键的总数（后续会有21行键信息）。
- `0`：分子的电荷数（0表示电中性）。
- 后续多个`0`：分别表示自旋多重度、同位素数量、立体化学标记等（本例均为0，说明无特殊标记）。
- `999`：表该分子无特定的结晶学信息（如晶胞参数）。
- `V2000`：标识分子结构数据采用的格式版本（V2000是常用的旧版本，结构简单；V3000支持更复杂的分子）。
### **2. 原子信息行（共21行，对应21个原子）**

每行代表一个原子的详细信息，以第5行为例：

```C
-0.0857 0.6088 0.4403 C 0 0 0 0 0 0 0 0 0 0 0 0
```

各字段含义（从左到右）：
- C：元素符号（此处为碳原子），其他行如O表示氧原子，`H`表示氢原子。
- 后续12个`0`：分别表示原子的电荷（0为中性）、同位素标记（0为天然同位素）、立体化学构型（0为无特殊构型）、原子类型等（本例均为0，说明无特殊属性）。
- `1.2333 0.5540 0.7792`：原子的三维坐标（单位通常为埃Å），分别对应X、Y、Z轴，用于确定原子在空间中的位置。因此可以根据是否有z轴信息判断出是否是3D结构，如果不是可以使用openbabel或者rdkit生成3D结构

- `openbabel`
```shell
obabel 2D.sdf -O 3D.sdf --gen3d -addH -p 7.4
```

- `rdkit`
```python
from rdkit import Chem
from rdkit.Chem import AllChem

m1 = Chem.SDMolSupplier('2D.sdf')[0]
m2 = Chem.AddHs(m1, addCoords=True)
AllChem.EmbedMolecule(m2, randomSeed=1, enforceChirality=True)
AllChem.UFFOptimizeMolecule(m2)

with Chem.SDWriter('3D.sdf') as writer:
    writer.write(m2)
```


### **3. 键信息行（共21行，对应21个化学键）**

每行代表一个化学键的信息，以第一行为例：

```C
1 5 1 0 0 0 0
```

各字段含义（从左到右）：

- `1`：化学键连接的第一个原子编号（对应原子信息行的序号，第1个原子是氧原子）。
- `5`：化学键连接的第二个原子编号（第5个原子是碳原子）。
- `1`：键类型（1=单键，2=双键，3=三键，4=芳香键；本例中“1”表示单键）。
- 后续4个`0`：表示键的立体化学信息（如顺反异构）、共轭性等（0表示无特殊标记）。

再如第6行键信息：

```C
4 12 2 0 0 0 0
```

表示第4个原子（氧原子）与第12个原子（碳原子）之间是双键（键类型为2）。

#### **4. 分子结构结束标记**

```C
M END
```

这一行标志“分子结构数据块”的结束，后续内容为分子的属性信息。

## **三、属性数据块（Data Block）**

这部分以“`> <标签名>`”开头，记录分子的附加属性（如数据库ID、物理化学性质、计算参数等），每个属性由“标签+值”组成。

### **1. 基础标识属性**

```C
> <PUBCHEM_COMPOUND_CID>
2244
```

- 标签`PUBCHEM_COMPOUND_CID`：表示该化合物在PubChem数据库中的唯一ID（与标题块的“2244”对应），用于跨平台检索。

### **2. 构象相关属性**
- RMSD
标签`PUBCHEM_CONFORMER_RMSD`：表示分子构象的均方根偏差（单位通常为埃），0.6说明该构象与参考构象的偏差较小，结构稳定。
```C
> <PUBCHEM_CONFORMER_RMSD>
0.6
```

- DIVERSEORDER
标签`PUBCHEM_CONFORMER_DIVERSEORDER`：记录分子不同构象的多样性排序，用于构象筛选（如药物设计中优先选择多样性高的构象）

```C
> <PUBCHEM_CONFORMER_DIVERSEORDER>
1
11
10
...（省略部分内容）

```

### **3. 电荷与能量属性**
标签`PUBCHEM_MMFF94_PARTIAL_CHARGES`：表示用MMFF94力场计算的原子部分电荷（如第1个原子带-0.23电荷），用于分析分子极性和反应性

```C
> <PUBCHEM_MMFF94_PARTIAL_CHARGES>
18
1 -0.23
10 -0.15
...（省略部分内容）

```

标签`PUBCHEM_MMFF94_ENERGY`：用MMFF94力场计算的分子总能量（单位通常为kcal/mol），能量越低说明构象越稳定。
```C
> <PUBCHEM_MMFF94_ENERGY>
39.5952
```

### **4. 结构与活性属性**

标签`PUBCHEM_PHARMACOPHORE_FEATURES`：记录分子的药效团特征（如“acceptor”表示氢键受体），用于药物设计中预测分子与靶点的结合能力。

```C
> <PUBCHEM_PHARMACOPHORE_FEATURES>
5
1 2 acceptor
1 3 acceptor
...（省略部分内容）

```

标签`PUBCHEM_HEAVY_ATOM_COUNT`：表示分子中重原子（除H外的原子，如C、O）的总数，13个重原子说明分子属于中等大小的有机分子。

```C
> <PUBCHEM_HEAVY_ATOM_COUNT>
13
```

### **5. 其他属性**

还包括立体化学计数（如`PUBCHEM_ATOM_DEF_STEREO_COUNT`表示明确的立体中心数量）、转子数量（`PUBCHEM_EFFECTIVE_ROTOR_COUNT`表示可旋转键数量，影响分子柔性）、形状指纹（`PUBCHEM_SHAPE_FINGERPRINT`用于分子形状相似性比较）等，均为药物研发、化学分析中的关键参数。

## **四、文件结束符/分割符**
这是SDF文件的终止标志，表示整个文件结束。如果一个SDF文件包含多个分子（如化合物库），则每个分子结束后会有一个`$$$$`，并紧跟下一个分子的标题块。

```C
$$$$
```

## Reference
```C
https://pubchem.ncbi.nlm.nih.gov/compound/2244
https://en.wikipedia.org/wiki/Chemical_table_file#SDF
```