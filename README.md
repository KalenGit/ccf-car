# ccf-car
汽车行业用户观点主题及情感识别————专业陪跑二十年
keras+tensorflow

## CCF 2018 汽车行业用户观点主题及情感识别 
初赛 Rank 16, 复赛A/B Rank 12/22

## 赛题
对文本内容中的讨论主题和情感信息来分析评论用户对所讨论主题的偏好

文本分类，合并主题和情感，分30类

## 方案
### 1. 合并重复id的标签，直接多标签分类，也方便交叉验证

### 2. 分词：word level + char level

  数据量小，单用字比单用词好点；

  300d 预训练固定 + 100d 可训练；

  可以分开训练再融合，也可以直接在模型里合一起；

  我们使用的是https://github.com/Embedding/Chinese-Word-Vectors ；

  其中字词同时训练的，在模型里直接拼接效果非常好。
  
  可以使用不同的预训练词向量做融合（未尝试）。

### 3. 数据增强

  随机去两个样本拼起来，再把它们的标签取maximum, 就得到新样本，我们是每个batch随机采样一个当前batch的和一个全局采样的拼起来。

  提高数据增强数据的比例能略微提升线上成绩。
  
  类似的可以在模型中对句子按一定长度扫描或划分，共享模型权值，预测结果取max。这相当与一种隐性的数据增强（怕跑不动，未尝试）。

### 4. 双向RNN（GRU/LSTM）编码+注意力/胶囊

  注意力模型和胶囊网络效果差不多，注意力模型跑得更快。胶囊网络可以每几个胶囊负责一个主题，也可以所有胶囊Flatten后预测，效果也差不多。

### 5. 输出

  直接输出30类。RNN输出或CRF没有更好地效果（猜测是各标签没啥联系）。

### 6. 训练指标F1，总出现NaN， 花费了很多时间测试修改（偶尔loss也出现NaN，未彻底解决，不是样本问题）

  F1最优结果阈值基本在0.5附近，也可以就看loss，调阈值提交；

  训练集中多标签占比为10%， 提交数量为1.1倍较好。

### 7. 模型融合

  每个模型做五折交叉验证，每折五次平均，效果很好。

  各模型取均值或加权平均。

  第一次尝试stacking，用所有模型预测值当特征，用一个全连接NN做上层，基本同样的五折交叉验证，线上比均值略低。
