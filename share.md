#CFWCR心路历程
我们的方法基于业界流行的相关滤波的框架。我们使用了单cnn特征的多尺度追踪方案。我们发现现有的很多追踪器融合了cnn特征和传统的机器学习特征，如hog特征，cn颜色特征等。在我们的实验中，我们发现cnn的浅层特征具有物体轮廓的信息，高层的深度特征具有物体的语义信息，将cnn的浅层和高层特征进行融合能使得追踪器具有很好的性能。于是，我们摒弃了传统的特征，只使用了cnn的特征，这一做法，使得我们的追踪器无论是在速度上还是精度上都有了不小的提高。  

当时我们在做VOT竞赛的时候，首先跑了ECO，发现复现不了作者的结果，在16上的EAO大概是0.35，在17上的EAO大概是0.26。在MD大神的代码的基础上，虽然还没复现大神的结果，但是我们有了很好的baseline。我们当时考虑了很多的改进措施：  
a、特征组合方式，CNN和传统特征的加权组合，CNN不同层之间的加权组合。  
b、对pca作用的思考和实验  
c、模型更新策略实验  
d、非正矩形框追踪  
e、特征归一化方式  
f、其他cnn模型以及模型集成  
g、其他超参数的调试，如搜索区域，前馈图片尺度，样本更新策略，不同的resize方法，调整训练学习率，不同的窗函数阈值，不同的多尺度参数等等。  
h、端到端训练一个cnn替换vgg，有点像CFCF的思路。   

世事总是不尽入人意，我们花了一个月做了很多的尝试，90％都失败了，不得不说这种非端到端的训练框架如果对每一部分了解不充分的话，很难调试。即使是在深入读了论文和阅读完MD大神的代码的基础上，我们仍然走了很多弯路。  

思路a的产生是我们考虑到特征之间可能是有轮廓和语义上的重复性的，在这么多超参数下，组合这么多特征很可能有很多冗余，这也是MD大神在ECO中提出PCA能取到很好效果的原因。我们发现仅仅用CNN特征结果就很好了，那么其他特征是不是必要的呢？我们在改了特征之后，相应地改了多尺度等的超参数，发现结果还能提升。后来，我们又尝试了一些特征组合方式，发现仅仅用CNN特征的结果最好。在最后，我们加入了对CNN不同层特征的得分矩阵进行加权的方法，性能略有提升，不过这个参数容易在某个数据集过拟合。  

在有了思路a的实验结果后，我们只剩下CNN特征，这样需要调试的超参数就少了一些。特征的减少也导致过拟合的现象减轻了，于是，我们思考PCA对于性能的提升是否是必要的。我们发现，去掉PCA之后，EAO还能提高。
思路c的产生在于我们看ECO论文的时候发现，MD大神直接将更新步长定死为5了，有点暴力了，而且可能是针对某个tracking数据集调出来的。我们尝试了不同的步长和根据得分的大小来决定是否更新的方法，都还没MD大神直接设置为5好。  

思路d的产生在于我们考虑到EAO这个指标是根据重叠比率来评估的，如果框是非正的，可能会和标注的框有更大的重叠度，但是搜索了大量的论文，发现这方面的工作实在是太少，可能需要很大的工作量，遂放弃。
思路e和f和h来自于做cnn项目的一些经验，不过由于时间关系有限，我们都只是粗略地尝试了一些就放弃了。
