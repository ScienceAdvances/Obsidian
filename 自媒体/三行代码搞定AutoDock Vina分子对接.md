分子对接是药物发现和蛋白研究中的核心步骤，但传统流程中，从分子下载、软件配置到结果分析，往往需要繁琐的操作和大量参数调试，让新手望而却步。

今天就给大家分享一套极简方案——**用3行核心代码搞定基于AutoDock Vina的分子对接**，甚至还能借助Gnina的CNN评分提升精度！无需手动点击，全程命令行操作，小白也能快速上手。

## 为什么选择Gnina？
安装代码：https://github.com/gnina/gnina
视频教程：https://www.youtube.com/watch?v=MG3Srzi5kZ0
Slides PPT: https://bits.csb.pitt.edu/rsc_workshop2021/docking_with_gnina.slides.html#/
示例代码：https://colab.research.google.com/drive/1QYo5QLUE80N_G28PlpYs6OKGddhhd931
示例代码：https://colab.research.google.com/drive/1GXmk1v8C-c4UtyKFqIm9HnsrVYH0pI-c
在介绍代码前，先简单说下工具：

**Gnina** 是基于AutoDock Vina开发的分子对接工具，它保留了Vina的轻量、快速特性，还加入了**卷积神经网络（CNN）评分函数**，能更精准地预测蛋白-配体结合亲和力，尤其适合虚拟筛选和结合模式预测。

我做过简单的测试，用共结晶的配体和蛋白使用AutoDock Vina做分子对接和Gnina的全蛋白对接结果差不多，所以省去了找box步骤便可以批量做分子对接，从而达到虚拟筛选的作用
 

下面直接上干货！整套流程包含**配体/蛋白下载、分子对接、结果汇总**，核心代码仅3行，全程自动化。

  
## 准备工作

在运行前，需要准备两个文件（格式示例如下）：

- `Ligand.list`：配体ID列表（第一列名称，第二列PubChem的CID）

```shell
ligand1 12345
ligand2 67890
```

- `Receptor.list`：蛋白ID列表（第一列名称，第二列RCSB的PDB ID）

```shell
protein1 1ABC
protein2 2DEF
```

同时，确保安装了必要工具：`gnina`、`python`、`pdbfix`、`xargs`（Linux/macOS自带，Windows可通过WSL或Cygwin使用）。

## 第1行：批量下载配体（PubChem）

从PubChem数据库批量下载配体的SDF文件，存到`Ligands`文件夹：

```bash
cut -f2 Ligand.list | xargs -P3 -I {} -n1 python src/get_pubchem.py --cid {} --output Ligands/{}.sdf
```

- **解析**：

- `cut -f2 Ligand.list`：提取列表中第二列的PubChem CID（配体唯一标识）；

- `xargs -P3`：用3个进程并行下载，加速获取；

- `get_pubchem.py`：辅助脚本（可自行编写，功能是调用PubChem API下载SDF格式配体）。

  
  

### 第2行：批量下载蛋白（RCSB PDB）

从RCSB PDB数据库下载蛋白的PDB文件，存到`Receptors`文件夹：

```bash

cut -f2 Receptor.list | xargs -P2 -I {} -n1 python src/get_pdb.py -i {} -o Receptors

```

- **解析**：

- 类似配体下载逻辑，提取PDB ID后并行下载蛋白结构；

- `get_pdb.py`：辅助脚本（调用RCSB API下载PDB文件）。

  
  

### 第3行：批量运行分子对接（Gnina）

这是核心步骤！用Gnina批量处理所有蛋白-配体组合，自动完成对接并输出结果：

```bash

cat Receptor.Ligand | xargs -n2 -P1 bash -c '

gnina \

--receptor "Receptors/$1.pdb" \

--ligand "Ligands/$2.sdf" \

--autobox_ligand "Receptors/$1.pdb" \

--out DockResult/"$1--$2.sdf" \

--cpu 32 \

--scoring vina \

--seed 1 \

--cnn_verbose \

--exhaustiveness 256 \

--num_modes 50 \

--device 0 &> "logs/$1--$2.log"

' bash

```

- **关键参数解析**：

- `--receptor`/`--ligand`：指定蛋白和配体文件；

- `--autobox_ligand`：根据蛋白自动生成对接盒子（无需手动设置中心和大小，新手友好！）；

- `--scoring vina`：使用Vina经典评分函数（也可加`--cnn`启用CNN评分）；

- `--exhaustiveness 256`：搜索强度（值越大结果越可靠，耗时略增）；

- `--device 0`：若有GPU，指定显卡加速（无GPU可省略，自动用CPU）。

  
  

## 一步生成对接总结报告

对接完成后，用一行代码汇总所有结果（结合能、排名等），输出CSV表格：

```bash

python3 src/get_score.py --dockdir DockResult --ligand Ligand.list --receptor Receptor.list --output DockSummary.csv

```

打开`DockSummary.csv`，就能直观看到每个蛋白-配体组合的对接分数，轻松筛选最优候选！

  
  

## 新手小贴士

1. **并行参数**：`xargs -P`后的数字（如`-P3`）代表并行进程数，根据电脑CPU核心数调整（别超过核心数，避免卡顿）；

2. **AutoDock Vina兼容**：若想用原生Vina，只需把`gnina`换成`vina`，参数基本通用；

3. **盒子调整**：若`--autobox_ligand`生成的盒子不合适，可手动用`--center_x` `--center_y` `--center_z`和`--size_x` `--size_y` `--size_z`指定；

4. **可视化**：用PyMOL或ChimeraX打开`DockResult`中的SDF文件，可直观查看对接构象。

  
  

用这三行代码，从分子下载到结果分析全流程自动化，再也不用对着复杂界面点点点！无论是新手练手还是批量虚拟筛选，都能高效搞定。

  

赶紧收藏起来试试吧，有任何问题欢迎在评论区交流～