# 自底向上分析boltdb源码

本书是采用自底向上的方式来介绍boltdb内部的实现原理。其实我们经常都在采用**自底向上**或者**自顶向下**这两种方式来思考和求解问题。

例如：我们阅读源码时，通常都是从最顶层的接口点进去，然后层层深入内部。这其实本质上就是一种自顶向下的方式。

又比如我们平常做开发时，都是先将系统进行拆分、解耦。然后一般都会采用从下而上或者从上而下的方式来进行开发迭代。

又比如在执行OKR的时候，我们通常都是先定目标、然后依据该目标再层层分解。最后分解到可执行的单元为止。这其实也是一种自顶向
下的方式；在真正执行时，我们通常又是从原先分解到最底层的原子单元开始执行，然后层层递进。最终所有的原子单元执行完后，我们
的目标也就实现了。这其实又是自底向上的完成任务方式。


前面所提到的上下其实是指某些事物、组件、目标内部是存在相互依赖、因果、先后、递进关系，而依赖方或者结果方则位于上，被依赖方、原因
方位于下；个人认为自下而上的方式比较适合执行每一个任务或者解决子问题，每一层做完时，都可以独立的进行测试和验证。当我们层层自下
而上开发。最后将其前面的所有东西拼接在一起时，就构成了一个完整的组件或者实现了该目标。

回到最初的话题，为什么本书要采用自底向上的方式来写呢？

对于一个文件型数据库而言，**所谓的上**指的是暴露给用户侧的调用接口。**所谓的下**又指它的输出(数据)最终要落到磁盘这种
存储介质上。采用自底向上的方式的话，也就意味着我们先从磁盘这一层进行分析。然后逐步衍生到内存；再到用户接口这一层。层层之间是
被依赖的一种关系。这样的话，其实就比较好理解了。在本书中，本人采用自底向上的方式来介绍。希望阅读完后，有一种自己从0到1构建了
一块数据库的快感。

当然也可以采用自顶向下的方式来介绍，这时我们就需要在介绍最上层时，先假设它所依赖的底层都已经就绪了，我们只分析当层内容。然后层层
往下扩展。

之前和一位大佬进行过针对此问题的探讨，在不同的场景、不同的组件中。具体采用**自底向上**还是**自顶向下**来分析。见仁见智，也具体问题具体分析。当要达成的目标足够清晰时，通过自顶向下的方式可以倒推达成目标需要完成的几个阶段任务。然后再依次进行细分展开。


下面是我们的boltdb微信交流群，有问题、疑问都可以扫码加入群一起交流讨论,二维码过期的话也可以加群管理员或者加微信号`wen2282186474`申请入群，邀请入群。

![imgs/boltdb群二维码.jpeg](imgs/boltdb群二维码.jpeg)

![imgs/群管理微信二维码.jpeg](imgs/群管理微信二维码.jpeg)
