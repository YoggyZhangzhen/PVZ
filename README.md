# 植物大战僵尸项目？篮球大战蔡徐坤还差不多！
史上最强MasterPiece，面向对象OOP的究极实战！
作者：张桢
13000字纯手敲！！！ 超级无敌细节文档，基于植物大战僵尸
实现了包括 图像化app图标

<img width="72" height="92" alt="image" src="https://github.com/user-attachments/assets/9cfa0656-e1dc-4fa3-829a-eb8898e0f68f" />
<img width="886" height="581" alt="image" src="https://github.com/user-attachments/assets/06df5ca8-ecad-41bc-9081-d33663cec049" />
面试官到这里是不是想给我开SP了，淡定哈
蔡徐坤植物，五条悟的大招虚式-茈，更是有定制化子弹特效，以及蔡徐坤“你干嘛”音效绑定
<img width="650" height="472" alt="image" src="https://github.com/user-attachments/assets/34be8d18-e88e-40b1-9d02-e31dbceda4af" />
<img width="642" height="491" alt="image" src="https://github.com/user-attachments/assets/9dc7a1ef-19cf-4464-969b-ad63c06d2daa" />
<img width="409" height="189" alt="image" src="https://github.com/user-attachments/assets/7bde50ad-927c-46d4-89d4-98e1483a8046" />

## 1.项目总体陈述 1

我们需要实现如下功能

1，图形界面：==游戏主界面==，==准备界面==

2，==放置植物==，==删除植物==，==生成僵尸==，==攻击==

3，==胜负判断==

4，==卡牌冷却==

5，==场景选择==（白天黑夜

6，==背景音乐==，碰撞，防止植物，收集阳光

7，==显示阳光槽，植物槽==

8，==九种植物==，==七种僵尸==

9，==特殊功能==，快速生成僵尸，增加阳光

10，文件操作（用户名读取，游戏最长时间读取），基本数据结构使用

## 2.类型实现逻辑 

<img width="663" height="481" alt="image" src="https://github.com/user-attachments/assets/8c049db9-f171-43c6-963c-ae63dff56b37" />


### 2.0zObject

<img width="618" height="219" alt="image" src="https://github.com/user-attachments/assets/bf0f51e7-8da2-458a-beac-e7b2e318caee" />


### 2.1maindialog类

继承自QDialog：

三个槽函数：草地场景，黑暗场景，返回上一级

每个场景启动前删除原来的scene对象，然后new一个当前场景的类型对象，草地和黑暗场景关联开始场景，开始场景可以通过两个按键的信号调用开始草地和黑暗场景，再scenecpp中发出这些信号

例子

```c++
void mainDialog::back()
{
    this->setFixedSize(800, 600);
    delete scene;
    scene = new zStartScene(this);
    connect(this->scene, SIGNAL(toLawn()), this, SLOT(startLawn()));
    connect(this->scene, SIGNAL(toDarkLawn()), this, SLOT(startDark()));
}
```



### 2.2zanim类

继承zobject，所以重写了act，不过不需要血量，有生存情况alive

动画有一个属性frame控制动画帧数

在这个头文件里面同时我们还实现了豌豆击中动画，火动画，僵尸死掉动画，烧死动画，土豆爆炸动画，僵尸头掉下动画，樱桃炸弹爆炸动画，撑杆跳僵尸死掉动画，新闻僵尸死掉动画，新闻僵尸头掉下动画，这些动画全部继承自zanim，每一个里面有一个Qmovie类型的指针，需要析构，这个指针实现

在每一个子类的act里面实现按照各自动画距离消失的帧数（设置this->alive

frame说明它几帧后消失

构造函数：

把QMovie绑定到QLabel（zAnim继承自QLabel)开启动画

面试题：==这个帧数是怎么确定的？====为什么要有这个帧==

防止重复播放动画，动画结束就设置alive为false，在每个scene里面会，然后每20ms发送一次超时信号调用onTimer函数，==其中进行act和对于已经不alive的对象的清除==

| **动画类型**             | **frame 值** | **持续时间**     |
| ------------------------ | ------------ | ---------------- |
| `zPeaHit`（豌豆命中）    | 2            | 很短，瞬间消失   |
| `zZombieDie`（僵尸死亡） | 50           | 僵尸倒地的过程   |
| `zBoom`（爆炸）          | 40           | 炸弹爆炸特效     |
| `zBurnDie`（燃烧死亡）   | 85           | 僵尸被烧死的时间 |

这些动画通常是 **短暂的特效**，比如 **子弹命中、爆炸、燃烧、僵尸死亡**，不会一直存在，所以需要 **一个方式控制动画的生命周期**。这里的 `frame` **就是用来控制动画存在时间的**

<img width="648" height="231" alt="image" src="https://github.com/user-attachments/assets/abc671e2-7e5c-49cb-a5cf-3c6ccec075b2" />
你看我多敬业呀！！！快开sp吧哥

我在PIL 查看帧数，发现是100ms一帧，所以我这个20ms一次act应该把frame乘以5

#### **`frame` 作用**

1. **控制动画存活时间**，防止动画无限播放。
2. **确保游戏不会创建太多无用动画，影响性能**。
3. **短动画快速消失，长动画缓慢播放后删除**。



### 2.3zbonus类

设计奖励类，有zSun和zSunFall两个子类

前者有level，speed，accelerate，x_speed(initialize 0)

后者有level和speed，两者都有一个鼠标pressevent。两个sun类都有一个QMovie指针指向一个Sun.gif动态图

#### 他们的act实现了什么

在act中，和zanim一样每次减去帧数，如果帧数小于0就标记死亡alive=false，如果this->y()<=this->level即没有到达目标终点，则如果是sun修改通过accelerate每一帧修改speed，然后用move函数改变sun对象的x轴y轴位置为

```c++
if (this->y() <= this->level)
{
    this->speed += this->accelerate;
    this->move(this->x() + this->x_speed, this->y() + this->speed);
}
```

#### 点击阳光鼠标事件

点击后alive标记成false，增加25点scene对象的sunPoint，播放获得阳光的音效

==这个scene属性是在继承的zObject里面的==

#### 其他注意点

他们也有一个frame，==这个frame的作用是什么呢？==

初始化750帧，15秒后消失动画

析构函数要记得释放QMovie成员对象anim

#### zbonus中用到的数学原理

zSun和zSunFall分别代表向日葵产生的和掉下来的阳光，后者匀速，前者因为要==先上升后下降==，我们设置了一个加速度

看看这有多巧妙！

```c++
zSun::zSun(QWidget* parent) : zBonus(parent)
{
    this->setGeometry(260, 80, 80, 80);  // ✅ 初始位置（固定）
    this->setMovie(anim);  // ✅ 绑定 GIF 动画
    anim->start();  // ✅ 播放动画
    this->show();  // ✅ 显示阳光

    this->speed = -(qrand() % 5 + 7);  // ✅ 速度为负数，先向上移动
    this->accelerate = 2;  // ✅ 加速度（影响下降）
    this->level = 200;  // ✅ 控制最终下落的 Y 轴高度
    this->x_speed = qrand() % 5 - 2;  // ✅ 随机 X 方向移动速度
    this->frame = 750;  // ✅ 控制阳光生命周期（750 帧 ≈ 25 秒）
}

```

#### *每一个向日葵产生的阳光相对位置不一样怎么办？

这里构造函数虽然初始化了位置，但是实际上在向日葵植物类的act函数中我们会按照向日葵的位置调整阳光的位置和阳光最高能达到的下降终点,包括这个level也是相对的

```` c++
void zSunFlower::act()
{
    if (this->TimerSun <= 0)  // ✅ 当冷却时间归零，生成阳光
    {
        this->TimerSun = this->TimerSun_max;  // ✅ 重置计时器（等待下次生成）
        
        zSun* sun = new zSun(scene);  // ✅ 生成阳光
        sun->setGeometry(this->x(), this->y() + 15 - (qrand() % 5), 80, 80);  // ✅ 让阳光在向日葵附近出现
        sun->level = this->y() + 40;  // ✅ 设定阳光的下降终点
        
        scene->Bonuses.append(sun);  // ✅ 把阳光添加到游戏场景
    }
    else
    {
        this->TimerSun--;  // ✅ 计时器递减
    }
}

````



#### 随机性的引入

向日葵产生的阳光x轴速度是随机在[-2,2]通过qrand实现，然后初始向上移动速度也是一个随机负数（向下为正方向）。

#### 下落的阳光随机性通过什么实现

这个初始位置的随机性，以及最终这个level的随机性得到一个随机的终点

```c++
zSunFall::zSunFall(QWidget* parent) : zBonus(parent)
{
    this->setGeometry(qrand() % 600 + 320, 0, 80, 80);  // ✅ 随机 X 轴生成阳光
    this->setMovie(anim);
    anim->start();
    this->show();
    this->speed = 2;  // ✅ 下落速度
    this->level = qrand() % 400 + 100;  // ✅ 随机终点高度
    this->frame = 750;  // ✅ 750 帧后消失
}

```

匀速下降

```c++
this->move(this->x(), this->y() + this->speed);  // ✅ 让阳光以 `speed` 匀速下降
```



### 2.4zcard类

所有植物卡片继承于zcard基类，每个类同样有一个QMovie对象

