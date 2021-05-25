## 松散的末端

# 用户数据

这个`b2Fixture` ,`b2Body公司`，和`b2Joint`类允许您将用户数据附加为uintptr\。当您检查Box2D数据结构并想确定它们如何与游戏引擎中的对象相关时，这很方便。

例如，典型的做法是将actor指针附加到该actor上的刚体。这将设置循环引用。如果你有演员，你就能得到尸体。如果你有尸体，你就能找到演员。

```c++
GameActor* actor = GameCreateActor();
b2BodyDef bodyDef;
bodyDef.userData.pointer = reinterpret_cast<uintptr_t>(actor);
actor->body = myWorld->CreateBody(&bodyDef);
```

也可以使用它来保存整数值而不是指针。

以下是一些需要用户数据的示例：

- 使用碰撞结果对参与者施加伤害。
- 如果玩家在轴对齐的方框内，则播放脚本事件。
- 当Box2D通知您一个关节将被破坏时，访问游戏结构。

请记住，用户数据是可选的，您可以在其中放入任何内容。但是，你应该始终如一。例如，如果要在一个实体上存储一个actor指针，那么应该在所有实体上保留一个actor指针。不要将actor指针存储在一个主体上，而将foo指针存储在另一个主体上。将actor指针转换为foo指针可能会导致崩溃。

默认情况下，用户数据指针为0。

对于fixture，您可以考虑定义一个用户数据结构来存储特定于游戏的信息，例如材质类型、效果挂钩、声音挂钩等。

```c++
struct FixtureUserData
{
    int materialIndex;
    // ...
};
 
FixtureUserData myData = new FixtureUserData;
myData->materialIndex = 2;
 
b2FixtureDef fixtureDef;
fixtureDef.shape = &someShape;
fixtureDef.userData.pointer = reinterpret_cast<uintptr_t>(myData);
 
b2Fixture* fixture = body->CreateFixture(&fixtureDef);
// ...
 
delete fixture->GetUserData();
body->DestroyFixture(fixture);
```

# 自定义用户数据

您可以定义嵌入Box2D数据结构中的自定义数据结构。这是通过定义`B2_USER_SETTINGS`并提供文件`b2用户设置.h`. 看到了吗`b2_settings.h`了解详情

# 隐性破坏

Box2D不使用引用计数。所以如果你毁了一具尸体，它就真的消失了。访问指向已销毁实体的指针具有未定义的行为。换句话说，你的程序可能会崩溃并烧毁。为了帮助解决这些问题，调试构建内存管理器使用FDFDFDFD填充已销毁的实体。在某些情况下，这有助于更容易地发现问题。

如果销毁Box2D实体，则需要确保删除对已销毁对象的所有引用。如果只有对实体的单个引用，这很容易。如果有多个引用，可以考虑实现一个handle类来包装原始指针。

通常在使用Box2D时，您将创建和销毁许多实体、形状和关节。Box2D在某种程度上自动化了这些实体的管理。如果销毁实体，则所有关联的形状和关节都将自动销毁。这叫做隐性破坏。

销毁实体时，其所有附着的形状、关节和触点都将被销毁。这叫做隐性破坏。任何与这些关节和/或触点相连的物体都会被唤醒。这个过程通常很方便。但是，您必须意识到一个关键问题：

> **注意安全**：当一个实体被破坏时，附着在该实体上的所有固定装置和接头都将自动销毁。必须使指向这些形状和关节的任何指针无效。否则，如果以后尝试访问或破坏这些形状或关节，则程序将死掉。

