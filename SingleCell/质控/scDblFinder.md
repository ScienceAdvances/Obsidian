```
srt %<>% Seurat::as.SingleCellExperiment() %>% 
  scDblFinder::scDblFinder(samples='Sample') %>% 
  SeuratObject::as.Seurat()
p <- scCustomize::DimPlot_scCustom(
      seurat_object = srt, 
      figure_plot=TRUE,
      group.by='scDblFinder.class',
      colors_use = c('#00bfc4','#f8766d')
      )
gs(p,outdir=outdir,name='_UMAP_CellType',w=8,h=6)

srt %<>% subset(subset=scDblFinder.class=='singlet')
```

```
https://plger.github.io/scDblFinder/index.html
```