```c++
class zCard : public zObject
{
public:
    zCard(QWidget* parent = 0);
    ~zCard();
    int plantIndex;  // ✅ 代表该卡片对应的植物类型，用来给当前的植物放置geometry位置，没有实现游戏中的卡槽
    int sunPoint = 50;  // ✅ 该植物需要的阳光数量
    int frame_max = 1, frame = 1;  // ✅ 最大冷却时间和当前剩余冷却时间，一开始在构造函数中初始化都一样
    virtual void act();
    QWidget* front;//冷却进度条
    QWidget* back;//遮罩
    QLabel* frontText;//阳光需求文字
    void setIndex(int index);
    void transFront();
protected:
    void mousePressEvent(QMouseEvent* event);  // ✅ 处理鼠标点击
};
```

#### 可能的思路*（仅供参考）

##### **✅ 玩家点击植物卡片**

1. **触发 `mousePressEvent()`**
2. **检查阳光是否足够**
3. **检查卡片是否冷却**
4. **设置 `scene->currentCard`**
5. **播放选中音效**
6. **卡片“跟随鼠标”移动**

##### **✅ 玩家点击地面种植植物**

1. **`zScene::mousePressEvent()` 触发**
2. **检查 `scene->currentCard` 是否有效**
3. **消耗阳光**
4. **在 `Plants` 列表里添加新植物**
5. **重置 `currentCard`，让卡片回到原位**
6. **让 `frame` 归零，进入冷却**

#### raise函数

使当前对象显示在最上方

#### act函数：（front和back图层的控制

==冷却时间逻辑：实际上在卡片上面放了两个半透明黑色图层，一个关联阳光，放下面，一个关联冷却时间，放上面。冷却时间0，图层没了，阳光足够，下面这个图层没了==

一开始back和front都是半透明黑色

然后两个都会变化，一开始冷却时间是满的然后随着frame的减小，冷却条的高度降低，冷却条是front，对于back，如果阳光不够，back显示，够了的话back消失，==back和front是两个颜色图层==，在构造函数初始化成灰色。==每一次act调用transfront==，检测back是否要设置成透明以及front的动态变化情况，把==当前的card raise到上一个图层==

```c++
void zCard::transFront()
{
    front->setGeometry(0, 6, 100, 54 * this->frame / this->frame_max);  // ✅ 让 `front` 高度根据冷却时间调整
    if (scene->sunPoint >= this->sunPoint)
    {
        back->setGeometry(0, 0, 0, 0);  // ✅ 阳光足够，隐藏 `back`
    }
    else
    {
        back->setGeometry(0, 6, 100, 54);  // ✅ 阳光不足，卡片变灰
    }
}

```

front->setStyleSheet("background-color: rgba(0, 0, 0, 50%);");半透明黑色

```c++
void zCard::transFront()
{
    front->setGeometry(0, 6, 100, 54 * this->frame / this->frame_max);  // ✅ 让 `front` 高度根据冷却时间调整
}

```

#### 卡片位置的确定

plantindex可以确定每个植物卡的位置，其中在zscene.cpp中实现了对index的赋值，index是根据plantIndex赋值的，但是9对应0

```c++
void zCard::setIndex(int index)
{
    this->setGeometry(125, 40 + 60 * index, 100, 60);
}

```

<img width="286" height="198" alt="image" src="https://github.com/user-attachments/assets/19dc389a-6544-40ea-ad0b-689200b39d2d" />

#### 父类构造和父子类析构

父类析构释放back，front，frontText

然后在每一个植物卡片子类里面析构对应的anim（QMovie对象

构造函数

```c++
zCard::zCard(QWidget* parent) : zObject(parent)
{
    this->setCursor(Qt::PointingHandCursor);
    back = new QWidget(this);
    back->setStyleSheet("background-color: rgba(0, 0, 0, 50%);");
    back->show();
    back->raise();
    front = new QWidget(this);
    front->setStyleSheet("background-color: rgba(0, 0, 0, 50%);");
    front->show();
    front->raise();
    frontText = new QLabel(this);
    frontText->setText("50");
    frontText->setGeometry(60, 33, 40, 20);//相对每一个card的坐标
    frontText->setAlignment(Qt::AlignHCenter);
    frontText->setFont(QFont("Calibri", 11));
    frontText->show();
    frontText->raise();
    this->show();
    this->raise();
}
```

| **代码部分**                         | **作用**                           |
| ------------------------------------ | ---------------------------------- |
| `setCursor(Qt::PointingHandCursor);` | 让鼠标变成“手型”                   |
| `back`                               | **黑色遮罩**（阳光不足时覆盖卡片） |
| `front`                              | **冷却条**（种植后逐渐消失）       |
| `frontText`                          | **显示阳光需求**                   |
| `this->show(); this->raise();`       | **确保 UI 正确显示**               |

==在这个widget里面鼠标变成点击==，然后对文字等进行初始化，相对位置处理

| **代码**              | **作用**                      | **防止被谁挡住** |
| --------------------- | ----------------------------- | ---------------- |
| `back->raise();`      | 确保阳光不足时，卡片变灰      | `frontText`      |
| `front->raise();`     | 确保冷却条在卡片上方          | `back`           |
| `frontText->raise();` | 确保阳光需求文字始终可见      | `front`          |
| `this->raise();`      | 确保整个 `zCard` 在 UI 最上方 | 其他 UI 组件     |

#### 子类构造

```c++
zSunFlowerCard::zSunFlowerCard(QWidget* parent) : zCard(parent)
{
    this->setMovie(anim);
    anim->start();
    this->frontText->setText("50");
    this->frame_max = 100;
    this->frame = 100;
    this->plantIndex = 1;
    this->sunPoint = 50;
}
```

每一个子类构造类似此种形式，播放其动画，设置相关的价格，以及冷却时间和当前冷却时间还要植物id，注意，铲子也是一个卡片类，它没有冷却时间哦，==所以shovel的back一直不存在，冷却条front也一直不存在。==

#### 种植物1：鼠标触发事件（放置植物逻辑的一部分

判断卡片是不是空，移动到现在的位置

如果点击了左键，如果冷却条front存在或者阳光不足播放不足阳光音乐，否则就是满足，播放放置的音乐，记录卡片位置，选中当前卡片。

#### 种植物2：*******难点：悬停bug（这tm超级重要

```c++
if (scene->currentCard != nullptr)
{
//        scene->currentCard->move(scene->currentPos);
}
```

总结这一行作用：

1，==用户在已经点击卡片的情况下点击其他的==，原来的卡片不会回到原来位置（我们需要它回去

2，选择卡片后，卡片会悬停在新的位置

<img width="532" height="543" alt="image" src="https://github.com/user-attachments/assets/3d701edf-3a19-413c-832c-74be71496f24" />

| **步骤**                    | **描述**                                                     |
| --------------------------- | ------------------------------------------------------------ |
| **1. 玩家点击左键**         | 触发 `mousePressEvent`，检查是否选中了卡片，是否可以放置植物。 |
| **2. 检查是否可以放置植物** | 判断目标格子是否已经有植物，并检查阳光是否足够。             |
| **3. 放置植物**             | 通过 `putPlant(m_cell)` 创建植物并放置到目标位置。           |
| **4. 更新植物信息**         | 设置植物的位置、记录行列号、添加到 `Plants` 列表、扣除阳光、更新冷却时间。 |
| **5. 清除当前卡片**         | 放置植物后，清空 `currentCard`，准备选择下一个卡片。         |

```c++
void zCard::mousePressEvent(QMouseEvent* event)
{
    if (scene->currentCard != nullptr)
    {
        scene->currentCard->move(scene->currentPos);
    }
    if (event->button() == Qt::LeftButton)
    {
        if (this->front->height() > 0)
        {
            QSound::play(":/Sounds/rc/NotEnoughSun.wav");
            scene->currentCard = nullptr;
            return;
        }
        if (this->scene->sunPoint < this->sunPoint)
        {
            QSound::play(":/Sounds/rc/NotEnoughSun.wav");
            scene->currentCard = nullptr;
            return;
        }
        QSound::play(":/Sounds/rc/Place.wav");
        scene->currentPos = this->pos();
        scene->currentCard = this;
    }
    else
    {
        scene->currentCard = nullptr;
    }
}
```

#### 种植物3：*******关联：怎么让卡片随着鼠标动，然后种植物

种植物逻辑在zscene中理解

##### 移动事件

```c++
void zScene::mouseMoveEvent(QMouseEvent* event)
{
    m = event->pos();
    if (this->currentCard != nullptr)
    {
        this->currentCard->move(m + QPoint(-40, 1));
    }
}
```

##### 点击事件

点击后种植物，得到格子的合法性后判断是否要进行putplant

``` c++
void zScene::mousePressEvent(QMouseEvent *event)
{
    QPoint m_cell = this->getCell();  // 获取鼠标点击位置的格子坐标
    if (event->button() == Qt::LeftButton)
    {
        if ((m_cell.x() > -1) && (this->currentCard != nullptr))  // 检查卡片有效性
        {
            for (int i = 0; i < Plants.count(); i++)
            {
                if ((Plants[i]->raw == m_cell.y()) && (Plants[i]->column == m_cell.x()) && this->currentCard->plantIndex > 0)
                {
                    QSound::play(":/Sounds/rc/NotEnoughSun.wav");  // 播放音效：如果卡片已经放置植物则取消操作
                    return;
                }
            }
            this->currentCard->move(this->currentPos);  // 将卡片恢复到原始位置
            this->putPlant(m_cell);  // 放置植物
            QSound::play(":/Sounds/rc/Place.wav");  // 播放音效：放置植物
        }
    }
    else
    {
        if (this->currentCard != nullptr)
        {
            this->currentCard->move(this->currentPos);  // 如果不是左键，恢复卡片位置
        }
        this->currentCard = nullptr;  // 清除当前卡片
    }
}

```

