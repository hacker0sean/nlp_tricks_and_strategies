#预处理部分
1. 停用词的选取，对于不同任务要不同考虑，有的任务需要大的停用词表，有的任务需要小的停用词表甚至不需要停用词，像AI-challenge中的细粒度情感分析，如果使用BIGRU的话，可能不需要停用词效果比较好。
-网上中英文的都能找到，但是需要根据自己的任务决定是否需要增加和删除一些，比如not、不是等带有否等意义的停用词在有些任务中还是会影响结果的。
- 词筛选，出现次数太多和太少的都可以考虑去掉，太多的一般是这类文本的常用词，太少的往往是拼写错误、命名实体、特殊词组等。
- 也可以考虑根据tfidf来筛选数据平衡
2. 文本更正，中文如果是正常的文本多数都不涉及，但是很多恶意的文本，里面会有大量非法字符，比如在正常的词语中间插入特殊符号，倒序，全半角等。还有一些奇怪的字符，就可能需要你自己维护一个转换表了。如果是英文的，就涉及拼写检查，可以用python包pyenchant实现，比如 mothjer -> mother
3. 文本泛化
 - 表情符号、数字、人名、地址、网址、命名实体等，用关键字替代就行。这个视具体的任务，可能还得往下细化。比如数字也分很多种，普通数字，手机号码，座机号码，热线号码、银行卡号，QQ号，微信号，金钱，距离等等，并且很多任务中，这些还可以单独作为一维特征。还得考虑中文数字阿拉伯数字等。
 - 中文将字转换成拼音，许多恶意文本中会用同音字替代。- 如果是英文的，那可能还得做词干提取、形态还原等，比如fucking,fucked -> fuck
4. 分词
  - 这部分主要是中文，可以只保留长度大于1的词，本人在多个任务中实验过，对结果精度没什么影响，但是可以有效降低特征维，
 - 当然有的时候单纯使用字模型效果比较好，具体使用char还是word可以都测试下。
5. 数据平衡
  -  相信现实中多数任务的数据集都是很不平衡的，不平衡包括两个方面一是数据量不平衡；二是数据多样性不平衡。
   - 主要从采样和数据增强两面解决。比如直接上／下采样到匹配的量，比如先聚类，再按类别上下采样等等。
   - 数据增强，比如随机的增删一些词，将文本翻译成其他语言再翻里细说了。
6. 将词性标注加入训练数据
#模型
1. 主要两个方向吧，词袋模型和词向量模型。
 - 词袋模型更多应用于传统的机器学习分类算法。
 - 词向量是传统方法和深度学习方法都可以用。常见的词向量模型word2vec，fasttext，glove。有空可以都写一个库，将其保存下来。
2. 通用的技巧
 - 可以只保留某几个词性的词语，比如只要形容词和名词
 - 可以加入词的词性特征
 - 可以加入词的情感特征词袋模型
3. 词袋模型 考虑是用词频，还是只关注是否出现用01标示就好。这方面好像就想到这一点，欢迎其他人补充。
4. 词向量模型- 英语还有字符向量，中文还有笔画向量，这是之前在阿里云栖大会上听阿里的人说的，说是实验效果有不少提高，我没实验过，也对这个持怀疑态度。但是在一些特殊场景上适用也是有可能的，比如一些恶意文本，有时候就得用一些形似的字替代。
- 如果是应用于传统方法，那么根据词向量合成句子向量又比较有讲究了，平均，加权平均等，之前在一个任务中试过根据tfidf来加权平均，有一点提升，但是效果很有限。
- 在深度学习模型中，就涉及最大取多少个词的问题，我采取的措施是对正负样本分别统计，综合考虑长度与样本覆盖，用 均值 + n*方差的方式确定，尽量能完全覆盖80%以上的负样本，剩下的再截断，长度对rnn一类的算法性能影响比较大，对cnn类要好很多，所以,cnn类的可以稍微长点关系也不大。
- rnn用双向lstm，一般会比lstm效果好
- 截断从前面截断还是从后面截断不同的任务上可能不同，需要注意。
- 对不均衡的数据，除了数据增强和采样，还可以用何凯明大神最新的facal loss损失函数。但是我自己实验发现，在提高了少类样本准确率的同时，多类的准确率是有所降低的，这个需要根据你自己的具体任务来权衡。
5. 类别太多可以考虑层次分类，不好分的先合并，然后再做内部分类。
6. 迁移特征，尝试从公开的语料库去训练模型，然后可以将模型分类的标签向量作为自己分类模型的输入特征，当然要考虑任务的相关性。
7. 词向量训练的样本数量尽可能大，可以考虑把验证集加进去，再加一点自己爬去的词库。
 

#超参数调节
1.  [A Sensitivity Analysis of (and Practitioners’ Guide to) Convolutional Neural Networks for Sentence Classification](https://arxiv.org/abs/1510.03820)

#神经网络结构
1. fastText（满足大部分场景分类任务，cnn选大的卷积核，dropout必须，bn看情况。多分类未必分类层必须要用softmax，sigmoid有时候效果更佳）
2. TextCNN（与上同）
3. RCNN4
4. char-CNN
5. char-RNN(已测试BIGRU，在文本分类上的效果不错) 
6. HAN

#过拟合
1. dropout和batchnormalization，可以参考cs231n

#奇淫异技
在word embedding后面加dropout
原理就是其实embedded的词向量已经表示文本的特征(语义，位置)，因为目标是文本分类，跟cnn后面dropout一样随机去掉一些特征，抑制潜在的过拟合，但是我的经验是在we层后面用要比在cnn后面用性能更好一些。