为了帮助您使关节指针无效，Box2D提供了一个名为[ b2](https://box2d.org/documentation/classb2_destruction_listener.html)你可以实现并提供给你的世界对象。然后，世界对象将通知您关节将被隐式破坏

请注意，当接头或固定装置被明确销毁时，没有通知。在这种情况下，所有权是明确的，您可以当场执行必要的清理。如果愿意，可以调用自己的实现[ b2](https://box2d.org/documentation/classb2_destruction_listener.html)使清除代码保持集中

在许多情况下，隐性破坏是一种极大的便利。它也会使你的程序崩溃。您可以将指向形状和关节的指针存储在代码中的某个位置。当相关联的实体被销毁时，这些指针将成为孤立的。如果考虑到关节通常是由与相关实体的管理无关的代码部分创建的，则情况会变得更糟。例如，测试台创建一个[ b2MouseJoint公司](https://box2d.org/documentation/classb2_mouse_joint.html)用于交互式操纵屏幕上的实体。

Box2D提供了一个回调机制，在隐式销毁发生时通知应用程序。这使应用程序有机会使孤立指针无效。本手册后面将介绍这种回调机制。

您可以实现`b2DestructionListener`这就允许[b2World公司](https://box2d.org/documentation/classb2_world.html)当某个形状或关节因关联实体被破坏而被隐式破坏时通知您。这将有助于防止代码访问孤立指针。

```c++
class MyDestructionListener : public b2DestructionListener
{
    void SayGoodbye(b2Joint* joint)
    {
        // remove all references to joint.
    }
};
```

然后，可以将销毁侦听器的实例注册到world对象。您应该在全局初始化期间执行此操作。

```c++
myWorld->SetListener(myDestructionListener);
```

# 像素和坐标系

回想一下，Box2D使用MKS（米、千克和秒）单位和弧度表示角度。你可能有困难使用米，因为你的游戏是以像素表示的。为了在试验台上处理这个问题，我有全部*游戏*以米为单位工作，只需使用OpenGL视口转换将世界缩放到屏幕空间。

```c++
float lowerX = -25.0f, upperX = 25.0f, lowerY = -5.0f, upperY = 25.0f;
gluOrtho2D(lowerX, upperX, lowerY, upperY);
```

如果你的游戏必须以像素为单位，那么当你从Box2D传递值时，你应该把你的长度单位从像素转换成米。同样，您应该将从Box2D接收到的值从米转换为像素。这将提高物理模拟的稳定性。

你必须想出一个合理的换算系数。我建议你根据角色的大小做出选择。假设您决定每米使用50像素（因为您的角色高75像素）。然后可以使用以下公式将像素转换为米：

```c++
xMeters = 0.02f * xPixels;
yMeters = 0.02f * yPixels;
```

反过来：

```c++
xPixels = 50.0f * xMeters;
yPixels = 50.0f * yMeters;
```

你应该考虑在游戏代码中使用MKS单位，在渲染时只需转换为像素。这将简化游戏逻辑，并减少出错的机会，因为渲染转换可以隔离为少量代码。

如果使用转换因子，则应尝试全局调整它，以确保没有任何中断。你也可以尝试调整它来提高稳定性。

# 调试图纸

您可以实现b2DebugDraw类来获得物理世界的详细绘图。以下是可用实体：

- 形状轮廓
- 联合连通性
- 宽相位轴对齐边界框（AABBs）
- 质心

![debug_draw](images/debug_draw.png)

这是绘制这些物理实体的首选方法，而不是直接访问数据。原因是，许多必要的数据都是内部的，可能会发生变化。

试验台使用调试绘图工具和接触监听器绘制物理实体，因此它是如何实现调试绘图以及如何绘制接触点的主要示例。

# 局限性

Box2D使用几种近似方法有效地模拟刚体物理。这带来了一些限制。

以下是当前的限制：

1. 把沉重的物体堆在轻得多的物体上是不稳定的。当质量比超过10:1时，稳定性降低。
2. 如果一个较轻的物体支撑着一个较重的物体，由关节连接的身体链可能会伸展。例如，一个破坏球连接到一系列轻量物体上可能不稳定。当质量比超过10:1时，稳定性降低。
3. 形状与形状的碰撞通常有0.5cm左右的斜坡。
4. 连续碰撞不会处理关节。所以你可以看到关节在快速移动的物体上伸展。
5. Box2D使用辛欧拉积分方案。它不再现弹丸的抛物线运动，只有一阶精度。但它速度快，稳定性好。
6. Box2D使用迭代解算器来提供实时性能。你不会得到精确的刚性碰撞或像素完美的精确度。增加迭代次数将提高精度。



