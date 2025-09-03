![image.png](https://s2.loli.net/2025/09/03/Zg9FbtoLwX1COrQ.png)

桑基图（Sankey diagram），即桑基能量分流图，也叫桑基能量平衡图。它是一种特定类型的流程图，图中延伸的**分支的宽度对应数据流量的大小**，通常应用于能源、材料成分、金融等数据的可视化分析。因1898年Matthew Henry Phineas Riall Sankey绘制的“蒸汽机的能源效率图”而闻名，此后便以其名字命名为“桑基图”。

```
using(ggalluvial)
head(majors)
```

|   |   |   |
|---|---|---|
|student|semester|curriculum|
|1|CURR1|Painting|
|2|CURR1|Painting|
|6|CURR1|Sculpure|
|8|CURR1|Painting|
|9|CURR1|Sculpure|
|10|CURR1|Painting|
|11|CURR1|Digital Art|

```
#生成alluvia格式 （long to wide数据格式）
dcast(setDT(majors),student~semester, value.var = "curriculum")
```

|   |   |   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|
|student|CURR1|CURR3|CURR5|CURR7|CURR9|CURR11|CURR13|CURR15|
|1|Painting|Painting|Painting|Painting|Painting|Painting|Painting|Painting|
|2|Painting|Painting|Painting|Painting|Painting|Painting|||
|6|Sculpure|Sculpure|Painting|Painting|Painting|Painting|Painting|Painting|
|8|Painting|Painting|Painting|Painting|Painting|Painting|Painting||
|9|Sculpure|Art History|Art History|Painting|Painting|Painting|Painting|Painting|
|10|Painting|Painting|Painting|Painting|Painting|Painting|||

```
#key,semester列是代表坐标轴axis
#value,curriculum列是层
#id,student列是冲积流

gg <- ggplot(majors_alluvia,aes(axis1 = CURR1, axis2 = CURR7, axis3 = CURR13))
#定义三条strate层
gg +
  #冲击流：
  geom_alluvium(aes(fill = as.factor(student)), width = 2/5, discern = F) +
  #width，宽度所占图的比例，默认1/3

  #strate层：
  geom_stratum(width = 2/5, discern = F) +

  #添加文本，统计每层的频率
  geom_text(stat = "stratum", discern = F, aes(label = after_stat(stratum)))
```

# References

[https://mp.weixin.qq.com/s/Xc50nTczwcc0r8KROUU8Ig](https://mp.weixin.qq.com/s/Xc50nTczwcc0r8KROUU8Ig)


![image.png](https://s2.loli.net/2025/09/03/jyWRgr783AQ6FEh.png)
```
set.seed(42424)   
x <- rnorm(10000,sd=0.35)-0.3
y <- x + rnorm(10000,sd=0.6)+0.3

dat = tibble(x=x,y=y)  %>% 
    mutate(group= case_when(x>0.5 & y>1~"Hyper-up",
    x<(-1) & y<(-1)~"Hypo-down",
    TRUE~"NO")) %>% mutate(group=factor(group,levels = c("Hyper-up","NO","Hypo-down")))


ggplot(data=dat,aes(x,y,color=group))+
    geom_point()+
    xlim(-2,1.2)+ylim(-3.2,3.2)+
    geom_hline(yintercept=-1,size=1.5,linetype =2)+
    geom_hline(yintercept=1,size=1.5,linetype =2)+
    geom_vline(xintercept=-1,size=1.5,linetype =2)+
    geom_vline(xintercept=0.5,size=1.5,linetype =2)+
    scale_color_manual(values=c("#a93b3be0","#9e9e9e","#657cb8"))+
    labs(x="m6A modification(log2FC)",y="Gnene expression(log2FC)")+
    annotate(geom = "text",x=-1.5,y=3,label="Hypo-up",size=10,color="#e0cd6c")+
    annotate(geom = "text",x=-1.5,y=-2,label="Hypo-down",size=10,color="#657cb8")+
    annotate(geom = "text",x=0.5,y=3,label="Hyper-up",size=10,color="#a93b3be0")+
    annotate(geom = "text",x=0.5,y=-2,label="Hyper-down",size=10,color="#6db67b")+
    theme(axis.title = element_text(size = 25),legend.position = "none",
        axis.text = element_text(size=22))

ggsave("m6a.pdf",width = 6,height = 6)

```