case0是铲子，表示删除植物，删除后把现在的卡片移动到原来位置然后，置空

##### 格子获取

这个qrect是一个矩形类，初始化需要四个参数

```c++
QRect(int x, int y, int width, int height);
```

- **`x, y`**：矩形的左上角坐标（即起始点的位置）。
- **`width, height`**：矩形的宽度和高度。

```c++
QPoint zScene::getCell()
{
    if (this->rect.contains(this->m))
    {
        return QPoint((this->m.x() - this->rect.left()) / this->cellSize.x(),
                      (this->m.y() - this->rect.top()) / this->cellSize.y());
    }
    else
    {
        return QPoint(-1, -1);
    }
}
```

##### 种植物

``` c++
void zScene::putPlant(QPoint t_cell)
{
    zPlant* plant;
    switch (this->currentCard->plantIndex)
    {
    case 0:
        for (int i = 0; i < Plants.count(); i++)
        {
            if ((Plants[i]->raw == t_cell.y()) && (Plants[i]->column == t_cell.x()))
            {
                delete Plants[i];
                Plants.removeAt(i);
                this->currentCard->move(this->currentPos);
                this->currentCard = nullptr;
                return;
            }
        }
        this->currentCard = nullptr;
        return;
        break;
    case 1:
        plant = new zSunFlower(this);
        break;
    case 2:
        plant = new zPeaShooter(this);
        break;
    case 3:
        plant = new zWallNut(this);
        break;
    case 4:
        plant = new zRepeater(this);
        break;
    case 5:
        plant = new zPotatoMine(this);
        break;
    case 6:
        plant = new zFireTree(this);
        break;
    case 7:
        plant = new zCherryBomb(this);
        break;
    case 8:
        plant = new zIcePeaShooter(this);
        break;
    case 9:
        plant = new zMushroom(this);
        break;
    }
    plant->setGeometry(this->rect.x() + 10 + this->cellSize.x() * t_cell.x(), this->rect.y() - 15 + this->cellSize.y() * t_cell.y(), 120, 100);
    plant->raw = t_cell.y();
    plant->column = t_cell.x();
    this->Plants.append(plant);
    this->sunPoint -= this->currentCard->sunPoint;
    this->currentCard->frame = this->currentCard->frame_max;
    this->currentCard = nullptr;
}

```

#### 总结种植物

就是 点击卡片后，卡片要随着鼠标动，如果这个时候点了其他的卡片，原来的那个卡片要回到原来的位置

这里有一个点击事件，和一个鼠标移动事件（原来让卡片跟着鼠标动）

然后第二次点击草地进行种植，获取当前鼠标点击的格子在草坪区间内的格子坐标，如果不在草坪内坐标就是-1，如果坐标不是-1，就可以进行植物种植

同时铲子也是一个卡片，如果点击铲子后再点击合法的草坪，就可以去掉当前植物，然后把铲子卡片复原到原来的位置

### 2.5zflyingobject类 

#### 类型特性

```c++
class zFlyingObject : public zObject
{
public:
    zFlyingObject(QWidget* parent = 0);
    virtual void act();
    int raw;
    bool canFire = false;
};
```

每一个子弹有行属性和能否变成火豌豆属性。因为火焰豌豆和mush是不能变的

总共有四种子弹，蘑菇，豌豆，火焰豆，冰豆

mush受到距离的限制才会发射，且会有一个生存周期用timerfly表示，其他子弹子类都只需要一个speed属性，同时扩展一个QMovie对象绑定动画

mush和其他类型不一样，除了speed还有一个timerfly作为生存周期，其他是通过screen的contain来判断要不要设置alive为false的

```c++
class zMush : public zFlyingObject
{
public:
    zMush(QWidget* parent = 0);
    ~zMush();
    void act();
private:
    QMovie* anim = new QMovie(":/FlyingObjects/rc/Mush.gif");
    int speed;
    int TimerFly;
};
```

#### 1.1版本更新：没有解决的bug：冰豌豆变火焰豆（未实现）

#### 1.2版本更新：没有解决的bug：僵尸断手（未实现）

#### 1.3版本更新：没有解决的bug：僵尸被冻结后的变色

#### 构造与析构

析构函数释放this->anim

构造函数设置动画播放，设置canFire属性，设置速度（蘑菇还要设置飞行生存时间）

```c++
zMush::zMush(QWidget* parent) : zFlyingObject(parent)
{
    this->setMovie(anim);
    anim->start();
    this->show();
    this->speed = 12;
    this->canFire = false;
    this->TimerFly = 22;
}
```

==行没有初始化，需要在植物类里面初始化==

#### act函数

```c++
void zMush::act()
{
    this->raise();
    this->TimerFly --;
    if (this->TimerFly < 0)
    {
        this->alive = false;
    }
    if (!(this->scene->screen.contains(this->pos())))
    {
        this->alive = false;
    }
    this->move(this->x() + this->speed , this->y());
    zZombie* zombie;
    foreach (zombie, this->scene->Zombies)
    {
        if ((qAbs(zombie->x() - this->x() + zombie->offset + 60) < 20) && ((this->raw) == (zombie->raw)) && (this->alive))
        {
            this->alive = false;
            zombie->hit(10);
            return;
        }
    }
}
```

蘑菇和其他子弹不同的是有一个生存周期，==除了scene对象的screen矩阵属性可以判断它是否生存以外，timerfly生存周期也可以，模拟蘑菇只能射出一段距离而不是全图==，其实我也想过用格子的手段实现，但是明显timerfly更简单。如果生存情况确定了，就在每帧移动这个子弹，然后在scene的僵尸列表里里判断每一个僵尸的位置是否有满足,豌豆集中后会有击中效果，这个动画效果会被加入scene的动画列表对象里，冰豌豆还会除法冰冻（没有实现图片，实现了速度减少，利用多态实现对于普通，路障，撑杆跳的速度降低

```c++
zAnim* pea_anim = new zPeaHit(scene);
pea_anim->setGeometry(this->x() + 20, this->y(), 40, 40);
```

这个动画设置在子弹的后面一点显示

僵尸中点位置减去子弹位置加上一些偏移量的绝对值满足在一个区间内，即到达了僵尸的前后两个边缘；同时子弹和僵尸在同一行且子弹alive，设置子弹为alive，调用该僵尸的hit函数然后return避免攻击到了多个僵尸

#### 注意这个hit，这里要设置true，因为群体伤害

```c++
void zFirePea::act()
{
    this->raise();
    if (!(this->scene->screen.contains(this->pos())))
    {
        this->alive = false;
    }
    this->move(this->x() + this->speed , this->y());
    zZombie* zombie;
    foreach (zombie, this->scene->Zombies)
    {
        if ((qAbs(zombie->x() - this->x() + zombie->offset + 60) < 20) && ((this->raw) == (zombie->raw)) && (this->alive))
        {
            this->alive = false;
            zAnim* pea_anim = new zFire(scene);
            pea_anim->setGeometry(this->x() + 20, this->y(), 40, 40);
            this->scene->Anims.append(pea_anim);
            zombie->hit(10);
            zZombie* zombie_2;
            foreach (zombie_2, this->scene->Zombies)
            {
                if ((qAbs(zombie_2->x() - this->x() + zombie_2->offset + 60) < 60) && ((this->raw) == (zombie_2->raw)))
                {
                    zombie_2->hit(10, true);
                }
            }
            return;
        }
    }
}
```

火焰豆还有群体伤害效果，在里面再次实现一个foreach循环，找到这个场景里的其他僵尸满足当前条件的，静音攻击，不用再播放特效了，所以放在里面，对于原来那个僵尸造成20点伤害x2，其他是10，范围扩大到60而不是20

#### 这个偏移量的作用？

offset用于对可能拿着东西的僵尸设置位置偏移，这会影响到攻击情况，比如失去东西offset就会减少，跳跃僵尸起跳后offset会调整，比如这里我就会变大，使得需要更近能打到，失去护甲也可以增加offset

在 `zCommonZombie` 的 `hit()` 方法中：

```c++
this->xpos += this->offset;
```

**作用：**

- 当某些僵尸失去护甲（如 **路障僵尸** 或 **铁桶僵尸**）时，它们的 `x` 坐标 **向右偏移 `offset` 像素**。
- 这样可以 **模拟僵尸失去护甲后变瘦**，使其在屏幕上位置有所调整，防止动画突兀。

#### 调用hit函数

```c++
void zPoleZombie::hit(int damage, bool silence)
{
    if (damage >= 200)
    {
        this->alive = false;
        zAnim* death_anim = new zBurnDie(scene);
        death_anim->setGeometry(this->x() - 20 + this->offset, this->y() + 25, 180, 150);
        this->scene->Anims.append(death_anim);
        return;
    }
    if (!(silence))
    {
        QSound::play(":/Sounds/rc/Pea.wav");
    }
    this->strength -= damage;
    if (this->strength <= 0)
    {
        this->alive = false;
        zAnim* death_anim = new zPoleZombieDie(scene);
        death_anim->setGeometry(this->x() - 30, this->y(), 300, 200);
        this->scene->Anims.append(death_anim);
        zAnim* death_head = new zPoleZombieHead(scene);
        death_head->setGeometry(this->x(), this->y() - 50, 300, 300);
        this->scene->Anims.append(death_head);
    }
}
```

