![image.png](https://s2.loli.net/2025/08/08/xNeEp3z1UXPqVGk.png)

```
export PYTORCH_LIB=$(python3 -c "import torch; import os; print(os.path.join(torch.__path__[0], 'lib'))")

export LD_LIBRARY_PATH=$PYTORCH_LIB:$LD_LIBRARY_PATH

  

mamba activate vina

# mamba install pdbfixer

mkdir -p Ligands Receptors DockResult logs

```

## Download ligands from PubChem
```shell
cut -f2 Ligand.list | xargs -P3 -I {} -n1 python src/get_pubchem.py --cid {} --output Ligands/{}.sdf
```


## Download proteins from RCSB
```shell
cut -f2 Receptor.list | xargs -P2 -I {} -n1 python src/get_pdb.py -i {} -o Receptors
```

## Run Gnina
```shell
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
## Docking Summary
```shell
python3 src/get_score.py --dockdir DockResult --ligand Ligand.list --recptor Receptor.list --output DockSummary.csv
```
