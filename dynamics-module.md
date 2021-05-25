## 动力学模块

动力学模块是Box2D中最复杂的部分，也是您可能与之交互最多的部分。Dynamics模块位于Common和Collision模块之上，因此您现在应该对这些模块比较熟悉了。

动态模块包含：

- 夹具等级
- 刚体类
- 接触等级
- 联合类
- 世界级
- 侦听器类

这些类之间有很多依赖关系，所以很难描述一个类而不引用另一个类。在下面，您可能会看到一些尚未描述的类的引用。因此，在仔细阅读本章之前，您可能需要快速浏览本章。

以下章节将介绍动力学模块。

# 身体

物体有位置和速度。你可以对物体施加力、力矩和脉冲。实体可以是静态的、运动的或动态的。以下是车身类型定义：

### b2_静体

静态物体在模拟下不会移动，其行为就像它有无限大的质量一样。在内部，Box2D为质量和反向质量存储零。静态实体可以由用户手动移动。静止物体的速度为零。静态实体不会与其他静态或运动学实体碰撞。

### 运动体

运动物体在模拟下根据其速度运动。运动体对力没有反应。它们可以由用户手动移动，但通常运动体是通过设置其速度来移动的。运动学实体的行为就好像它有无限大的质量，但是，Box2D为质量和反向质量存储零。运动学实体不会与其他运动学或静态实体碰撞。

### 动态车身

一个动态物体被完全模拟。它们可以由用户手动移动，但通常它们是根据力移动的。动态实体可以与所有实体类型发生碰撞。动态物体总是有有限的，非零质量。如果你试图把一个动态物体的质量设为零，它会自动获得一公斤的质量，而且它不会旋转。

实体是固定装置（形状）的支柱。尸体携带固定装置，并在世界各地移动。Box2D中的实体始终是刚体。这意味着附着在同一刚体上的两个固定装置永远不会相对移动，附着到同一个刚体上的固定装置也不会碰撞。

夹具具有碰撞几何体和密度。通常，实体从固定装置获得其质量特性。但是，可以在构造实体后替代质量特性。

通常，您将指针指向您创建的所有实体。这样可以查询实体位置以更新图形实体的位置。你还应该保留身体指针，这样当你用完身体指针的时候就可以销毁它们。

## 身体定义