如果僵尸死了，修改alive状态，设置一个掉头和一个死亡僵尸动画，脑袋的y轴下面一点，尸体的x轴左边一点

hit的silence有默认值，所以可以只传递一个参数，这个damage超过200的烧死特效对应樱桃炸弹和土豆雷，他们的伤害是1200



### 2.6zscene类（最终核心类

#### 类属性分析

```c++
class zScene : public zObject
{
    Q_OBJECT
public:
    explicit zScene(QWidget* parent = 0);
    ~zScene();
    QPoint getCell();

    QList<zZombie*> Zombies;
    QList<zPlant*> Plants;
    QList<zFlyingObject*> FlyingObjects;
    QList<zAnim*> Anims;
    QList<zBonus*> Bonuses;
    QList<zCard*> Cards;

    QPoint m;
    QPoint cellSize = QPoint(1, 1);
    QRect rect = QRect(0, 0, 1, 1);
    QRect screen = QRect(170, 0, 900, 600);

    QTimer* timer = nullptr;
    QSound* music = nullptr;
    QLabel* SunFront = new QLabel(this);
    QLabel* SunBack = new QLabel(this);
    QMovie* sunback = new QMovie(":/Interface/rc/SunBack.png");        //backgroud of value of sun



    void removeDeath();            //remove dead item
    void act();
    void createZombie();
    void judge();
    virtual void uiSetup();
    void putPlant(QPoint t_cell);
    void putZombie(int raw, int type);

    bool hasEnemy[6];
    int sunPoint = 50;
    int tempSunPoint;
    int threat = 0;
    int TimerLose = 0;
    zCard* currentCard = nullptr;
    QPoint currentPos;
protected:
    void mouseMoveEvent(QMouseEvent* event);
    void mousePressEvent(QMouseEvent* event);
signals:
    void toTitle();
    void toLawn();
    void toDarkLawn();
};
```

<img width="146" height="40" alt="image" src="https://github.com/user-attachments/assets/7a0ecb61-f373-49fc-a883-650d87b186dd" />

==sunback是这个动画，sunfront是这个数字，会变的==

移除死亡，act，创建僵尸，判定胜负函数，和ui初始化虚函数，放植物，放僵尸逻辑

QTimer计时器，关联一个槽函数对应onTimer，music是背景音乐，然后

```c++
int sunPoint = 50;
int tempSunPoint;
int threat = 0;
int TimerLose = 0;
```

timerLose的逻辑是如果僵尸到达家的坐标，设置成100，然后100帧后返回标题，这个威胁度关联僵尸的创造，

==当前卡片和当前点也可以设置一个属性，以便于在种植操作的时候去记录处理的点位置，====sunPoint是初始阳光值==

```c++
connect(this->scene, SIGNAL(toTitle()), this, SLOT(back()));
```

几个信号函数在miandialog中实现

```c++
QPoint m;
QPoint cellSize = QPoint(1, 1);
QRect rect = QRect(0, 0, 1, 1);
QRect screen = QRect(170, 0, 900, 600);
```

这些格子大小不是最后用到的，在两个游戏场景具体实现

##### 开始动画场景

```c++
class zStartScreen : public zScene
{
    Q_OBJECT
public:
    explicit zStartScreen(QWidget* parent = 0);
    ~zStartScreen();
private:
    QMovie* background = new QMovie(":/Interface/rc/StartScreen.jpg");
    QWidget* front = new QWidget(this);
    int frame = 100;
private slots:
    void onTimer();
};
```

多了个绑定的图片，然后有一个帧数，一个QWidget对象，背景是QMovie

这个front类似冷却条，实现明暗变化，就是用帧数关联这个图片的大小（灰度图片

##### 开始场景

```c++
class zStartScene : public zScene
{
    Q_OBJECT
public:
    explicit zStartScene(QWidget* parent = 0);
    ~zStartScene();
private:
    QMovie* background = new QMovie(":/Background/rc/Title.jpg");
    QMovie* lawn = new QMovie(":/Interface/rc/zombatar_background_crazydave.png");
    QMovie* dark = new QMovie(":/Interface/rc/zombatar_background_menu.png");
    QLabel* btn1 = new QLabel(this);
    QLabel* btn2 = new QLabel(this);
    QLabel* title = new QLabel(this);
    QLabel* UserName = new QLabel(this);
    QLabel* BestTime = new QLabel(this);
protected:
    void mousePressEvent(QMouseEvent *event);
};
```

两个按钮，两个QMovie，一个主体，用户名和最佳时间显示

##### 游戏场景

```c++
class zDarkScene : public zScene
{
    Q_OBJECT
public:
    explicit zDarkScene(QWidget* parent = 0);
    ~zDarkScene();
protected:
    void keyPressEvent(QKeyEvent *event);
private:
    QMovie* background = new QMovie(":/Background/rc/background2.jpg");
    QPushButton* exit = new QPushButton(this);
    QPoint cell;
    void uiSetup();
private slots:
    void onTimer();
    void leave();
};
```

两个游戏场景都是有一个键盘事件（快捷键释放僵尸），一个背景，然后有一个退出按钮，格子屬性，计时和离开槽函数

==有一个界面初始化函数，负责把卡片，推出按钮，阳光槽初始化==

#### scene功能实现

##### 构造函数

```c++
zScene::zScene(QWidget* parent) : zObject(parent)
{
    this->setMouseTracking(true);
    this->grabKeyboard();              //limit keyboard input
}
```

跟踪鼠标，键盘

```c++
while (!Zombies.empty())
{
    delete Zombies[0];                 //delete QList
    Zombies.removeAt(0);
}

if (!(this->SunFront == nullptr)) delete this->SunFront;
if (!(this->SunBack == nullptr)) delete this->SunBack;
if (!(this->sunback == nullptr))delete this->sunback;
if (!(this->timer == nullptr))delete this->timer;
```

析构函数释放对象delete加remove，然后释放指针，如阳光槽动画QLabel和对应的QMovie以及计时器指针

##### 获得格子

```c++
QPoint zScene::getCell()
{
    if (this->rect.contains(this->m))
    {
        return QPoint((this->m.x() - this->rect.left()) / this->cellSize.x(),
                      (this->m.y() - this->rect.top()) / this->cellSize.y());
    }
    else
    {
        return QPoint(-1, -1);
    }
}

```

m是在鼠标移动事件中更新的，这里获取了cell的点位,用于putplant

##### 清除死亡僵尸

```c++
int p = 0;
while (p < Plants.count())
{
    if (!(Plants[p]->alive))
    {
        delete (Plants[p]);
        Plants.removeAt(p);
    }
    else
    {
        p++;
    }
}
```

非常简单，查找列表中的生存状态，进行清除

但是对于僵尸类我们还要设置==hasEnemy值==，以便判断这行有没有僵尸,这里的数据在各个类型里面append了，不直接用行列判断，减少了复杂度

###### ***面试可以说：hasEnemy数组的意义！！！

更核心的，这个数组的意义在于判断植物是否要发射豆子，而攻击判定在豆子类上面而不是植物！！！！植物不能用距离判定

在ontimer里面先removedeath然后调用act，然后调用createZombie

##### act

很简单，调用所有对象的act

##### 创造僵尸

```c++
void zScene::createZombie()
{
    if (this->threat < 9001)
    {
        this->threat ++;
    }
    if (this->Zombies.count() < (this->threat / 600))
    {
        if (this->threat < 5000)
        {
            this->putZombie(qrand() % 5, 0);
        }
        else
        {
            this->putZombie(qrand() % 5, qrand() % 7);
        }
    }
}
```

创造僵尸函数实现了对威胁度的运算，有一个上限值是9000，场上僵尸的数量*600小于威胁度就创造僵尸，如果威胁度小于5000，只会在0-4行创造普通僵尸，否则随机一个僵尸种类

##### **判定胜负

```c++
void zScene::judge()
{
    if (this->TimerLose > 1)
    {
        this->TimerLose --;
    }
    else
    {
        if (this->TimerLose == 1)
        {
            emit toTitle();
        }
        else
        {
            zZombie* zombie;
            foreach(zombie, Zombies)
            {
                if (zombie->x() + zombie->offset < 130)
                {
                    this->currentCard = nullptr;
                    while (!Cards.empty())
                    {
                        delete Cards[0];
                        Cards.removeAt(0);
                    }
                    this->SunBack->hide();
                    this->SunFront->hide();
                    this->move(0, 0);
                    QSound::play(":/Sounds/rc/Lose.wav");
                    this->TimerLose = 100;
                    return;
                }
            }
        }
    }
}
```

==一开始timerlose是0,如果有僵尸加上他的偏移量到达左边界，当前卡片指针置空，删除所有卡片类和阳光槽（模拟游戏效果），把画面移动到左边一点的位置，播放失败声音，然后逐渐减少timerlose，直到等于1的时候发送前往标题信号==

在maindialog中响应信号，调用back函数回到析构当前scene创建一个startscene

##### 放植物（逻辑见上

注意里面剪掉阳光，卡片冷却时间恢复，当前卡片指针置空，卡片位置恢复，==植物的column可以在铲子铲植物的时候用到==

##### 放僵尸

比较简单，两个参数，行，类型，根据类型生成对应的僵尸，然后设置它的行属性和初始位置，在僵尸列表加上

