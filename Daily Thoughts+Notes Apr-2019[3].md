# Daily Thought (2019.4.8 - 2019.4.13)
**Do More Thinking!** ♈ 

**Ask More Questions!** ♑

**Nothing But the Intuition!** ♐

### 1. 对resnet本质的一些思考
FROM: https://zhuanlan.zhihu.com/p/60668529

**非线性激活层**

目的：引入了非线性，让模型具有更强的拟合能力

副作用：ReLU会造成的低维数据的坍塌（collapse）

解释：低维度的feature在通过ReLU的时候，这个feature会像塌方了一样，有一部分被毁掉了，或者说失去了

具体验证方式就是我们对于一个feature,经过一个映射T,映射到低维的embedding space，然后将这个低维embedding经过RELU，然后在通过同样的T的逆变换，将其映射到原来的featrue的空间

会发现像下面所展示的一样这时候的feature已经发生了很大的变化

![](__pics/resnet_known_1.jpg)

ReLU这个东西，其实就是一个滤波器，只不过这个滤波器的作用域不是信号处理中的频域，而是特征域。那么滤波器又有什么作用呢？维度压缩

也就是降维，我们有m个feature被送入ReLU层，过滤剩下n个（n<m）

**低维数据流经非线性激活层会发生坍塌（信息丢失），而高维数据就不会**

维度低的feature，分布到ReLU的激活带上的概率小，因此经过后信息丢失严重，甚至可能完全丢失。而维度高的feature，分布到ReLU的激活带上的概率大，虽然可能也会有信息的部分丢失，但是无伤大雅，大部分的信息仍然得以保留。

因为维度低的feature信息冗余度较低，所以一旦损失一部分后果较严重，然而对于高维度的信息损失基本大部分都是需要淘汰的，经过学习优胜劣汰下来的有用信息保留度会高。

无法激活的值，当信息无法流过ReLU时，该神经元的输出就会变为0。而在反向传播的过程中，ReLU对0值的梯度为0，即发生了梯度消失。

对于一个M维的数据，我们可以将其看成是在M维空间中的一个M维流形（manifold）。而其中的有用信息，就是在该M维空间中的一个子空间（子空间的维度记为N维，N<=M）中的一个N维流形。非线性激活层相当于压缩了这个M维空间的维度（还记得前面提过的维度压缩吗？）。若是该M维空间中的M维流形本来就不含有冗余信息（M=N），那么再对其进行维度压缩，必然导致信息的丢失。

而维度低的数据其实就是这么一种情况：其信息的冗余度高的可能性本来就低，如果强行对其进行非线性激活（维度压缩），则很有可能丢失掉有用信息，甚至丢失掉全部信息（输出为全0）。

与非线性激活层不同的是，线性激活层并不压缩特征空间的维度。

**使用激活层原则**
- 1. 对含有冗余信息的数据使用非线性激活（如ReLU），对不含冗余信息的数据使用线性激活（如一些线性变换）。
- 2. 两种类型的激活交替灵活使用，以同时兼顾非线性和信息的完整性。
- 3. 由于冗余信息和非冗余信息所携带的有用信息是一样多的，因此在设计网络时，对内存消耗大的结构最好是用在非冗余信息上。

**ResNet就是满足这样原则的设计**

降低数据中信息的冗余度。

![](__pics/resnet_known_2.jpg)

x分支：对非冗余信息采用了线性激活（通过skip connection获得无冗余的identity部分）

参差分支：然后对冗余信息采用了非线性激活（通过ReLU对identity之外的其余部分进行信息提取/过滤，提取出的有用信息即是残差）。

**从数据中拿掉了非冗余信息的identity部分，会导致余下部分的信息冗余度变高。这就像从接近饱和的溶液中移走了一部分溶质，会使得剩下的溶液的饱和度降低，一个道理。**

**Resnet它带来的好处：**

- 1. 由于identity之外的其余部分的信息冗余度较高，因此在对其使用ReLU进行非线性激活时，丢失的有用信息也会较少，ReLU层输出为0的可能性也会较低。这就降低了在反向传播时ReLU的梯度消失的概率，从而便于网络的加深，以大大地发挥深度网络的潜能。
- 2. 特征复用能加快模型的学习速度，因为参数的优化收敛得快（从identity的基础上直接学习残差，总比从头学习全部来得快）。

**最后是两个小tips：**
- 1. 如果一个信息可以完整地流过一个非线性激活层，则这个非线性激活层对于这个信息而言，相当于仅仅作了一个线性激活。
- 2. 解决由非线性激活导致的反向传播梯度消失的窍门，就是要提高进行非线性激活的信息的冗余度。
