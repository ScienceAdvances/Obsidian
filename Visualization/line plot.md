
```
plot_stat = plot_data  %>%  group_by(group)  %>% 
    dplyr::summarise(mean = mean(Score),sd=sd(Score))

ggplot(plot_stat,aes(x=group,y=mean,group=1))+
  geom_point(size=4)+
  geom_line(colour="red",size=1.5)+
  geom_errorbar(data=plot_stat,mapping=aes(ymin = mean - sd, ymax = mean + sd), 
                width = 0.1,cex=1.0,colour="blue")+
  labs(x='Treatment Time (h)',y='Score',title = "")+
  # annotate("text", x = 2, y = 0.5, label = str_glue("Kruskal-Test, p = {kp}") )+
  theme_test(base_size = 20)+
  theme(legend.title = element_blank(),
        legend.text = element_text(family = 'sans'), #这里可以改字体为新罗马字体
        legend.position = c(.2,.9),
        legend.direction = "horizontal",
        plot.title = element_text(hjust = 0.5,size = 18),
        axis.text = element_text(color = 'black',family = 'sans'),
        axis.title = element_text(family = 'sans',size = 18,color = 'black'))
```