```c++
void zScene::putZombie(int raw, int type)
{
    zZombie* zombie;
    switch(type)
    {
    case 0:
        zombie = new zCommonZombie(this, 0);
        break;
    case 1:
        zombie = new zCommonZombie(this, 1);
        break;
    case 2:
        zombie = new zCommonZombie(this, 2);
        break;
    case 3:
        zombie = new zCommonZombie(this, 3);
        break;
    case 4:
        zombie = new zCommonZombie(this, 4);
        break;
    case 5:
        zombie = new zPoleZombie(this);
        break;
    case 6:
        zombie = new zNewsZombie(this);
        break;
    }
    zombie->raw = raw;
    zombie->setGeometry(950, zombie->raw * 100 - 25 + qrand() % 5, 340, 200);
    this->Zombies.append(zombie);
}
```

##### 鼠标移动事件和鼠标点击事件（种植物

前者跟踪鼠标，后者获取当前格子然后如果点击的是左键，且格子合理，卡片非空，就可以进行

==如果当前位置行列有植物==，播放种植失败

否则还原卡片，种植植物，播放音乐

如果不是点击左键直接返回卡片位置，卡片指针置空

#### 战斗场景（白天，黑夜

格子数量5x9

```c++
zLawnScene::zLawnScene(QWidget* parent) : zScene(parent)
{
    this->setGeometry(-120, 0, 1400, 600);
    this->cellSize = QPoint(81, 100);
    this->rect = QRect(250, 85, 729, 500);
    this->setMovie(this->background);
    this->background->start();
    this->show();
    timer = new QTimer(this);
    connect(timer, SIGNAL(timeout()), this, SLOT(onTimer()));
    timer->start(20);
    this->uiSetup();
}
```

设置窗口位置（注意这个负数

战斗草坪，背景动画，==计时器用来触发onTimer==，设置ui界面

析构删除背景指针

按钮快捷键可以创造僵尸或者直接返回

| 代码                                          | 作用                                 |
| --------------------------------------------- | ------------------------------------ |
| `setGeometry(950, 0, 60, 60);`                | **设置位置和大小**（右上角，60x60）  |
| `setFlat(true);`                              | **去掉边框和背景**，让按钮更简洁     |
| `setIcon(QIcon(":/Interface/rc/Leave.png"));` | **设置按钮图标**（需要 `.qrc` 资源） |
| `setIconSize(QSize(60,60));`                  | **确保图标大小适配按钮**             |
| `setStyleSheet("background: transparent");`   | **去除按钮默认背景色**               |
| `setCursor(Qt::PointingHandCursor);`          | **鼠标悬停时变成手型**               |

💡 **最终效果**：
一个 **透明的 60x60 按钮**，只有 **`Leave.png` 图标**，鼠标悬停变手型，点击可触发退出操作。

==悬停变成点击手势是一个小知识点==

##### **向左偏移实现

==失败的时候move到0，0（0，0是相对widget，因此房子显示==

<img width="543" height="400" alt="image" src="https://github.com/user-attachments/assets/6baf3bd4-c0b7-4871-9cdb-aa2bb864261d" />

初始位置是-120，0，-120就是在widget的左边一点

<img width="591" height="395" alt="image" src="https://github.com/user-attachments/assets/30ae9508-e948-47e7-83ed-9b97e5435124" />

##### **僵尸数量没有变多bug

一开始一直没发现，后来发现threat里面的+=好像改成=了，

##### leave

leave由exit按钮触发，点击发送totitle信号

##### ==（无敌核心）onTimer逻辑==

```c++
void zLawnScene::onTimer()
{
    this->removeDeath();
    this->act();
    this->SunFront->setText(QString::number(this->sunPoint));
    this->createZombie();
    if (qrand() % 521 < 1)
    {
        zBonus* sun = new zSunFall(this);
        Bonuses.append(sun);
    }
//    this->exit->raise();
    this->judge();
}
```

==清除尸体，执行act，更新阳光，创造新僵尸，掉落阳光，判定输赢==

黑暗场景完全一样，就是改了豌豆变成个小蘑菇，不会掉落阳光

#### 开始场景

##### **设置选择场景的按钮**

```c++
btn1->setStyleSheet("QLabel{border: 5px solid #000000;} QLabel:hover{border:10px solid #EE0000;}");
btn2->setStyleSheet("QLabel{border: 5px solid #000000;} QLabel:hover{border:10px solid #EE0000;}");
```

- 给 `btn1` 和 `btn2` 按钮设置样式
  - 默认情况下，按钮有 **5px 黑色边框**。
  - **鼠标悬停时，边框变成 10px 红色**。
<img width="259" height="518" alt="image" src="https://github.com/user-attachments/assets/552424e9-e0fe-432e-81e5-e191fff5dcb3" />

```css
QLabel {border: 5px solid #000000;}
```

- 作用：为 `QLabel`（`btn1` 和 `btn2`）设置 **默认边框**

- 

  ```
  border: 5px solid #000000;
  ```

  - **边框宽度**：5 像素
  - **边框类型**：实线（`solid`）
  - **边框颜色**：黑色（`#000000`）



##### 文件操作和析构

```c++
QString user_name=string_list[0];
QString best_time=string_list[1];
QString str1="User: "+user_name;
QString str2="BestTime: "+best_time;
UserName->setGeometry(40, 430, 300, 100);
BestTime->setGeometry(40,460,300,100);
UserName->setText(str1);
BestTime->setText(str2);
UserName->setFont(QFont("Consolas", 14));
BestTime->setFont(QFont("Consolas", 14));
UserName->show();
BestTime->show();
file.close();

```

```c++
zStartScene::~zStartScene()
{
    delete this->background;
    delete this->lawn;
    delete this->dark;
    delete this->btn1;
    delete this->btn2;
    delete this->title;
}
```

##### 鼠标点击触发

判断event->pos是否包含在QRect内，如何emit跳转页面信号

#### 开始动画场景

##### 构造析构

```c++
zStartScreen::zStartScreen(QWidget* parent) : zScene(parent)
{
    this->setGeometry(0, 0, 800, 600);
    this->setMovie(this->background);
    this->background->start();
    this->show();
    this->front->setGeometry(0, 0, 800, 600);
    this->front->show();
    this->front->setStyleSheet("background: rgba(0,0,0,1)");
    timer = new QTimer(this);
    connect(timer, SIGNAL(timeout()), this, SLOT(onTimer()));
    timer->start(20);
}

```

设置两个页面，如何把front设置为灰色，设置QTimer对象，用timeout信号连接onTimer槽函数，周期也是20ms，析构delete这两个页面

对于这个灰度设置

**红色 (`R`)**：取值范围 `0 ~ 255`

**绿色 (`G`)**：取值范围 `0 ~ 255`

**蓝色 (`B`)**：取值范围 `0 ~ 255`

**透明度 (`A`)**：

- `0` → **完全透明**
- `1` → **完全不透明**
- `0.5` → **半透明**

##### Ontimer

```c++
void zStartScreen::onTimer()
{
    if (frame > 0)
    {
        frame --;
        if (frame > 50)
        {
            this->front->setStyleSheet("background: rgba(0,0,0," + QString::number((frame - 50) / 50.0) +")");
        }
        if (frame < 30)
        {
            this->front->setStyleSheet("background: rgba(0,0,0," + QString::number((30 - frame) / 30.0) +")");
        }
    }
    else
    {
        emit toTitle();
    }
}
```

透明度先越来越高，然后越来越低，怎么实现呢，一开始frame-50就是透明度，那么事件减到30肯定front变透明了

然后从0/30-30/30增高

这个和卡的front的区别，卡的front灰度没变，只是每次根据frame与framemax的比例调整front的高度，front逐渐往上减小，那么应该设置front为半透明

```c++
void zCard::transFront()
{
    front->setGeometry(0, 6, 100, 54 * this->frame / this->frame_max);
    if (scene->sunPoint >= this->sunPoint)
    {
        back->setGeometry(0, 0, 0, 0);
    }
    else
    {
        back->setGeometry(0, 6, 100, 54);
    }
}

```

### 2.7zzombie类 

#### 类属性分析

```c++
class zZombie : public zObject
{
    Q_OBJECT
public:
    zZombie(QWidget* parent = 0);
    virtual void act();
    virtual void hit(int damage, bool silence = false);
    virtual void ice();
    int raw;
    int offset = 0;
    int eatFrame = 0;
    bool iced = false;
    bool shield = false;
    float speed;
};
```

僵尸有被攻击函数，被冰冻函数，行，偏移量（可能拿盾牌），吃植物的冷却帧数，是否被冻，是否有护盾，速度

普通僵尸多出来的属性

```c++
class zCommonZombie : public zZombie
{
    Q_OBJECT
public:
    zCommonZombie(QWidget* parent = 0, int type = 0);
    ~zCommonZombie();
    void act();
    void hit(int damage, bool silence = false);
    void ice();
private:
    QMovie* walk = nullptr;
    QMovie* attack = nullptr;
    QMovie* prop_walk = nullptr;
    QMovie* prop_attack = nullptr;
    bool prop, iron;
    int prop_strength;
    float xpos;
};

```

走路攻击动画，以及拿着道具时候的走路攻击，然后是否有道具和铁门，道具血量，x轴位置，从最右边走过来（950，地图是900边界）（路障也算道具

```c++
this->move(this->xpos, this->y());
```

`move()` 的位置是相对于它的父控件的位置(就是widget)的。

