

在医学影像诊断领域，一项突破性研究正悄然改变着放射科医生的工作模式。近日，发表于《Nature Communications》的最新成果显示，研究团队成功开发出适用于 3D 脑部 CT 影像的多模态大语言模型 BrainGPT，其生成的诊断报告在临床盲测中达到 74% 的人类混淆率，为实现人机协同诊断开辟了新路径。

**![图1：3D脑部CT报告生成技术全流程示意图](https://s2.loli.net/2025/08/29/tDWEuhYQzVqPXO4.png)
## 三维影像的空白地带  

以往大模型在医学影像的尝试大多集中在二维胸片，病灶种类相对单一。而三维脑 CT 涉及脑出血、梗死、萎缩、占位等复杂异质病变，且一份规范报告必须精确到“程度-解剖位-征象-印象”四级要素。任何一级缺失都会直接影响临床决策。因此，作者把问题拆成三件事：  
1.  构建大规模三维脑 CT-文本配对数据（18 885 例，74 万张切片）；    
2.  让模型学会在 3D 语境里做“多图推理”；   
3.  建立能够衡量“临床信息密度”的评估框架，而非传统 BLEU/CIDEr 这类翻译指标。 

## 模型：从 Otter 到 BrainGPT 的四级跃迁  
基座是开源多图 Otter（CLIP-ViT-L/14 + LLaMA-7B）。作者提出“临床视觉指令微调（CVIT）”，把放射科日常问答模板和关键词指引显式地写进 prompt，形成四条递进路径：  
- Plain：仅角色定义——“你是神经放射科助手”；  
- Example：3-shot 示范；  
- Template：结构化问答，如“请描述出血量、位置、是否破入脑室”；  
- Keyword：直接给出“程度/解剖位/征象/印象”四类关键词，要求模型填空式输出。  

仅用 2×A100 训练 12 小时，参数量保持 7 B 不变，便得到 BrainGPT-plain 到 BrainGPT-keyword 四个版本。

作者开源了
- 最优模型权重：https://huggingface.co/Charliebear/BrainGPT
- 训练代码：https://zenodo.org/records/14852686

![image.png](https://s2.loli.net/2025/08/29/tvyIYhRQe3WzuOA.png)

## 结果 1：传统指标失灵，新预处理“扶正”分数  
直接拿 BLEU-4/CIDEr-R 评价长报告，分数低得离谱（BLEU-4 接近 0）。作者发现症结在于：  
- 人类报告是“条目式”——先后次序不影响正确性；  
- 大量否定句“未见出血”拉低 n-gram 重叠。  

于是他们引入两条预处理：  
1. Sentence Pairing：利用 Sentence-BERT 把生成报告与金标准逐句对齐，解除顺序惩罚；  
2. Negation Removal：过滤掉含 “no/without” 的句子，只保留阳性发现。  

结果 CIDEr-R 从 5.9 暴增至 153，BLEU-4 升到 57；更重要的是，CVIT 的阶梯效应（keyword > template > example > plain）首次在数值上显著拉开差距，证明指令设计的确在传递医学知识。
![image.png](https://s2.loli.net/2025/08/29/4wSJNkVgE5hTyze.png)

## 结果 2：FORTE——面向特征的放射任务评估  
传统指标无法回答“模型漏掉了哪个关键信息”。作者提出 FORTE，把每条句子拆成四元组：  
- Degree（大小/密度）：如“1.5 cm”、“高密度”；  
- Landmark（解剖）：如“左基底节”、“鞍上池”；  
- Feature（征象）：如“急性梗死”、“占位效应”；  
- Impression（诊断）：如“考虑转移瘤”。  

为 4 类关键词各维护同义词表后计算 F1。BrainGPT-keyword 在四类分别达到 0.649、0.574、0.533、0.548，相比基座模型提升 3–5 倍，且 CVIT 越深入，F1 越高，形成严格单调关系。
![image.png](https://s2.loli.net/2025/08/29/YJGEqiDxKrzobc2.png)

## 结果 3：零样本外部验证 CQ500——未见过的创伤也能写  
CQ500 是公开的急性颅脑外伤数据集（133 例），训练集中几乎没有同类病变。零样本测试中，BrainGPT-keyword 对“mass effect” 准确率 0.75，“midline shift” 0.38，“hemorrhage” 0.50；经 negation removal 后分别升至 0.91、0.61。模型还能一次性识别同一张片上的多种出血类型（硬膜下、蛛网膜下、脑实质），并给出“右侧枕叶脑挫裂伤伴硬膜下血肿，中线左偏 5 mm”这样的复合描述，显示跨域语义迁移能力。

![image.png](https://s2.loli.net/2025/08/29/PekJhoinSWqLcdB.png)

## 结果 4：LLM-as-a-judge 与人机 Turing 测试  
团队用 GPT-4o-mini（DocLens）做“二次裁判”，把报告拆成医学陈述再比对金标准。DocLens 与 FORTE 的 Pearson r=0.78，远高于其与传统指标的 r<0.4，验证 FORTE 确实能捕捉临床实质。  
更严格的考验是 11 位医师参与的图灵测试：只看文本，74 % 的 BrainGPT 报告被误判为人写；若同时给出 CT 影像，误判率仍达 56 %。医师们坦言，机器报告在“用语熟悉度”和“细节具体度”上已与人类难分伯仲，但在“临床语境聚焦”上仍略显冗长——这与 negation removal 之前的“interpretation spree”现象一致。
![image.png](https://s2.loli.net/2025/08/29/CjnvMHt1x6zbBQN.png)
## 总结

这项研究第一次把“大模型读 3D 脑 CT”推进到了可临床落地的边缘：在真实外部数据上，它已能写出信息完整、用词准确的结构化报告，且仅需单卡训练一夜。下一步，或许我们会让 AI 先在夜班急诊室充当第二读片人，而放射科医师只需把目光留给最疑难的病例。