在创建实体之前，必须创建实体定义([ b2BodyDef公司](https://box2d.org/documentation/structb2_body_def.html)). 主体定义包含创建和初始化主体所需的数据。

Box2D将数据复制出body定义；它不保留指向主体定义的指针。这意味着您可以循环使用实体定义来创建多个实体。

让我们回顾一下身体定义的一些关键成员。

## 车身类型

正如本章开头所讨论的，有三种不同的车身类型：静态、运动学和动态。您应该在创建时建立实体类型，因为稍后更改实体类型的成本很高。

```c++
b2BodyDef bodyDef;
bodyDef.type = b2_dynamicBody;
```

必须设置正文类型

## 位置和角度

实体定义为您提供了在创建时初始化实体位置的机会。这比在世界原点创建实体然后移动实体要好得多。

> **注意安全**：不要在原点创建实体，然后再移动它。如果在原点创建多个实体，则性能将受到影响。

一个团体有两个主要的兴趣点。第一点是物体的起源。固定装置和关节相对于实体的原点附着。第二个兴趣点是质心。质心由附着形状的质量分布确定，或明确设置为[ b2MassData公司](https://box2d.org/documentation/structb2_mass_data.html). Box2D的许多内部计算都使用质心位置。例如[b2Body公司](https://box2d.org/documentation/classb2_body.html)存储质心的线速度

在构建实体定义时，可能不知道质心位于何处。因此可以指定实体原点的位置。也可以以弧度指定实体的角度，该角度不受质心位置的影响。如果以后更改实体的质量特性，则质心可能会在实体上移动，但原点位置不会更改，附着的形状和关节也不会移动。

```c++
b2BodyDef bodyDef;
bodyDef.position.Set(0.0f, 2.0f); // the body's origin position.
bodyDef.angle = 0.25f * b2_pi; // the body's angle in radians.
```

刚体也是参照系。可以在该框架中定义装置和接头。这些固定装置和关节固定器在身体的局部框架内永远不会移动。

## 阻尼

阻尼用于降低物体的世界速度。阻尼不同于摩擦，因为摩擦只在接触时发生。阻尼不能代替摩擦力，这两种效果应该一起使用。

阻尼参数应介于0和无穷大之间，0表示无阻尼，无穷大表示完全阻尼。通常，您将使用介于0和0.1之间的阻尼值。我一般不使用线性阻尼，因为它使物体看起来像是漂浮的。

```c++
b2BodyDef bodyDef;
bodyDef.linearDamping = 0.0f;
bodyDef.angularDamping = 0.01f;
```

阻尼近似用于稳定性和性能。在较小的阻尼值下，阻尼效应主要与时间步长无关。在较大的阻尼值下，阻尼效果将随时间步长而变化。如果使用固定的时间步长（推荐），这不是问题。

## 重力秤

可以使用“重力比例”调整单个实体上的重力。但是要小心，增加重力会降低稳定性。

```c++
// Set the gravity scale to zero so this body will float
b2BodyDef bodyDef;
bodyDef.gravityScale = 0.0f;
```

## 睡眠参数

睡眠是什么意思？模拟物体是很昂贵的，所以我们越少模拟越好。当一个物体静止时，我们不想再模拟它了。

当Box2D确定一个实体（或一组实体）已停止时，该实体将进入睡眠状态，该状态的CPU开销非常小。如果一个身体是清醒的并且与一个睡着的身体相撞，那么这个睡着的身体就会醒来。如果连在身上的关节或接触点被破坏，尸体也会醒来。你也可以手动唤醒尸体。

实体定义允许您指定实体是否可以睡眠以及创建的实体是否处于睡眠状态。

```c++
b2BodyDef bodyDef;
bodyDef.allowSleep = true;
bodyDef.awake = true;
```

## 固定旋转

您可能希望刚体（例如角色）具有固定的旋转。这样的物体即使在负载下也不应该旋转。可以使用“固定旋转”设置来实现此目的：

```c++
b2BodyDef bodyDef;
bodyDef.fixedRotation = true;
```

固定旋转标志使转动惯量及其反向设置为零。

## 子弹

游戏模拟通常生成一系列以一定帧速率播放的图像。这叫做离散模拟。在离散仿真中，刚体可以在一个时间步长内进行大量的运动。如果物理引擎不能解释大运动，你可能会看到一些物体错误地通过对方。这种效应被称为隧道效应。

默认情况下，Box2D使用连续碰撞检测（CCD）来防止动态物体穿过静态物体。这是通过将形状从旧位置扫到新位置来完成的。引擎在扫描期间查找新的碰撞，并计算这些碰撞的碰撞时间（TOI）。将实体移动到其第一个TOI，然后解算器执行子步骤以完成整个时间步骤。子步骤中可能有其他TOI事件。

通常情况下，动态物体之间不使用CCD。这样做是为了保持性能合理。在某些游戏场景中，你需要动态物体来使用CCD。例如，您可能希望向一堆动态砖块发射高速子弹。如果没有CCD，子弹可能会穿过砖块。

Box2D中快速移动的对象可以标记为项目符号。子弹将执行CCD与静态和动态物体。你应该根据你的游戏设计来决定什么样的身体应该是子弹。如果您决定将尸体视为子弹，请使用以下设置。

```c++
b2BodyDef bodyDef;
bodyDef.bullet = true;
```

项目符号标志仅影响动态实体。

## 激活

您可能希望创建实体，但不参与碰撞或动力学。这种状态类似于睡眠，只是身体不会被其他身体唤醒，身体的固定装置也不会放在宽相位。这意味着身体不会参与碰撞、光线投射等。

可以创建处于非活动状态的实体，然后重新激活它。

```c++
b2BodyDef bodyDef;
bodyDef.active = true;
```

关节可以连接到非活动体。不会模拟这些关节。当你激活一个物体时，你应该小心它的关节没有变形。

请注意，激活实体几乎与从头创建实体一样昂贵。所以你不应该在流媒体世界中使用激活。使用流媒体世界的创建/销毁来节省内存。

## 用户数据

用户数据是空指针。这为您提供了一个钩子，用于将应用程序对象链接到实体。您应该保持一致，以便对所有主体用户数据使用相同的对象类型。

```c++
b2BodyDef bodyDef;
bodyDef.userData.pointer = reinterpret_cast<uintptr_t>(&myActor);
```

## 车身厂

尸体是用世界级的尸体工厂制造和销毁的。这使得世界可以用一个高效的分配器创建主体，并将主体添加到world数据结构中。

```c++
b2World* myWorld;
b2Body* dynamicBody = myWorld->CreateBody(&bodyDef);
 
// ... do stuff ...
 
myWorld->DestroyBody(dynamicBody);
dynamicBody = nullptr;
```

> **注意安全**：不要使用new或malloc创建实体。这个世界不会知道身体，身体也不会被正确初始化。

Box2D不保留对body定义或它所包含的任何数据的引用（除了用户数据指针）。因此，可以创建临时实体定义并重用相同的实体定义。

Box2D允许您通过删除[b2World公司](https://box2d.org/documentation/classb2_world.html)对象，它为您完成所有清理工作。但是，您应该注意将保存在游戏引擎中的身体指针作废。

销毁实体时，附着的固定装置和运动类型将自动销毁。这对于如何管理形状和关节指针具有重要的意义。

## 使用身体

创建实体后，可以对该实体执行许多操作。其中包括设置质量属性、访问位置和速度、应用力以及转换点和向量。

## 海量数据

物体有质量（标量）、质心（2矢量）和转动惯量（标量）。对于静态物体，质量和转动惯量设置为零。当物体有固定的转动时，它的转动惯量为零。

通常，在向实体添加固定装置时，将自动建立实体的质量特性。你也可以调整身体的质量。这通常是当你有特殊的游戏场景需要改变质量。

```c++
void b2Body::SetMassData(const b2MassData* data);
```

在直接设置实体的质量之后，您可能希望恢复到固定装置指定的自然质量。您可以使用：

```c++
void b2Body::ResetMassData();
```

车身质量数据可通过以下功能获得：

```c++
float b2Body::GetMass() const;
float b2Body::GetInertia() const;
const b2Vec2& b2Body::GetLocalCenter() const;
void b2Body::GetMassData(b2MassData* data) const;
```

## 状态信息

身体的状态有很多方面。您可以通过以下函数有效地访问此状态数据：

```c++
void b2Body::SetType(b2BodyType type);
b2BodyType b2Body::GetType();
void b2Body::SetBullet(bool flag);
bool b2Body::IsBullet() const;
void b2Body::SetSleepingAllowed(bool flag);
bool b2Body::IsSleepingAllowed() const;
void b2Body::SetAwake(bool flag);
bool b2Body::IsAwake() const;
void b2Body::SetEnabled(bool flag);
bool b2Body::IsEnabled() const;
void b2Body::SetFixedRotation(bool flag);
bool b2Body::IsFixedRotation() const;
```

## 位置和速度

可以访问实体的位置和旋转。这在渲染关联的游戏角色时很常见。您也可以设置位置，尽管这不太常见，因为您通常使用Box2D来模拟移动。

```c++
bool b2Body::SetTransform(const b2Vec2& position, float angle);
const b2Transform& b2Body::GetTransform() const;
const b2Vec2& b2Body::GetPosition() const;
float b2Body::GetAngle() const;
```

可以访问本地坐标和世界坐标中的质心位置。Box2D中的大部分内部模拟都使用质心。但是，您通常不需要访问它。相反，您通常会使用身体变换。例如，你可能有一个正方形的身体。物体的原点可能是正方形的一角，而质心位于正方形的中心。

```c++
const b2Vec2& b2Body::GetWorldCenter() const;
const b2Vec2& b2Body::GetLocalCenter() const;
```

可以访问线速度和角速度。线速度代表质心。因此，如果质量特性发生变化，线速度可能会发生变化。

## 力和冲动

你可以对物体施加力、力矩和脉冲。当应用力或冲量时，将提供一个施加载荷的世界点。这通常会产生绕质心的力矩。

```c++
void b2Body::ApplyForce(const b2Vec2& force, const b2Vec2& point);
void b2Body::ApplyTorque(float torque);
void b2Body::ApplyLinearImpulse(const b2Vec2& impulse, const b2Vec2& point);
void b2Body::ApplyAngularImpulse(float impulse);
```

施加一个力、转矩或脉冲会唤醒身体。有时这是不可取的。例如，你可能正在施加一个稳定的力，并想让身体睡觉来提高表现。在这种情况下，您可以使用以下代码。

```c++
if (myBody->IsAwake() == true)
{
    myBody->ApplyForce(myForce, myPoint);
}
```

## 坐标变换

body类有一些实用函数，可以帮助您在局部空间和世界空间之间转换点和向量。如果你不明白这些概念，请阅读吉姆·范维思和拉尔斯·毕肖普的《游戏和交互应用的基本数学》。这些函数是有效的（内联时）。

```c++
b2Vec2 b2Body::GetWorldPoint(const b2Vec2& localPoint);
b2Vec2 b2Body::GetWorldVector(const b2Vec2& localVector);
b2Vec2 b2Body::GetLocalPoint(const b2Vec2& worldPoint);
b2Vec2 b2Body::GetLocalVector(const b2Vec2& worldVector);
```

## 接触夹具、接头和触点

可以对实体的固定装置进行迭代。如果需要访问fixture的用户数据，这一点非常有用。

```c++
for (b2Fixture* f = body->GetFixtureList(); f; f = f->GetNext())
{
    MyFixtureData* data = (MyFixtureData*)f->GetUserData();
    // do something with data ...
}
```

类似地，可以迭代实体的关节列表。

主体还提供关联联系人的列表。你可以使用这个来获取关于当前联系人的信息。请小心，因为联系人列表可能不包含上一时间步中存在的所有联系人。

# 固定装置

回想一下，形状并不了解物体，可以独立于物理模拟来使用。因此，Box2D提供[B2固定装置](https://box2d.org/documentation/classb2_fixture.html)类将形状附加到实体。一个实体可以有零个或多个固定装置。具有多个固定装置的实体有时称为*复合体*

固定装置包括：

- 单一形状
- 宽相位代理
- 密度、摩擦和恢复
- 冲突过滤标志
- 指向父实体的后退指针
- 用户数据
- 传感器标志

这些将在以下章节中进行描述。

## 夹具创建

通过初始化固定装置定义，然后将定义传递给父实体来创建装置。

```c++
b2Body* myBody;
b2FixtureDef fixtureDef;
fixtureDef.shape = &myShape;
fixtureDef.density = 1.0f;
b2Fixture* myFixture = myBody->CreateFixture(&fixtureDef);
```

这将创建设备并将其附加到实体。您不需要存储fixture指针，因为当父实体被销毁时，fixture将自动销毁。可以在一个实体上创建多个装置。

可以销毁父实体上的固定装置。您可以这样做来建模一个易碎的对象。否则，您可以只留下固定装置，让尸体销毁负责销毁附着的固定装置。

```c++
myBody->DestroyFixture(myFixture);
```

## 密度

夹具密度用于计算父实体的质量特性。密度可以为零或正。一般来说，你应该在所有的固定装置上使用相似的密度。这将提高堆叠稳定性。

设置“密度”时，不会调整实体的质量。必须调用ResetMassData才能发生此情况。

```c++
b2Fixture* fixture;
fixture->SetDensity(5.0f);
b2Body*
body->ResetMassData();
```

## 摩擦

摩擦力用于使对象真实地沿着彼此滑动。Box2D支持静态和动态摩擦，但对两者使用相同的参数。在Box2D中精确模拟摩擦，摩擦强度与法向力成正比（称为库仑摩擦）。摩擦力参数通常设置在0和1之间，但可以是任何非负值。“摩擦力”（friction）值为0将禁用“摩擦力”（friction），值为1时，摩擦力将增强。计算两个形状之间的摩擦力时，Box2D必须组合两个父夹具的摩擦参数。使用几何平均值：

```c++
b2Fixture* fixtureA;
b2Fixture* fixtureB;
float friction;
friction = sqrtf(fixtureA->friction * fixtureB->friction);
```

所以，如果一个夹具的摩擦力为零，那么这个接触点的摩擦力就为零。

可以使用替代默认的混合摩擦力`b2Contact::SetFriction`. 通常在`b2ContactListener`回拨

## 归还

恢复用于使对象反弹。恢复值通常设置为0到1之间。请考虑将球扔到桌上。值为零表示球不会反弹。这称为非弹性碰撞。值为1意味着球的速度将被精确地反映出来。这叫做完全弹性碰撞。恢复使用以下公式组合。

```c++
b2Fixture* fixtureA;
b2Fixture* fixtureB;
float restitution;
restitution = b2Max(fixtureA->restitution, fixtureB->restitution);
```

恢复是以这种方式组合在一起的，这样你就可以在没有弹性地板的情况下拥有一个有弹性的超级球。

可以使用重写默认的混合恢复`b2Contact::SetRestitution`. 通常在[ b2ContactListener](https://box2d.org/documentation/classb2_contact_listener.html)回拨

当一个形状发生多个接触时，恢复过程是近似模拟的。这是因为Box2D使用迭代解算器。当碰撞速度很小时，Box2D还使用非弹性碰撞。这样做是为了防止抖动。看到了吗`b2_velocityThreshold` .

## 过滤

碰撞过滤允许您防止设备之间的碰撞。例如，假设你塑造了一个骑自行车的角色。您希望自行车与地形碰撞，角色与地形碰撞，但不希望角色与自行车碰撞（因为它们必须重叠）。Box2D支持使用类别和组进行这种冲突过滤。

Box2D支持16个碰撞类别。对于每个设备，您可以指定它属于哪个类别。还可以指定此设备可能与哪些其他类别发生冲突。例如，你可以在一个多人游戏中指定所有玩家不会互相碰撞，怪物也不会互相碰撞，但是玩家和怪物应该碰撞。这是用掩蔽位完成的。例如：

```c++
b2FixtureDef playerFixtureDef, monsterFixtureDef;
playerFixtureDef.filter.categoryBits = 0x0002;
monsterFixtureDef.filter.categoryBits = 0x0004;
playerFixtureDef.filter.maskBits = 0x0004;
monsterFixtureDef.filter.maskBits = 0x0002;
```

以下是发生碰撞的规则：

```c++
uint16 catA = fixtureA.filter.categoryBits;
uint16 maskA = fixtureA.filter.maskBits;
uint16 catB = fixtureB.filter.categoryBits;
uint16 maskB = fixtureB.filter.maskBits;
 
if ((catA & maskB) != 0 && (catB & maskA) != 0)
{
    // fixtures can collide
}
```

使用碰撞组可以指定整数组索引。可以使具有相同组索引的所有装置始终碰撞（正索引）或从不碰撞（负索引）。组索引通常用于表示某种程度上相关的事物，例如自行车的零件。在下面的示例中，fixture1和fixture2始终冲突，但是fixture3和fixture4从不冲突。

```c++
fixture1Def.filter.groupIndex = 2;
fixture2Def.filter.groupIndex = 2;
fixture3Def.filter.groupIndex = -8;
fixture4Def.filter.groupIndex = -8;
```

根据类别和掩码位过滤不同组索引的夹具之间的冲突。换句话说，组过滤比类别过滤具有更高的优先级。

请注意，Box2D中会出现额外的碰撞过滤。下面是一个列表：

- 静态实体上的固定装置只能与动态实体碰撞。
- 运动实体上的固定装置只能与动态实体碰撞。
- 同一物体上的固定装置不会相互碰撞。
- 可以选择启用/禁用由关节连接的实体上的装置之间的碰撞。

有时，可能需要在创建设备后更改碰撞过滤。你可以设置[B2过滤器](https://box2d.org/documentation/structb2_filter.html)在现有夹具上使用[ b2Fixture:：GetFilterData](https://box2d.org/documentation/classb2_fixture.html#ad956250d9f684a407992ec178320127e)和[ B2Fixture:：设置筛选器数据](https://box2d.org/documentation/classb2_fixture.html#a2c5e0d12c174927a4ad550459be334ad). 请注意，在下一个时间步骤之前，更改筛选器数据不会添加或删除联系人（请参阅世界级）。

## 传感器

有时游戏逻辑需要知道何时两个夹具重叠，但不应该有碰撞响应。这是通过使用传感器来完成的。传感器是检测碰撞但不产生响应的装置。

您可以将任何设备标记为传感器。传感器可以是静态的、动态的或动态的。记住，每个身体可能有多个固定装置，并且可以有传感器和固体固定装置的任意组合。另外，传感器只在至少一个物体是动态的时才形成接触，所以你不会得到运动学对运动学，运动学对静态，静态对静态的接触。

传感器不会产生接触点。有两种方法可以获取传感器的状态：

1. `b2Contact::IsTouching`
2. `b2ContactListener::BeginContact`和`联系人列表：：EndContact`

# 关节

关节用于将实体约束到世界或彼此。游戏中的典型例子包括碎布玩偶、跷跷板和滑轮。关节可以用许多不同的方式组合在一起以产生有趣的运动。

有些关节提供限制，因此可以控制运动范围。有些接头提供的电机可用于以规定的速度驱动接头，直到超过规定的力/扭矩。

联合电机有多种用途。可以使用电动机通过指定与实际位置和所需位置之间的差成比例的关节速度来控制位置。也可以使用电动机来模拟关节摩擦力：将关节速度设置为零，并提供一个小的，但有意义的最大电机力/转矩。然后，电机将试图阻止接头移动，直到负载变得太强。

## 关节定义

每个关节类型都有一个定义，该定义源自[ b2JointDef公司](https://box2d.org/documentation/structb2_joint_def.html). 所有的关节都连接在两个不同的物体之间。一个物体可能静止不动。允许静态和/或运动实体之间的运动类型，但没有影响，并且需要一些处理时间。

可以为任何关节类型指定用户数据，还可以提供标志以防止附着的实体彼此碰撞。这实际上是默认行为，必须设置collizeconnected布尔值以允许到连接的实体之间发生碰撞。

许多运动类型定义要求您提供一些几何数据。接头通常由锚点定义。这些点固定在附着的实体中。Box2D要求在局部坐标中指定这些点。这样，即使当前实体变换违反了关节约束，也可以指定关节，这是保存和重新加载游戏时常见的情况。此外，某些关节定义需要知道实体之间的默认相对角度。这是正确约束旋转所必需的。

初始化几何数据可能很繁琐，因此许多关节都具有初始化函数，这些函数使用当前的实体变换来删除大部分工作。但是，这些初始化函数通常只能用于原型设计。产品代码应该直接定义几何图形。这将使关节行为更加健壮。

其余运动类型定义数据取决于运动类型。我们现在来报道这些。

## 联合工厂

关节是使用世界工厂方法创建和销毁的。这就引出了一个老问题：

> **注意安全**：不要尝试使用new或malloc在堆栈或堆上创建关节。必须使用的创建和销毁方法创建和销毁实体和关节[b2World公司](https://box2d.org/documentation/classb2_world.html)班级

下面是一个旋转关节的寿命示例：

```c++
b2World* myWorld;
b2RevoluteJointDef jointDef;
jointDef.bodyA = myBodyA;
jointDef.bodyB = myBodyB;
jointDef.anchorPoint = myBodyA->GetCenterPosition();
 
b2RevoluteJoint* joint = (b2RevoluteJoint*)myWorld->CreateJoint(&jointDef);
 
// ... do stuff ...
 
myWorld->DestroyJoint(joint);
joint = nullptr;
```

当你的指针被销毁后，最好将其作废。如果试图重用指针，这将使程序以可控方式崩溃。

关节的寿命并不简单。注意这个警告：

> **注意安全**：连接的实体被破坏时，关节被破坏。

这种预防措施并不总是必要的。你可以组织你的游戏引擎，这样关节总是在连接的身体之前被破坏。在这种情况下，您不需要实现侦听器类。有关详细信息，请参见隐式销毁部分。

## 使用关节

许多模拟会创建关节，并且在关节被破坏之前不会再次访问它们。但是，运动类型中包含许多有用的数据，可以用来创建丰富的模拟。

首先，可以从关节获取实体、锚点和用户数据。

```c++
b2Body* b2Joint::GetBodyA();
b2Body* b2Joint::GetBodyB();
b2Vec2 b2Joint::GetAnchorA();
b2Vec2 b2Joint::GetAnchorB();
void* b2Joint::GetUserData();
```

所有接头都有反作用力和扭矩。这是在锚定点处作用于主体2的反作用力。你可以使用反作用力来断开关节或触发其他游戏事件。这些函数可能会进行一些计算，因此如果不需要结果，请不要调用它们。

```c++
b2Vec2 b2Joint::GetReactionForce();
float b2Joint::GetReactionTorque();
```

## 定距接头

最简单的关节之一是距离关节，它说两个物体上两点之间的距离必须是恒定的。指定距离运动类型时，两个实体应该已经就位。然后在世界坐标系中指定两个定位点。第一个锚定点连接到实体1，第二个锚定点连接到实体2。这些点表示距离约束的长度。

![anchor-length](images/anchor-length.png)

以下是距离运动类型定义的示例。在这种情况下，我们决定让物体碰撞。

```c++
b2DistanceJointDef jointDef;
jointDef.Initialize(myBodyA, myBodyB, worldAnchorOnBodyA,
worldAnchorOnBodyB);
jointDef.collideConnected = true;
```

距离接头也可以制成软接头，如弹簧-阻尼器连接。请参阅测试床中的Web示例以了解其行为。

柔软度是通过调整定义中的两个常数来实现的：刚度和阻尼。直接设置这些值是不直观的，因为它们都是以牛顿为单位的。Box2D提供和API来计算频率和阻尼比的这些值。

```c++
void b2LinearStiffness(float& stiffness, float& damping,
    float frequencyHertz, float dampingRatio,
    const b2Body* bodyA, const b2Body* bodyB);
```

把频率想象成谐振子的频率（比如吉他弦）。频率以赫兹为单位。通常，频率应小于时间步长频率的一半。因此，如果使用的是60Hz的时间步长，则距离关节的频率应小于30Hz。原因与奈奎斯特频率有关。

阻尼比是无量纲的，通常介于0和1之间，但可以更大。在1时，阻尼是临界的（所有振荡应消失）。

```c++
float frequencyHz = 4.0f;
float dampingRatio = 0.5f;
b2LinearStiffness(jointDef.stiffness, jointDef.damping, frequencyHz, dampingRatio, jointDef.bodyA, jointDef.bodyB);
```

也可以为定距接头定义最小和最大长度。看到了吗`b2DistanceJointDef`了解详情

## 旋转铰

旋转关节迫使两个物体共用一个固定点，通常称为铰链点。旋转关节只有一个自由度：两个物体的相对旋转。这称为关节角。

![anchor-angle](images/anchor-angle.png)

要指定旋转，需要在世界空间中提供两个实体和一个定位点。初始化函数假定实体已处于正确位置。

在这个例子中，两个物体通过第一个物体质心处的旋转关节连接起来。

```c++
b2RevoluteJointDef jointDef;
jointDef.Initialize(myBodyA, myBodyB, myBodyA->GetWorldCenter());
```

当车身B围绕角度点逆时针旋转时，旋转关节角度为正。与Box2D中的所有角度一样，旋转角度以弧度度量。根据惯例，使用Initialize（）创建关节时，旋转关节角度为零，而不管两个实体当前的旋转方向如何。

在某些情况下，您可能希望控制关节角度。为此，旋转关节可以选择性地模拟关节限制和/或马达。

关节限制强制关节角度保持在下限和上限之间。该限制将施加尽可能多的扭矩来实现这一点。限制范围应包括零，否则在模拟开始时关节将倾斜。

关节马达允许您指定关节速度（角度的时间导数）。速度可以是负的也可以是正的。电动机可以有无穷大的力，但这通常是不可取的。回想一个永恒的问题：

> *当不可抗拒的力量遇到不可移动的物体时会发生什么？*

我可以告诉你这不漂亮。所以你可以为关节马达提供最大扭矩。除非所需扭矩超过规定的最大值，否则万向节电机将保持规定的速度。当超过最大扭矩时，接头将减速，甚至可以反转。

可以使用关节马达来模拟关节摩擦力。只需将关节速度设置为零，并将最大扭矩设置为某个较小但有意义的值。电机将试图防止接头旋转，但会屈服于较大的负载。

这里是对上述旋转关节定义的修正；这一次关节有一个限制和一个马达启用。电机设置为模拟关节摩擦。

```c++
b2RevoluteJointDef jointDef;
jointDef.Initialize(bodyA, bodyB, myBodyA->GetWorldCenter());
jointDef.lowerAngle = -0.5f * b2_pi; // -90 degrees
jointDef.upperAngle = 0.25f * b2_pi; // 45 degrees
jointDef.enableLimit = true;
jointDef.maxMotorTorque = 10.0f;
jointDef.motorSpeed = 0.0f;
jointDef.enableMotor = true;
```

可以访问旋转关节的角度、速度和电机转矩。

```c++
float b2RevoluteJoint::GetJointAngle() const;
float b2RevoluteJoint::GetJointSpeed() const;
float b2RevoluteJoint::GetMotorTorque() const;
```

还可以更新每个步骤的电机参数。

```c++
void b2RevoluteJoint::SetMotorSpeed(float speed);
void b2RevoluteJoint::SetMaxMotorTorque(float torque);
```

联合发动机有一些有趣的能力。您可以在每个时间步长更新关节速度，以便可以使关节像正弦波一样来回移动，或者根据您想要的任何函数。

```c++
// ... Game Loop Begin ...
 
myJoint->SetMotorSpeed(cosf(0.5f * time));
 
// ... Game Loop End ...
```

也可以使用关节马达来跟踪所需的关节角度。例如：

```c++
// ... Game Loop Begin ...
 
float angleError = myJoint->GetJointAngle() - angleTarget;
float gain = 0.1f;
myJoint->SetMotorSpeed(-gain * angleError);
 
// ... Game Loop End ...
```

一般来说，增益参数不应该太大。否则你的关节可能会变得不稳定。

## 棱柱状节理

棱柱关节允许两个物体沿指定的轴相对平移。棱柱形关节可防止相对旋转。因此，棱柱运动类型具有单自由度。

![anchor-translation](images/anchor-translation.png)

棱柱关节的定义与旋转关节的描述相似；用平动代替角度，用力代替力矩。使用此类比，提供了一个带有连接限制和摩擦电机的棱柱形连接定义示例：

```c++
b2PrismaticJointDef jointDef;
b2Vec2 worldAxis(1.0f, 0.0f);
jointDef.Initialize(myBodyA, myBodyB, myBodyA->GetWorldCenter(), worldAxis);
jointDef.lowerTranslation = -5.0f;
jointDef.upperTranslation = 2.5f;
jointDef.enableLimit = true;
jointDef.maxMotorForce = 1.0f;
jointDef.motorSpeed = 0.0f;
jointDef.enableMotor = true;
```

旋转关节有一个隐式轴从屏幕出来。棱柱关节需要一个与屏幕平行的显式轴。这个轴固定在两个物体上，并跟随它们的运动。

与旋转关节一样，使用Initialize（）创建关节时，棱柱关节平移为零。所以要确保零在你的翻译上限和下限之间。

使用棱柱运动类型与使用旋转运动类型类似。以下是相关的成员职能：

```c++
float PrismaticJoint::GetJointTranslation() const;
float PrismaticJoint::GetJointSpeed() const;
float PrismaticJoint::GetMotorForce() const;
void PrismaticJoint::SetMotorSpeed(float speed);
void PrismaticJoint::SetMotorForce(float force);
```

## 滑轮接头

滑轮用于创建理想化的滑轮。滑轮把两个物体连在一起。一个身体向上，另一个身体向下。滑轮绳的总长度根据初始配置保持不变。

```c++
length1 + length2 == constant
```

可以提供模拟块和滑车的比率。这会导致皮带轮的一侧比另一侧延伸得更快。同时，一侧的约束力比另一侧小。你可以用它来创建机械杠杆。

```c++
length1 + ratio * length2 == constant
```

例如，如果比率为2，则length1将以length2速率的两倍变化。此外，连接到车身1的绳索中的力将是连接到车身2的绳索的一半的约束力。

![anchor-ground](images/anchor-ground.png)

当一侧完全伸出时，皮带轮可能会很麻烦。另一边的绳子长度为零。此时约束方程变为奇异（bad）。您应该配置冲突形状以防止这种情况发生。

以下是皮带轮定义示例：

```c++
b2Vec2 anchor1 = myBody1->GetWorldCenter();
b2Vec2 anchor2 = myBody2->GetWorldCenter();
 
b2Vec2 groundAnchor1(p1.x, p1.y + 10.0f);
b2Vec2 groundAnchor2(p2.x, p2.y + 12.0f);
 
float ratio = 1.0f;
 
b2PulleyJointDef jointDef;
jointDef.Initialize(myBody1, myBody2, groundAnchor1, groundAnchor2, anchor1, anchor2, ratio);
```

滑轮接头提供当前长度。

```c++
float PulleyJoint::GetLengthA() const;
float PulleyJoint::GetLengthB() const;
```

## 齿轮接头

如果你想创造一个复杂的机械装置，你可能需要使用齿轮。原则上，您可以在Box2D中通过使用复合形状来建模齿轮齿来创建齿轮。这不是很有效率，可能是乏味的作者。你还得小心地把齿轮排成一行，这样牙齿才能顺利啮合。Box2D有一种更简单的创建齿轮的方法：齿轮联接。

![joint](images/joint.png)

齿轮接头只能连接旋转和/或棱柱形接头。

与皮带轮传动比一样，可以指定传动比。但是，在这种情况下，传动比可能为负值。还要记住，当一个关节是旋转关节（角）而另一个关节是棱柱形（平移），那么齿轮传动比将有长度单位或一个超长单位。

```
coordinate1 + ratio * coordinate2 == constant
```

这里有一个齿轮接头的例子。主体myBodyA和myBodyB是来自两个关节的任何实体，只要它们不是同一个实体。

```c++
b2GearJointDef jointDef;
jointDef.bodyA = myBodyA;
jointDef.bodyB = myBodyB;
jointDef.joint1 = myRevoluteJoint;
jointDef.joint2 = myPrismaticJoint;
jointDef.ratio = 2.0f * b2_pi / myLength;
```

注意，齿轮接头取决于另外两个接头。这造成了一种脆弱的局面。如果这些关节被删除，会发生什么？

> **注意安全**：始终在齿轮上的旋转/棱柱形接头之前删除齿轮接头。否则，由于齿轮联接中的孤立关节指针，代码将以糟糕的方式崩溃。在删除任何相关实体之前，还应删除齿轮联接。

## 鼠标关节

在试验台上使用鼠标关节来操纵身体。它试图将实体上的一个点推向光标的当前位置。旋转没有限制。

鼠标关节定义具有目标点、最大力、频率和阻尼比。目标点最初与实体的锚定点重合。最大力用于防止多个动态物体相互作用时的剧烈反应。你可以把这个弄大点。频率和阻尼比用于创建类似于距离关节的弹簧/阻尼器效果。

许多用户都尝试过让鼠标关节适应游戏。用户往往希望实现精确定位和瞬时响应。鼠标关节在这种情况下工作不太好。您可能希望考虑使用运动学实体。

## 车轮接头

车轮接头将车身B上的一点限制为车身a上的一条直线。车轮接头还提供悬架弹簧。详见b2WheelJoint.h和Car.h。

![chelun](images/chelun.png)

## 焊接接头

焊接接头试图约束两个实体之间的所有相对运动。查看试验台上的悬臂梁。h，以了解焊接接头的行为。

使用焊接接头来定义易碎结构是很有诱惑力的。但是，Box2D解算器是迭代的，因此关节有点软。所以由焊接接头连接的身体链会弯曲。

相反，最好从具有多个固定装置的单个实体开始创建可破碎实体。当实体断裂时，可以销毁固定装置并在新实体上重新创建。请参阅测试床中的可破坏示例。

## 钢丝绳接头

绳接头限制两点之间的最大距离。这有助于防止身体链拉伸，即使在高负荷下。有关详细信息，请参见b2RopeJoint.h和rope_joint.cpp。

## 摩擦接头

摩擦接头用于自上而下的摩擦。关节提供二维平动摩擦力和角摩擦力。有关详细信息，请参见b2FrictionJoint.h和apply_force.cpp。

## 电动机接头

通过运动关节，可以通过指定目标位置和旋转偏移来控制实体的运动。您可以设置达到目标位置和旋转所需的最大电机力和扭矩。如果车身被阻挡，它将停止，接触力将与最大电机力和扭矩成比例。看到了吗[ b2MotorJoint公司](https://box2d.org/documentation/classb2_motor_joint.html)有关详细信息，请参阅motor\u joint.cpp。

## 车轮接头

车轮接头是专门为车辆设计的。它提供平移和旋转。翻译有一个弹簧和减震器来模拟车辆悬架。旋转使车轮旋转。可以指定旋转电机来驱动车轮并应用制动。看到了吗[B2车轮接头](https://box2d.org/documentation/classb2_wheel_joint.html)，wheel_joint.cpp和car.cpp以获取详细信息。

# 联络

触点是由Box2D创建的对象，用于管理两个装置之间的冲突。如果设备有子设备，例如链状形状，则每个相关子设备都存在一个接触。有不同种类的接触[B2联系人](https://box2d.org/documentation/classb2_contact.html)，用于管理不同类型夹具之间的接触。例如，有一个contact类用于管理多边形多边形碰撞，另一个contact类用于管理圆-圆碰撞。

以下是一些与联系人相关的术语。

#### 接触点

接触点是两个形状接触的点。Box2D近似于与少量点的接触。

#### 接触法向

接触法线是从一个形状指向另一个形状的单位向量。按照惯例，从fixtureA到fixtureB的法线点。

#### 触点分离

分离与渗透相反。当形状重叠时，分离为负。Box2D的未来版本可能会创建具有正分隔的接触点，因此您可能需要在报告接触点时检查标志。

#### 接触歧管

两个凸多边形之间的接触可以产生最多2个接触点。这两个点使用相同的法线，因此它们被分组到一个接触流形中，这是一个连续接触区域的近似值。

#### 正常脉冲

法向力是施加在接触点上的力，以防止形状穿透。为了方便起见，Box2D使用脉冲。法向冲量就是法向力乘以时间步长。

#### 切向脉冲

切向力在接触点处产生，以模拟摩擦力。为了方便起见，这被存储为一个脉冲。

#### 联系人ID

Box2D尝试重新使用时间步的接触力结果作为下一时间步的初始猜测。Box2D使用联系人id跨时间点匹配联系人点。ID包含有助于区分一个接触点和另一个接触点的几何特征索引。

当两个夹具的AABBs重叠时，会创建接触。有时碰撞过滤会阻止创建接触。随着AABBs停止重叠，接触被破坏。

因此，您可能会发现，可能存在为不接触的装置（仅是它们的AABBs）创建的联系人。嗯，这是对的。这是个“鸡还是蛋”的问题。我们不知道我们是否需要一个接触对象，直到一个接触对象被创建来分析碰撞。如果形状不接触，我们可以立即删除联系人，或者我们可以等到AABBs停止重叠。Box2D采用后一种方法，因为它允许系统缓存信息以提高性能。

## 接触等级

如前所述，contact类由Box2D创建和销毁。联系人对象不是由用户创建的。但是，您可以访问contact类并与其交互。

您可以访问原始接触歧管：

```c++
b2Manifold* b2Contact::GetManifold();
const b2Manifold* b2Contact::GetManifold() const;
```

您可以修改流形，但这通常不受支持，用于高级用途。

有一个函数来获取`b2WorldManifold` :

```c++
void b2Contact::GetWorldManifold(b2WorldManifold* worldManifold) const;
```

这将使用实体的当前位置来计算接触点的世界位置。

传感器不会产生歧管，因此它们使用：

```c++
bool touching = sensorContact->IsTouching();
```

此功能也适用于非传感器。

你可以从联系人那里得到固定装置。从那些你可以得到尸体。

```c++
b2Fixture* fixtureA = myContact->GetFixtureA();
b2Body* bodyA = fixtureA->GetBody();
MyActor* actorA = (MyActor*)bodyA->GetUserData().pointer;
```

您可以禁用联系人。这只在[ b2ContactListener:：预解决](https://box2d.org/documentation/classb2_contact_listener.html#a416f85eb45a1099053402b15a19a7de0)事件，讨论如下

## 访问联系人

您可以通过多种方式访问联系人。你可以直接访问世界和身体结构的接触。您还可以实现一个联系人侦听器。

您可以迭代世界上的所有联系人：

```c++
for (b2Contact* c = myWorld->GetContactList(); c; c = c->GetNext())
{
    // process c
}
```

您还可以迭代实体上的所有联系人。这些是用接触边结构存储在图形中的。

```c++
for (b2ContactEdge* ce = myBody->GetContactList(); ce; ce = ce->next)
{
    b2Contact* c = ce->contact;
    // process c
}
```

您还可以使用下面介绍的联系人侦听器访问联系人。

> **注意安全**：关闭访问联系人[b2World公司](https://box2d.org/documentation/classb2_world.html)和[b2Body公司](https://box2d.org/documentation/classb2_body.html)可能会错过一些在时间步长中间出现的瞬时接触。使用[ b2ContactListener](https://box2d.org/documentation/classb2_contact_listener.html)以获得最准确的结果

## 联系听众

您可以通过实现[ b2ContactListener](https://box2d.org/documentation/classb2_contact_listener.html). 联系人侦听器支持几个事件：开始、结束、预求解和后求解。

```c++
class MyContactListener : public b2ContactListener
{
public:
 
void BeginContact(b2Contact* contact)
{ /* handle begin event */ }
 
void EndContact(b2Contact* contact)
{ /* handle end event */ }
 
void PreSolve(b2Contact* contact, const b2Manifold* oldManifold)
{ /* handle pre-solve event */ }
 
void PostSolve(b2Contact* contact, const b2ContactImpulse* impulse)
{ /* handle post-solve event */ }
};
```

> **注意安全**：不要保留对发送到的指针的引用[ b2ContactListener](https://box2d.org/documentation/classb2_contact_listener.html). 相反，将接触点数据的深度副本放入自己的缓冲区。下面的例子展示了一种方法。

在运行时，您可以创建侦听器的实例并将其注册到[ B2World:：SetContactListener](https://box2d.org/documentation/classb2_world.html#a614549967fb8a1584b61c11e2d553d42). 当world对象存在时，确保侦听器仍在作用域中。

### 开始接触事件

当两个固定装置开始重叠时称之为。这被称为传感器和非传感器。此事件只能在时间点内发生。

### 结束接触事件

当两个固定装置停止重叠时，称之为。这被称为传感器和非传感器。当一个物体被摧毁时，这可能被调用，所以这个事件可以发生在时间步之外。

### 预解决事件

这在碰撞检测之后，但在冲突解决之前调用。这使您有机会根据当前配置禁用联系人。例如，可以使用此回调并调用b2Contact:：SetEnabled（false）来实现一个单边平台。每次通过冲突处理都会重新启用该联系人，因此您需要在每个时间点禁用该联系人。由于连续的碰撞检测，预解决事件可能在每个接触的每个时间步触发多次。

```c++
void PreSolve(b2Contact* contact, const b2Manifold* oldManifold)
{
    b2WorldManifold worldManifold;
    contact->GetWorldManifold(&worldManifold);
    if (worldManifold.normal.y < -0.5f)
    {
        contact->SetEnabled(false);
    }
}
```

预解算事件也是确定碰撞点状态和接近速度的好地方。

```c++
void PreSolve(b2Contact* contact, const b2Manifold* oldManifold)
{
    b2WorldManifold worldManifold;
    contact->GetWorldManifold(&worldManifold);
 
    b2PointState state1[2], state2[2];
    b2GetPointStates(state1, state2, oldManifold, contact->GetManifold());
 
    if (state2[0] == b2_addState)
    {
        const b2Body* bodyA = contact->GetFixtureA()->GetBody();
        const b2Body* bodyB = contact->GetFixtureB()->GetBody();
        b2Vec2 point = worldManifold.points[0];
        b2Vec2 vA = bodyA->GetLinearVelocityFromWorldPoint(point);
        b2Vec2 vB = bodyB->GetLinearVelocityFromWorldPoint(point);
 
        float approachVelocity = b2Dot(vB -- vA, worldManifold.normal);
 
        if (approachVelocity > 1.0f)
        {
            MyPlayCollisionSound();
        }
    }
}
```

### 后解决事件

在“后求解”事件中，可以收集碰撞脉冲结果。如果您不关心脉冲，您可能应该只实现pre-solve事件。

在联系人回调中实现改变物理世界的游戏逻辑是很诱人的。例如，可能发生碰撞，该碰撞会应用损害并尝试销毁关联的角色及其刚体。但是，Box2D不允许您在回调中更改物理世界，因为您可能会破坏Box2D当前正在处理的对象，从而导致孤立指针。

处理接触点的推荐做法是缓冲您关心的所有联系人数据，并在时间点之后处理这些数据。您应始终在时间步后立即处理接触点；否则，其他一些客户端代码可能会改变物理世界，使联系人缓冲区失效。当你处理接触缓冲区时，你可以改变物理世界，但你仍然需要小心，你不要孤立的指针存储在接触点缓冲区。该测试台具有示例接触点处理，可以避免孤立指针。

CollisionProcessing测试中的这段代码展示了在处理接触缓冲区时如何处理孤立体。这是节选。请务必阅读列表中的注释。此代码假定所有接触点都已缓冲在b2ContactPoint数组m_点中。

```c++
// We are going to destroy some bodies according to contact
// points. We must buffer the bodies that should be destroyed
// because they may belong to multiple contact points.
const int32 k_maxNuke = 6;
b2Body* nuke[k_maxNuke];
int32 nukeCount = 0;
 
// Traverse the contact buffer. Destroy bodies that
// are touching heavier bodies.
for (int32 i = 0; i < m_pointCount; ++i)
{
    ContactPoint* point = m_points + i;
    b2Body* bodyA = point->fixtureA->GetBody();
    b2Body* bodyB = point->FixtureB->GetBody();
    float massA = bodyA->GetMass();
    float massB = bodyB->GetMass();
 
    if (massA > 0.0f && massB > 0.0f)
    {
        if (massB > massA)
        {
            nuke[nukeCount++] = bodyA;
        }
        else
        {
            nuke[nukeCount++] = bodyB;
        }
 
        if (nukeCount == k_maxNuke)
        {
            break;
        }
    }
}
 
// Sort the nuke array to group duplicates.
std::sort(nuke, nuke + nukeCount);
 
// Destroy the bodies, skipping duplicates.
int32 i = 0;
while (i < nukeCount)
{
    b2Body* b = nuke[i++];
    while (i < nukeCount && nuke[i] == b)
    {
        ++i;
    }
 
    m_world->DestroyBody(b);
}
```

## 接触过滤

通常在游戏中你不希望所有的物体都碰撞。例如，您可能希望创建一个只有特定字符才能通过的门。这叫做接触过滤，因为有些交互作用被过滤掉了。

Box2D允许您通过实现[ b2ContactFilter公司](https://box2d.org/documentation/classb2_contact_filter.html)班级。此类要求您实现一个ShouldCollide函数，该函数接收两个[B2形状](https://box2d.org/documentation/classb2_shape.html)指针。如果形状发生碰撞，则函数返回true。

ShouldCollide的默认实现使用第6章fixture中定义的b2FilterData。

```c++
bool b2ContactFilter::ShouldCollide(b2Fixture* fixtureA, b2Fixture* fixtureB)
{
    const b2Filter& filterA = fixtureA->GetFilterData();
    const b2Filter& filterB = fixtureB->GetFilterData();
 
    if (filterA.groupIndex == filterB.groupIndex && filterA.groupIndex != 0)
    {
        return filterA.groupIndex > 0;
    }
 
    bool collideA = (filterA.maskBits & filterB.categoryBits) != 0;
    bool collideB = (filterA.categoryBits & filterB.maskBits) != 0
    bool collide =  collideA && collideB;
    return collide;
}
```

在运行时，您可以创建联系人筛选器的实例并将其注册到[ b2World:：SetContactFilter](https://box2d.org/documentation/classb2_world.html#a85e6e1e911c7d6366f8c7d57a12b72ff). 当世界存在的时候，确保你的过滤器在范围内。

```c++
MyContactFilter filter;
world->SetContactFilter(&filter);
// filter remains in scope ...
```

# 世界

这个`b2World`类包含实体和关节。它管理模拟的所有方面，并允许异步查询（如AABB查询和光线投射）。您与Box2D的大部分交互都将与[b2World公司](https://box2d.org/documentation/classb2_world.html)对象

## 创造和毁灭世界

创造一个世界是相当简单的。你只需要提供一个重力向量和一个布尔值来指示身体是否可以睡觉。通常，您将使用“新建”和“删除”来创建和销毁一个世界。

```c++
b2World* myWorld = new b2World(gravity, doSleep);
 
// ... do stuff ...
 
delete myWorld;
```

## 利用世界

世界级包括制造和破坏身体和关节的工厂。这些工厂将在后面有关车身和接头的章节中讨论。还有一些与[b2World公司](https://box2d.org/documentation/classb2_world.html)我现在就来报道

## 模拟

世界级的汽车被用来驱动模拟。您可以指定时间点以及速度和位置迭代计数。例如：

```c++
float timeStep = 1.0f / 60.f;
int32 velocityIterations = 10;
int32 positionIterations = 8;
myWorld->Step(timeStep, velocityIterations, positionIterations);
```

时间点过后，你可以检查你的身体和关节，以获取信息。最有可能的是你会从身体上抓取位置，这样你就可以更新你的演员并渲染他们。你可以在游戏循环中的任何地方执行时间步，但是你应该知道事情的顺序。例如，如果要获取该帧中新实体的碰撞结果，则必须在时间点之前创建实体。

正如我在上面的HelloWorld教程中所讨论的，您应该使用固定的时间步长。通过使用较大的时间步长，您可以在低帧速率情况下提高性能。但一般来说，你应该使用不超过1/30秒的时间步长。1/60秒的时间步长通常可以提供高质量的模拟。

迭代计数控制约束解算器扫描世界上所有接触和关节的次数。迭代次数越多，模拟效果越好。但是不要用一个小的时间步长来换取大量的迭代次数。10赫兹和60赫兹的迭代次数比20赫兹要好得多。

踏步之后，你应该清除你施加在你身体上的任何力量。这是通过命令完成的`b2World::ClearForces`. 这样可以使用相同的力场执行多个子步骤。

```c++
myWorld->ClearForces();
```

## 探索世界

世界是身体、接触和关节的容器。您可以从世界各地抓取body、contact和joint列表并在它们上迭代。例如，这段代码唤醒了世界上所有的身体：

```c++
for (b2Body* b = myWorld->GetBodyList(); b; b = b->GetNext())
{
    b->SetAwake(true);
}
```

不幸的是，真正的程序可能更复杂。例如，以下代码被破坏：

```c++
for (b2Body* b = myWorld->GetBodyList(); b; b = b->GetNext())
{
    GameActor* myActor = (GameActor*)b->GetUserData().pointer;
    if (myActor->IsDead())
    {
        myWorld->DestroyBody(b); // ERROR: now GetNext returns garbage.
    }
}
```

在尸体被摧毁之前一切都会好起来的。一旦一个实体被销毁，它的下一个指针就失效了。所以打电话给`b2Body::GetNext()`将返回垃圾。解决方法是在销毁主体之前复制下一个指针。

```c++
b2Body* node = myWorld->GetBodyList();
while (node)
{
    b2Body* b = node;
    node = node->GetNext();
    
    GameActor* myActor = (GameActor*)b->GetUserData().pointer;
    if (myActor->IsDead())
    {
        myWorld->DestroyBody(b);
    }
}
```

这样可以安全地摧毁当前的身体。但是，您可能需要调用一个可能会摧毁多个实体的游戏函数。在这种情况下，你需要非常小心。解决方案是特定于应用程序的，但是为了方便起见，我将展示一种解决问题的方法。

```c++
b2Body* node = myWorld->GetBodyList();
while (node)
{
    b2Body* b = node;
    node = node->GetNext();
 
    GameActor* myActor = (GameActor*)b->GetUserData().pointer;
    if (myActor->IsDead())
    {
        bool otherBodiesDestroyed = GameCrazyBodyDestroyer(b);
        if (otherBodiesDestroyed)
        {
            node = myWorld->GetBodyList();
        }
    }
}
```

很明显，要想让它成功，GameCrazyBodyDestroyer必须诚实地说出它所摧毁的一切。

## AABB查询

有时需要确定区域中的所有形状。这个[b2World公司](https://box2d.org/documentation/classb2_world.html)类有一个使用宽阶段数据结构的快速日志（N）方法。在世界坐标系中提供AABB和[ b2QueryCallback公司](https://box2d.org/documentation/classb2_query_callback.html). 世界调用你的类，每个fixture的AABB与查询AABB重叠。返回true继续查询，否则返回false。例如，下面的代码查找所有可能与指定AABB相交的装置，并唤醒所有关联的实体。

```c++
class MyQueryCallback : public b2QueryCallback
{
public:
    bool ReportFixture(b2Fixture* fixture)
    {
        b2Body* body = fixture->GetBody();
        body->SetAwake(true);
 
        // Return true to continue the query.
        return true;
    }
};
 
// Elsewhere ...
MyQueryCallback callback;
b2AABB aabb;
 
aabb.lowerBound.Set(-1.0f, -1.0f);
aabb.upperBound.Set(1.0f, 1.0f);
myWorld->Query(&callback, aabb);
```

你不能对回调的顺序做任何假设。

## 光线投射

您可以使用光线投射来执行视线检查、射击等。您可以通过实现回调类并提供起点和终点来执行光线投射。世界级的每一个赛程都会被射线击中。回调函数提供了fixture、交点、单位法向量和沿光线的分数距离。你不能对回调的顺序做任何假设。

通过返回分数来控制光线投射的持续性。返回零值表示应终止光线投射。1的分数表示光线投射应该继续，就像没有发生命中一样。如果从参数列表返回分数，光线将被剪裁到当前交点。因此，您可以光线投射任何形状，光线投射所有形状，或通过返回适当的分数来光线投射最近的形状。

也可以返回-1的分数来过滤设备。然后，光线投射将继续进行，就像设备不存在一样。

下面是一个例子：

```c++
// This class captures the closest hit shape.
class MyRayCastCallback : public b2RayCastCallback
{
public:
    MyRayCastCallback()
    {
        m_fixture = NULL;
    }
 
    float ReportFixture(b2Fixture* fixture, const b2Vec2& point,
                        const b2Vec2& normal, float fraction)
    {
        m_fixture = fixture;
        m_point = point;
        m_normal = normal;
        m_fraction = fraction;
        return fraction;
    }
 
    b2Fixture* m_fixture;
    b2Vec2 m_point;
    b2Vec2 m_normal;
    float m_fraction;
};
 
// Elsewhere ...
MyRayCastCallback callback;
b2Vec2 point1(-1.0f, 0.0f);
b2Vec2 point2(3.0f, 1.0f);
myWorld->RayCast(&callback, point1, point2);
```

> **注意安全**：由于舍入错误，光线投射可以在静态环境中通过多边形之间的小裂缝。如果这在应用程序中不可接受，请尝试稍微重叠多边形。