```c++
class zPoleZombie : public zZombie
{
    Q_OBJECT
public:
    zPoleZombie(QWidget* parent = 0);
    ~zPoleZombie();
    void act();
    void hit(int damage, bool silence = false);
    void ice();
private:
    QMovie* walk = new QMovie(":/Zombies/rc/PoleZombieWalk.gif");
    QMovie* attack = new QMovie(":/Zombies/rc/PoleZombieAttack.gif");
    QMovie* run = new QMovie(":/Zombies/rc/PoleZombie.gif");
    QMovie* jump_1 = new QMovie(":/Zombies/rc/PoleZombieJump.gif");
    QMovie* jump_2 = new QMovie(":/Zombies/rc/PoleZombieJump2.gif");
    bool poled = true;
    bool jumping = false, jumping_1 = false;
    float xpos;
};
```

撑杆跳先生多了一个几个跳跃动画，和撑杆bool值，然后有两段跳跃布尔

报纸僵尸多了是否有纸，是否生气，纸的血量。

==记住了哦，撑杆是没有血量的==

#### 构造和析构函数

##### 普通僵尸构造

普通僵尸的构造函数要传入一个type，一开始可能生成两个类型的普通僵尸,设置攻击方式

```c++
switch (qrand() % 2)
{
case 0:
    this->walk = new QMovie(":/Zombies/rc/Zombie.gif");
    break;
case 1:
    this->walk = new QMovie(":/Zombies/rc/Zombie_2.gif");
    break;
}
this->attack = new QMovie(":/Zombies/rc/ZombieAttack.gif");
```

然后在此基础上根据type选择普通，旗帜，路障，铁栅栏

```c++
switch (type)
{
case 0:
    this->prop = false;
    this->iron = false;
    break;
case 1:
    this->prop = false;
    this->iron = false;
    this->offset = 20;
    delete this->walk;
    delete this->attack;
    this->walk = new QMovie(":/Zombies/rc/ZombieFlag.gif");
    this->attack = new QMovie(":/Zombies/rc/ZombieFlagAttack.gif");
    break;
case 2:
    this->prop = true;
    this->iron = false;
    this->prop_strength = 200;
    this->prop_walk = new QMovie(":/Zombies/rc/ZombieCone.gif");
    this->prop_attack = new QMovie(":/Zombies/rc/ZombieConeAttack.gif");
    this->offset = 20;
    break;
case 3:
    this->prop = true;
    this->iron = true;
    this->prop_strength = 400;
    this->prop_walk = new QMovie(":/Zombies/rc/ZombieBucket.gif");
    this->prop_attack = new QMovie(":/Zombies/rc/ZombieBucketAttack.gif");
    break;
case 4:
    this->prop = true;
    this->iron = true;
    this->shield = true;
    this->prop_strength = 400;
    this->prop_walk = new QMovie(":/Zombies/rc/ZombieShield.gif");
    this->prop_attack = new QMovie(":/Zombies/rc/ZombieShieldAttack.gif");
    break;
}
```

如果切换了，需要new一个新的走路和攻击，路障和铁桶的prop是true，铁栅栏全是true，然后设置prop的血量

```c++
this->speed = 0.25;
this->strength = 200;
this->xpos = 950;
if (this->prop)
{
    this->setMovie(this->prop_walk);
    this->walk->start();
    this->prop_walk->start();
    this->attack->start();
    this->prop_attack->start();
}
else
{
    this->setMovie(this->walk);
    this->walk->start();
    this->attack->start();
}
this->show();
```



速度，血量，x轴都一样，统一设置,上面是new一个动画对象，这里根据有没有障碍物设置动画启动，比较长的僵尸可以设置offset

##### 报纸和撑杆跳僵尸构造

很简单，把==走路和跑步，攻击QMovie对象启动==，设置速度，血量，x轴位置，然后设置offset大一点，==这里没有setMovie哦==

```c++
zPoleZombie::zPoleZombie(QWidget* parent) : zZombie(parent)
{
    this->setMovie(this->run);
    this->run->start();
    this->walk->start();
    this->attack->start();
    this->speed = 0.5;
    this->strength = 200;
    this->xpos = 950;
    this->show();
    this->offset = 145;
}
```

新闻僵尸也差不多，==拿报纸的走路攻击QMovie，不拿报纸的走路攻击，==速度血量，==多了个报纸血量==

##### **细节（面试可以说

==旗帜要删掉原来的对象，因为旗帜直接死亡，而其他对象会变回普通僵尸，所以其他的其实绑了四个动画==

##### **这可以装作是我后来的优化

==撑杆跳长度长，所以offset要大一点==

##### 僵尸析构（面试可以说

==普通僵尸析构只需要析构走路和攻击对象，因为如果有prop，prop会在死前被打掉==

撑杆跳僵尸就要析构全部状态，报纸僵尸也是

```c++
zPoleZombie::~zPoleZombie()
{
    delete this->walk;
    delete this->attack;
    delete this->run;
    delete this->jump_1;
    delete this->jump_2;
}
```

#### act函数

##### 普通僵尸act

==能吃优先吃植物，然后才考虑正常走路==

raise图片，遍历植物列表看看有没有能吃的（距离绝对值之差剪掉offset和某个参数小于某常数），且在同行，存活

如果吃植物帧小于等于0开吃放吃的音乐，否则就正常减少，如果有prop放prop_attack动画，没有就放正常的（==这就体现了prop_attack指针的好处==，植物受伤害是是和动画同频而不是声音，声音是20帧一次

如果不能攻击就正常走（有prop调用prop_walk动画，否则正常，然后xpos减去速度，移动当前位置

```c++
void zCommonZombie::act()
{
    this->raise();
    zPlant* plant;
    foreach (plant, this->scene->Plants)
    {
        if ((qAbs(plant->x() - this->x() - 55 - this->offset) < 40) && ((this->raw) == (plant->raw)) && (this->alive))
        {
            if (this->eatFrame <= 0)
            {
                QSound::play(":/Sounds/rc/Eat.wav");
                this->eatFrame = 20;
            }
            this->eatFrame --;
            if (this->prop)
            {
                this->setMovie(this->prop_attack);
            }
            else
            {
                this->setMovie(this->attack);
            }
            plant->hit(1);
            return;
        }
    }
    if (this->prop)
    {
        this->setMovie(this->prop_walk);
    }
    else
    {
        this->setMovie(this->walk);
    }
    this->xpos -= this->speed;
    this->move(this->xpos, this->y());
}
```

##### 撑杆跳act（倒逻辑

###### **跳跃逻辑

```c++
    if (this->jumping_1)
    {
        if (this->jump_2->currentFrameNumber() >= (this->jump_2->frameCount() - 1))
        {
            this->jump_2->stop();
            this->jumping_1 = false;
            this->speed /= 2;
        }
        return;
    }
    else if ((this->jumping) && (this->jump_1->currentFrameNumber() == (this->jump_1->frameCount() - 1)) && !(this->jumping_1))
    {
        this->jump_1->stop();
        this->setMovie(this->jump_2);
        this->jump_2->start();
        this->xpos -= 110;
        this->move(this->xpos, this->y());
        this->jumping_1 = true;
        this->jumping = false;
        return;
    }
    else if (this->jumping)
    {
        return;
    }
```

先要判断是不是跳跃状态进入，就是else if（==从下往上执行的==），

==**`this->jump_1->currentFrameNumber() == (this->jump_1->frameCount() - 1)`**: 如果第一次跳跃的动画已经播放到最后一帧，表示第一次跳跃已经完成，准备进行第二次跳跃。==(新知识点)

然后停止前面的动画，开启现在的，然后把x轴往前移动（因为跳过去了），move对象，设置jumping_1状态，以便进入上一个循环（二阶跳跃

==二阶跳跃后降低速度，然后才可以向下执行到动画，发现没有pole了，这个pole是在第一次触碰植物的时候变false的==

###### 吃逻辑

撑杆跳在吃的时候如果有杆子播放跳跃声音，然后调整跳跃相关的参数,去掉杆子，允许跳跃,播放jump1动画

```c++
foreach (plant, this->scene->Plants)
{
    if ((qAbs(plant->x() - this->x() - 55 - this->offset) < 40) && ((this->raw) == (plant->raw)) && (this->alive))
    {
        if (this->poled)
        {
            this->poled = false;
            this->jumping = true;
            this->setMovie(this->jump_1);
            this->jump_1->start();
            QSound::play(":/Sounds/rc/Pole.wav");
            return;
        }
        else
        {
            if (this->eatFrame <= 0)
            {
                QSound::play(":/Sounds/rc/Eat.wav");
                this->eatFrame = 20;
            }
            this->eatFrame --;
            this->setMovie(this->attack);
            plant->hit(1);
            return;
        }
    }
}
```

##### 报纸act

==报纸的angry由hit触发==

```c++
if (this->angrying)
{
    if (this->lose_paper->currentFrameNumber() >= (this->lose_paper->frameCount() - 1))
    {
        this->lose_paper->stop();
        this->angrying = false;
        this->speed *= 2;
        QSound::play(":/Sounds/rc/NewsLost.wav");
    }
    return;
}
```

生气了也有一个小情绪动画，播放到最后一帧停止，不生气了，速度变快，放个声音

```c++
foreach (plant, this->scene->Plants)
{
    if ((qAbs(plant->x() - this->x() - 55 - this->offset) < 40) && ((this->raw) == (plant->raw)) && (this->alive))
    {
        if (this->angrying)
        {
            return;
        }
        else
        {
            if (this->eatFrame <= 0)
            {
                QSound::play(":/Sounds/rc/Eat.wav");
                this->eatFrame = 27;
            }
            this->eatFrame --;
            if (this->paper)
            {
                this->setMovie(this->paper_attack);
                plant->hit(1);
            }
            else
            {
                this->setMovie(this->attack);
                plant->hit(2);
            }
            return;
        }
    }
}
```

