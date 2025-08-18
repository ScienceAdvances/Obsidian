
- **关于charmm36-jul2022.ff报错的问题**
（Fatal error:atom C1 not found in buid https://www.bilibili.com/opus/740141908904378392

wget http://mackerell.umaryland.edu/download.php?filename=CHARMM_ff_params_files/charmm36-jul2022.ff.tgz
- **二面角**
ERROR 1 [file topol_Protein_chain_A.itp, line 37637]:                                                                                                                                     
  No default Per. Imp. Dih. types for interaction                                                                                                                                         
  '2134  2137  2143  2138     4'.                                                                                                                                                         
<<<<<<< HEAD
http://bbs.keinsci.com/thread-25936-1-1.html 
=======
http://bbs.keinsci.com/thread-25936-1-1.html
打开usr/local/gromacs/share/top/amber99sb-ildn.ff里的ffbonded.itp 搜索[ dihedraltypes ]  这个有两栏，把第一栏里的全部内容复制到14sb的ffbonded.itp里的[ dihedraltypes ]  improper栏的下一栏 就相当与说把amber99sb-ildn.ff里的improper dihedraltypes 复制到14sb的对应位置，使用14sb就不会报错了
- **教程**
http://www.mdtutorials.com/gmx/complex/02_topology.html
>>>>>>> 699169a462f5fa75b1242d85996d5acdfe9af999

- 不能生成3D结构
RDKIT obabel多种方式都报错，oubchem网站生成3d结构也错误
```C
Downloaded 2D SDF for CID: 24871205_2D

==============================

*** Open Babel Warning in CorrectStereoAtoms

Could not correct 4 stereocenter(s) in this molecule (24871205)

with Atom Ids as follows: 11 13 14 17

Warning: Stereochemistry is wrong, using the distance geometry method instead

obabel -:"O1[C@@]2([C@]3(O[C@@H]4O[C@@H]([C@@H](O)[C@H](O)[C@H]4O)CO)[C@]5([C@@](C3)([C@](O[C@]15[H])(O)C2)[H])COC(=O)C6=CC=CC=C6)C" -O Ligands/24871205.sdf --gen3d -addH -p 7.4 --ignore-stereo
```