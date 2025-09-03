```
GSE111459_up <- GSE111459 %>% dplyr::filter(Change=="Up") %>% pull(1)
GSE122377_up <- GSE122377 %>% dplyr::filter(Change=="Up") %>% pull(1)
venn_list <- list(GSE111459_up, GSE122377_up)
names(venn_list) <- c("GSE111459", "GSE122377")
venn_gene <- purrr::reduce(venn_list, base::intersect)
fwrite(data.table(GeneID = venn_gene), str_glue("{outdir}/上调交集基因.csv"))
p <- ggvenn::ggvenn(venn_list,
    show_percentage = F, text_size = 8,
    fill_color = c("#1B9E77", "#D95F02", "#7570B3"), fill_alpha = 0.7,
    stroke_size = 0.5, stroke_color = c("white")
)
using::gs(p111,w=7,h=7,name='交集基因',outdir = outdir)
```

```
https://blog.csdn.net/weixin_42670653/article/details/116460184
```

## Python

```
matplotlib_venn.venn2(subsets = [set(list_A),set(list_B)], #绘图数据集
        set_labels = ('A','B'), #设置组名
        set_colors=("#098154","#c72e29"),#设置圈的颜色，中间颜色不能修改
        alpha=0.6,#透明度
        normalize_to=1.0,#venn图占据figure的比例，1.0为占满
       )
# plt.savefig(f"{OUTPUT_DIR}/venn.pdf",bbox_inches="tight");
```

## Venn

```

library(VennDiagram)
venn.plot <- venn.diagram(
  x = venn_list,
  filename = "输出路径.png",
  fill = c("#E64B35", "#EE4C97", "#00A087","#E18727","#F39B7F"),
  alpha = 0.5,
  # 其他参数如标签大小（cex）、分类颜色（cat.col）等按需调整
)
```