如果距离足够在同一行且僵尸没死，生气了就返回放动画

没生气就开吃，27帧放个动画，比普通僵尸慢点，如果没有检测到paper报纸就攻击力变成2倍

然后根据有没有报纸播放动画，move位置

#### hit函数

```c++
void zCommonZombie::hit(int damage, bool silence)
{
    if (damage >= 200)
    {
        this->alive = false;
        zAnim* death_anim = new zBurnDie(scene);
        death_anim->setGeometry(this->x() - 30, this->y() + 25, 180, 150);
        this->scene->Anims.append(death_anim);
        if (!(this->prop_walk == nullptr))
        {
            delete this->prop_walk;
        }
        if (!(this->prop_attack == nullptr))
        {
            delete this->prop_attack;
        }
        return;
    }
    if (!(silence))
    {
        if ((this->prop) && (this->iron))
        {
            QSound::play(":/Sounds/rc/ShieldHit.wav");
        }
        else
        {
            QSound::play(":/Sounds/rc/Pea.wav");
        }
    }
    if (this->prop)
    {
        this->prop_strength -= damage;
        if (this->prop_strength <= 0)
        {
            this->prop = false;
            this->xpos += this->offset;
            this->act();
            this->setMovie(this->walk);
            this->walk->start();
            delete this->prop_walk;
            delete this->prop_attack;
            this->prop_walk = nullptr;
            this->prop_attack = nullptr;
            this->shield = false;
        }
    }
    else
    {
        this->strength -= damage;
    }
    if (this->strength <= 0)
    {
        this->alive = false;
        zAnim* death_anim = new zZombieDie(scene);
        death_anim->setGeometry(this->x() - 30, this->y() + 25, 180, 150);
        this->scene->Anims.append(death_anim);
        zAnim* death_head = new zZombieHead(scene);
        death_head->setGeometry(this->x() + 50, this->y() + 25, 180, 200);
        this->scene->Anims.append(death_head);
    }
}
```

死亡的时候普通僵尸逻辑才是最复杂的，如果收到200以上的伤害就是遇到炸弹了，直接死，然后把死亡动画放入Scene的list里面，因为这个动画是死掉之后放的，不会在act实现，所以在scene实现。如果有prop要把对应的prop指针释放，==因为有护具也会直接死==,，

然后如果没有silence就放音乐，铁门僵尸（有prop和iron）的声音和正常僵尸是两种

然后如果有护盾，减护盾血量，护盾没了之后要偏移一下x轴，同时调用act（这个时候也可以吃东西），然后设置正常走路动画，删除护具QMovie动画对象，并且指针置空，bool防御盾牌也设置为空（有可能本来就空，不打紧

如果没有护具就直接扣血，如果血量小于0就播放死亡动画和死掉的脑袋，逻辑如上



撑杆跳被打的没什么区别，被炸播放被烧死，被打死播放被打死的头和身体



新闻僵尸还有一个被打掉报纸逻辑，如果报纸被打掉了，bool angry变成true，然后paper变成false，播放lose_paper动画，然后这个动画结束后，==就执行act里面的逻辑，播放完动画加速，然后因为没有纸执行快走动画==

```c++
if (this->paper)
{
    this->setMovie(this->paper_walk);
}
else
{
    this->setMovie(this->walk);
}
```



#### ice函数

==普通僵尸如果没盾牌==，没被冰冻就被冻住，设置成被冻，然后速度减半，把所有动画速度减慢

```c++
void zCommonZombie::ice()
{
    if ((!iced) && (!shield))
    {
        this->iced = true;
        this->speed /= 2;
        if (!(this->walk == nullptr))
        {
            this->walk->setSpeed(50);
        }
        if (!(this->attack == nullptr))
        {
            this->attack->setSpeed(50);
        }
        if (!(this->prop_walk == nullptr))
        {
            this->prop_walk->setSpeed(50);
        }
        if (!(this->prop_attack == nullptr))
        {
            this->prop_attack->setSpeed(50);
        }
    }
}
```

撑杆跳僵尸和新闻僵尸因为没有中间的delete一键速度减半就好了

```c++
void zPoleZombie::ice()
{
    if (!iced)
    {
        this->iced = true;
        this->speed /= 2;
        this->walk->setSpeed(50);
        this->attack->setSpeed(50);
        this->jump_1->setSpeed(50);
        this->jump_2->setSpeed(50);
        this->run->setSpeed(50);
    }
}
```

### 2.8zplant类

#### 类属性分析 

```c++
class zPlant : public zObject
{
    Q_OBJECT
public:
    zPlant(QWidget* parent = 0);
    virtual void act();
    virtual void hit(int damage);
    int raw, column;
};
```

多了一个hit函数，表示被攻击的情况，多了行列属性

```c++
class zIcePeaShooter : public zPlant
{
    Q_OBJECT
public:
    zIcePeaShooter(QWidget* parent = 0);
    ~zIcePeaShooter();
    void act();
private:
    QMovie* anim = new QMovie(":/Plants/rc/IcePeaShooter.gif");
    int TimerShoot, TimerShoot_max;
};
```



射手们有最大射击冷却时间，和当前冷却时间，一开始都设置为一样的值。樱桃炸弹只有一个爆炸时间因为没有周期，向日葵也类似，产生阳光，土豆雷有一个生长时间，也没有周期，火炬树桩什么都没有，所有人都有一个anim（QMOVIE对象），然后土豆雷和坚果关联多个图像。

#### 构造函数和析构

析构一如既往的释放QMovie对象，土豆雷坚果这种就要释放多个

##### 射击型，太阳花

比如豌豆射手这种，一开始播放Qmovie然后设置血量和最大射击冷却时间，和当前冷却时间为同一个数字比如200，太阳花的当前冷却时间随机，不大于最大时间

```c++
this->TimerSun = qrand() % this->TimerSun_max;
```

```c++
zPeaShooter::zPeaShooter(QWidget *parent) : zPlant(parent)
{
    this->setMovie(anim);
    anim->start();
    this->show();
    this->TimerShoot_max = 50;
    this->TimerShoot = this->TimerShoot_max;
    this->strength = 200;
}
```

##### 坚果

对于坚果这样的，我们start三个QMovie对象，都是对于setMovie一开始只set第一个anim，血少了才会set其他的，

土豆雷也存在两个状态，生长时间到了才会set成另一个

火炬树桩就只需要血量，樱桃炸弹血量可以设置很大，然后设置一个爆炸初始时间

==这个画面切换的原理？==

`QLabel` 只会处理当前绑定的单一 `QMovie`。

#### hit实现

zPlant父类的hit中实现了一个统一的被攻击逻辑参数是damage，每次被攻击的时候，血量减去damage，比僵尸的简单没有状态变化，如果strength小于0就死亡

```c++
void zPlant::hit(int damage)
{
    this->strength -= damage;
    if (this->strength <= 0)
    {
        this->alive = false;
    }
}
```

hit就不需要重写了

#### act逻辑

##### 射手家族和向日葵

```c++
void zPeaShooter::act()
{
    if (this->TimerShoot <= 0)
    {
        if (!(scene->hasEnemy[this->raw]))
        {
            this->TimerShoot = qrand() % 20;// not to shoot too many peas in a single timer event
            return;
        }
        this->TimerShoot = this->TimerShoot_max;
        zPea* pea = new zPea(scene);
        pea->setGeometry(this->x() + 20, this->y() + 15 - (qrand() % 5), 80, 40);
        pea->raw = this->raw;
        scene->FlyingObjects.append(pea);
        QSound::play(":/Sounds/rc/PeaHit.wav");
    }
    else
    {
        this->TimerShoot --;
    }
}
```

冷却时间到了就开始射击，如果scene里的bool数组hasEnemy判断为没有敌人，随机设置一个比较短的冷却时间返回。防止没有敌人的时候射击

否则重新设置冷却时间，new一个子弹，这个new zPea(scene)本质是这个flyingObject是一个QObject，然后QObject是一个Qlabel，Qlabe有一个构造函数用来关联scene这个widget

然后设置这个子弹的初始位置在x轴和y轴的前面一点，设置它的大小，然后设置子弹的行属性，在场景飞行物列表加入子弹，播放音乐==（刚发射需要播放音乐，而不是关联飞行物，这是一个细节）==

==注意这里有一个对飞行物的行初始化，在飞行物的构造函数中我们不进行初始化==

###### 分析hasEnemy数组逻辑，对僵尸判定

```c++
void zScene::removeDeath()
{
    int p = 0;
    while (p < Plants.count())
    {
        if (!(Plants[p]->alive))
        {
            delete (Plants[p]);
            Plants.removeAt(p);
        }
        else
        {
            p++;
        }
    }
    for (int i = 0; i < 6; i++)
    {
        this->hasEnemy[i] = false;
    }
    p = 0;
    while (p < Zombies.count())
    {
        if (!(Zombies[p]->alive))
        {
            delete (Zombies[p]);
            Zombies.removeAt(p);
        }
        else
        {
            this->hasEnemy[Zombies[p]->raw] = true;
            p++;
        }
    }
    p = 0;
    while (p < FlyingObjects.count())
    {
        if (!(FlyingObjects[p]->alive))
        {
            delete (FlyingObjects[p]);
            FlyingObjects.removeAt(p);
        }
        else
        {
            p++;
        }
    }
    p = 0;
    while (p < Anims.count())
    {
        if (!(Anims[p]->alive))
        {
            delete (Anims[p]);
            Anims.removeAt(p);
        }
        else
        {
            p++;
        }
    }
    p = 0;
    while (p < Bonuses.count())
    {
        if (!(Bonuses[p]->alive))
        {
            delete (Bonuses[p]);
            Bonuses.removeAt(p);
        }
        else
        {
            p++;
        }
    }
}
```

