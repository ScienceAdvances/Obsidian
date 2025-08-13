分子对接是药物发现和蛋白研究中的核心步骤，但传统流程中，从分子下载、软件配置到结果分析，往往需要繁琐的操作和大量参数调试，让新手望而却步。

今天就给大家分享一套极简方案——**用3行核心代码搞定基于AutoDock Vina的分子对接**，甚至还能借助Gnina的CNN评分提升精度！无需手动点击，全程命令行操作，小白也能快速上手。

![image.png](https://s2.loli.net/2025/08/13/XDEFIjePYRwz3MW.png)

## 为什么选择Gnina？

**Gnina** 是基于AutoDock Vina开发的分子对接工具，它保留了Vina的轻量、快速特性，还加入了**卷积神经网络（CNN）评分函数**，能更精准地预测蛋白-配体结合亲和力，尤其适合虚拟筛选和结合模式预测。

我做过简单的测试，用共结晶的配体和蛋白使用AutoDock Vina做分子对接和Gnina的全蛋白对接结果差不多，所以省去了找box步骤便可以批量做分子对接，从而达到虚拟筛选的作用
 
- 安装代码：https://github.com/gnina/gnina
- 视频教程：https://www.youtube.com/watch?v=MG3Srzi5kZ0
- Slides PPT: https://bits.csb.pitt.edu/rsc_workshop2021/docking_with_gnina.slides.html#/
- 示例代码：https://colab.research.google.com/drive/1QYo5QLUE80N_G28PlpYs6OKGddhhd931
- 示例代码：https://colab.research.google.com/drive/1GXmk1v8C-c4UtyKFqIm9HnsrVYH0pI-c

在介绍代码前，先简单说下工具：

下面直接上干货！整套流程包含**配体/蛋白下载、分子对接、结果汇总**，核心代码仅3行，全程自动化。
## 准备工作
创建输出文件
```shell
mkdir -p Ligands Receptors DockResult logs
```

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

同时，确保安装了必要工具：`gnina`、`python`、`pdbfixer`、`xargs`（Linux/macOS自带，Windows可通过WSL或Cygwin使用）。

## 第1行：批量下载配体（PubChem）

从PubChem数据库批量下载配体的SDF文件，存到`Ligands`文件夹：

```bash
cut -f2 Ligand.list | xargs -P5 -I {} -n1 python get_pubchem.py --cid {} --output Ligands/{}.sdf
```

- **解析**：
- `cut -f2 Ligand.list`：提取列表中第二列的PubChem CID（配体唯一标识）；
- `xargs -P5：用5个进程并行下载，加速获取；
- `get_pubchem.py`：辅助脚本（功能是调用PubChem API下载SDF格式配体）。

## 第2行：批量下载蛋白（RCSB PDB）

从RCSB PDB数据库下载蛋白的PDB文件，存到`Receptors`文件夹：

```bash
cut -f2 Receptor.list | xargs -P5 -I {} -n1 python get_pdb.py -i {} -o Receptors
```

- **解析**：
- 类似配体下载逻辑，提取PDB ID后并行下载蛋白结构；
- `get_pdb.py`：辅助脚本（调用pdbfixerAPI下载PDB文件）。
-
## 第3行：批量运行分子对接（Gnina）

这是核心步骤！用Gnina批量处理所有蛋白-配体组合，自动完成对接并输出结果：

```bash
cat Receptor.Ligand | xargs -n2 -P5 bash -c '
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
- `--scoring vina`：使用Vina经典评分函数；
- `--exhaustiveness 256`：搜索强度（值越大结果越可靠，耗时略增）；
- `--device 0`：若有GPU，指定显卡加速（无GPU可省略，自动用CPU）。

## 一步生成对接总结报告

对接完成后，用一行代码汇总所有结果（结合能、排名等），输出CSV表格：
```bash
python3 get_score.py --dockdir DockResult --ligand Ligand.list --receptor Receptor.list --output DockSummary.csv
```

打开`DockSummary.csv`，就能直观看到每个蛋白-配体组合的对接分数，轻松筛选最优候选！

| Protein | PDB  | Affinity  | PubChem   | Mode | CNN_PoseScore | CNN_Affinity |
|---------|------|-----------|-----------|------|---------------|--------------|
| A       | 7LL4 | -16.69515 | 128964461 | 6    | 0.016312076   | 1.672539234  |
| B       | 7LL4 | -16.58057 | 65238     | 12   | 0.014156873   | 1.495260119  |
| C       | 7LL4 | -16.4474  | 74977705  | 19   | 0.012859314   | 1.659733415  |
| D       | 7LL4 | -16.14142 | 45359824  | 15   | 0.013563964   | 1.578759313  |
| E       | 7LL4 | -16.12251 | 129008870 | 9    | 0.015223451   | 1.553036213  |
| F       | 7LL4 | -16.08059 | 74977517  | 16   | 0.013271038   | 1.474256396  |
| G       | 7LL4 | -15.82771 | 129008870 | 8    | 0.015228025   | 1.487853408  |

## 个性化设置

1. **盒子调整**：若`--autobox_ligand`生成的盒子不合适，可手动用`--center_x` `--center_y` `--center_z`和`--size_x` `--size_y` `--size_z`指定；
2. **共结晶**：可以利用该共结晶配体(lig.pdb)，帮助gnina设定box的位置 `--autobox_ligand lig.pdb` 
3. **可视化**：用PyMOL或ChimeraX打开`DockResult`中的SDF文件，可直观查看对接构象。
4. **热图**：可以根据`DockSummary.csv`文件绘制top对接结果的热力值图

