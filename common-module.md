## 通用模块

公共模块包含设置、内存管理和向量数学。

# 设置

标题b2Settings.h包含：

- 类型，如int32和float
- 常量
- 分配包装器
- 版本号

## 类型

Box2D定义了各种类型，如int8等，以便于确定结构的大小。

## 常量

Box2D定义了几个常量。这些都记录在b2Settings.h中。通常不需要调整这些常数。

Box2D使用浮点数学进行碰撞和模拟。由于舍入误差，定义了一些数值公差。有些公差是绝对的，有些是相对的。绝对公差使用MKS单位。

## 分配包装器

设置文件为大型分配定义了b2Alloc和b2Free。您可以将这些调用转发到您自己的内存管理系统。

## 版本

这个[b2版本](https://box2d.org/documentation/structb2_version.html)结构保存当前版本，因此您可以在运行时对此进行查询。

# 内存管理

关于Box2D设计的许多决定都是基于对快速高效地使用内存的需要。在本节中，我将讨论Box2D如何以及为什么分配内存。

Box2D倾向于分配大量的小对象（大约50-300字节）。通过malloc或new对小对象使用系统堆是低效的，并且可能导致碎片化。其中许多小对象的寿命可能很短，例如触点，但可以持续几个时间步。因此，我们需要一个能够有效地为这些对象提供堆内存的分配器。

Box2D的解决方案是使用一个名为[ B2Block分配器](https://box2d.org/documentation/classb2_block_allocator.html). SOA保留了大量大小不等的可增长池。当请求内存时，SOA将返回最适合请求大小的内存块。释放块后，它将返回到池中。这两种操作都很快，并且几乎不会引起堆流量。

因为Box2D使用SOA，所以您永远不应该新建或malloc一个主体、设备或接头。但是，您必须分配[b2World公司](https://box2d.org/documentation/classb2_world.html)靠你自己。这个[b2World公司](https://box2d.org/documentation/classb2_world.html)类为您提供用于创建实体、装置和运动类型的工厂。这使得Box2D可以使用SOA并对您隐藏血腥的细节。永远不要对实体、夹具或关节调用delete或free。

在执行时间点时，Box2D需要一些临时工作区内存。为此，它使用一个堆栈分配器[ B2Stack分配器](https://box2d.org/documentation/classb2_stack_allocator.html)以避免逐步堆分配。您不需要与堆栈分配器交互，但知道它在那里是很好的。

# 数学

Box2D包含一个简单的小向量和矩阵模块。这是为了满足Box2D和API的内部需求而设计的。所有成员都是公开的，因此您可以在应用程序中自由使用它们。

数学库保持简单，使Box2D易于移植和维护。