这里hasEnemy每次设置为空，然后扫描存活的僵尸标记这一行的hasEnemy，然后在Scene的onTimer中调用

```c++
void zLawnScene::onTimer()
{
    this->removeDeath();
    this->act();
    this->SunFront->setText(QString::number(this->sunPoint));
    this->createZombie();
    if (qrand() % 521 < 1)
    {
        zBonus* sun = new zSunFall(this);
        Bonuses.append(sun);
    }
    this->exit->raise();
    this->judge();
}
```

###### 其他类豌豆射手的

向日葵不需要对僵尸的判定，生产出一个bonus（一个zSun对象），然后把sun设置在同样的x轴，y轴稍微可以偏移一下，==初始化一下level，because要先上升后下降有一个下降终点==然后加入bonus列表



蘑菇和冰豆逻辑完全一样和普通豌豆，但是蘑菇的y轴下面一点

###### **可以在面试里说这是遇到的问题：

双发豌豆在冷却时间为5的时候也发了一发子弹，然后==这次是不重置冷却时间的==

忘记减减就会有无敌连发豌豆了哈哈哈哈

<img width="646" height="434" alt="image" src="https://github.com/user-attachments/assets/eb8388f5-3284-49b1-be90-a4a0016f4dd3" />

```c++
void zRepeater::act()
{
    if (this->TimerShoot <= 0)
    {
        if (!(scene->hasEnemy[this->raw]))
        {
            this->TimerShoot = qrand() % 20;// not to shoot too many peas in a single timer event
            return;
        }
        this->TimerShoot = this->TimerShoot_max;
        zPea* pea = new zPea(scene);
        pea->setGeometry(this->x() + 20, this->y() + 15 - (qrand() % 5), 80, 40);
        pea->raw = this->raw;
        scene->FlyingObjects.append(pea);
        QSound::play(":/Sounds/rc/PeaHit.wav");
    }
    else if (this->TimerShoot == 5)
    {
        if (!(scene->hasEnemy[this->raw]))
        {
            this->TimerShoot = qrand() % 20;// not to shoot too many peas in a single timer event
            return;
        }
        zPea* pea = new zPea(scene);
        pea->setGeometry(this->x() + 20, this->y() + 15 - (qrand() % 5), 80, 40);
        pea->raw = this->raw;
        scene->FlyingObjects.append(pea);
        QSound::play(":/Sounds/rc/PeaHit.wav");
//        this->TimerShoot --;
    }
    else
    {
        this->TimerShoot --;
    }
}
```



这次timeshoot也要减少，不然死循环了

##### 坚果，土豆雷，樱桃炸弹，火炬树桩

坚果在每一次act的时候检查血量，从而set对应的QMovie，==死了就直接在hit函数里面析构了==

土豆雷成长时间到了就setMovie另一个anim1，然后每次act检测附件也没有僵尸（在僵尸列表里找有没有距离比较近的且在同一行，这样不需要x，y轴了），如果有且这个僵尸活着，就爆炸动画，然后爆炸

樱桃炸弹也一样，僵尸这个时间变成了爆炸时间，而且是群体伤害范围比较大，row的话附近三行都能炸，炸完设置一个动画对象放一个动画

==火炬树桩的这个判定其实类似豌豆，只是相当于攻击判定成了对飞行物的==，如果飞行物能火化，并且同行近距离，就设置一个火豆在原来位置，设置火豆的行和位置和原来的一样，在列表加入火豆，然后删掉原来的对象，==并且也要在列表里remove==！！！

###### **可以说在面试里说这是遇到的问题：为什么delete完还要再列表里remove？

```c++
void zFireTree::act()
{
    int i = 0;
    while (i < scene->FlyingObjects.count())
    {
        zFlyingObject* obj = scene->FlyingObjects[i];
        if (obj->canFire && (obj->raw == this->raw) && (qAbs(this->x() - obj->x() - 10) < 20))
        {
            zFirePea* firepea = new zFirePea(scene);
            firepea->setGeometry(obj->x(), obj->y(), 80, 40);
            firepea->raw = obj->raw;
            scene->FlyingObjects.append(firepea);
            delete obj;
            scene->FlyingObjects.removeAt(i);
        }
        else
        {
            i++;
        }
    }
}
```



<img width="646" height="428" alt="image" src="https://github.com/user-attachments/assets/107e7fe5-3d7f-4914-a86f-d9814949a61b" />

如果不remove会死机，因为里面那个对象没有删掉，访问空指针，段错误

## 3.相应类型逻辑思路

## 4.遇到的bug以及优化思路

### 4.1蔡徐坤版本优化

#### PVZ-BVC：1.1版本更新：坤坤僵尸朴素版本

<img width="621" height="412" alt="image" src="https://github.com/user-attachments/assets/f2ffba16-910d-473c-bb18-818db36cd329" />

#### PVZ-BVC：1.2版本更新：无敌豌豆

<img width="640" height="444" alt="image" src="https://github.com/user-attachments/assets/be36925a-e3ad-4932-9e52-900b4284d55a" />

#### PVZ-BVC:1.3版本更新：ico上线

<img width="108" height="118" alt="image" src="https://github.com/user-attachments/assets/3542003a-557e-4e88-b804-7349c89ae853" />

实现了图标，通过把jpg改成ico文件，本来我想改dll的发现好像不太行，后来发现ico文件也可以用

#### PVZ-BVC：1.4版本更新：增加虚式茈

樱桃炸弹的爆炸效果改了改，范围改了改，为以后坤坤大招做准备

![a90ffa9c77caf2b9a8038bdcaba1c92](C:\Users\20167\Documents\WeChat Files\wxid_hjp56vp9zq5j22\FileStorage\Temp\a90ffa9c77caf2b9a8038bdcaba1c92.png)

#### PVZ-BVC:1.45实现坐标变换：

见上

#### PVZ-BVC:1.49点击执行（初步

![image-20250209164427091](C:\Users\20167\AppData\Roaming\Typora\typora-user-images\image-20250209164427091.png)

#### PVZ-BVC: 2.0重大版本更新：

- 版本1.6更新：增加动画你干嘛(over)
- 版本1.7更新：增加卡片坤坤射手(over)
- 版本1.8更新：增加坤坤射手篮球子弹类(over)
- 版本1.9更新：增加坤坤射手爆炸篮球子弹类(over)
- 版本2.0更新：坤坤射手落地实现(over)
- 版本2.1更新：坤坤射手的大招实现（五条悟（over)
- 版本2.2更新：增加坤坤射手的世界末日大招音效（鸡你太美(over)

打算新增一个植物，所以我得学学Photoshopp图

思路，制作一个坤坤射手，或者简化我们可以把子弹变成篮球

需要更新坤坤射手类，篮球子弹，篮球子弹爆炸效果（可以用豌豆爆炸替代，也可以直接在这里生成一个樱桃炸弹

第一种思路：坤坤射手射出的是篮球子弹，设置一个新属性can_Fire2，坤坤射手的can_Fire2是true，经过火炬树桩变成火焰篮球

新增坤坤射手卡片，坤坤射手类，篮球子弹，火焰篮球子弹，你干嘛打击音效动画类

第二种思路：增加一个逻辑判断，如果是坤坤射手，在父类添加is_kunkun的bool，是坤坤那么火焰子弹变成爆炸篮球

（可以其他实现的，坤坤的大招：世界毁灭，播放一个动画，所有僵尸收到1200伤害团灭

坤坤射手思路：正常子弹是篮球，触发正常效果，音效我不知道能不能加（如果能就加上你干嘛音效），子弹经过火炬树桩变成燃烧篮球，燃烧篮球就是炸弹，效果和樱桃炸弹一样，僵尸死亡会被烧死

#### PVZ-BVC：2.3 终极版新增坤坤僵尸：

坤坤就是正常僵尸，数据正常吧

1. 内存优化：
代码中存在完整的内存管理逻辑，属于实打实的内存优化，核心证据：
生命周期管控：zScene析构函数中遍历QList容器（如plants/zombies/bullets），逐个delete实体并调用clear()，确保程序退出时释放所有动态分配的对象；
实时内存清理：zScene::removeDeath()方法会遍历所有实体，筛选出 “死亡状态” 的对象（isDeath()为 true），及时delete并从QList中移除，避免无效对象长期占用内存，杜绝内存泄漏；
指针安全管理：所有游戏实体均通过 “基类指针 + 容器存储” 统一管理，无野指针、重复释放等问题，符合 C++ 内存安全最佳实践。
2. 性能优化：
减少无效计算：碰撞检测（如子弹击中僵尸、樱桃炸弹范围伤害）仅针对 “存活实体 + 有效范围” 判断，避免遍历所有实体做无效检测；
帧循环精简：zScene::onTimer()驱动的帧循环中，仅对 “存活实体” 调用act()方法更新状态，死亡实体提前被removeDeath()清理，减少每帧计算量；

