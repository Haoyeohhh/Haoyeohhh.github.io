---
title: 在Unity中基于Verlet算法模拟绳索
description: >-
  在Unity中运用Verlet算法加上胡克定律模拟绳索...
date: 2024-09-24 03:00:00 +0800
categories: [开发, Unity]
tags: [Unity, 绳索模拟, Verlet integration, 泰勒展开, 胡克定律]
comments: true
math: true
---
![Desktop View](assets\img\MdAssert\开发\Unity\在Unity中基于Verlet算法模拟绳索\1.gif){: w="640" h="360" }
# 在Unity中基于Verlet算法模拟绳索

## 基本原理

### Verlet 算法模拟运动

#### Verlet 算法推导

设某一时刻的点的位置函数为

$$
y=r\left ( t \right )
$$

根据泰勒公式

$$
f\left ( x \right )=\sum_{n=0}^{\infty}\frac{f\left ( x_{0} \right )}{n!}\cdot \left (  x-x_{0}\right )^{n}
$$

将位置函数二阶泰勒展开为

$$
r\left ( t \right )=r\left ( t_{0} \right )+{r}'\left ( t_{0} \right )\cdot \left ( t-t_{0} \right )+\frac{\\{r}''\left ( t_{0} \right)\cdot \left ( t-t_{0} \right )^{2}}{2}+O\left ( t^{3} \right )
$$

令

$$
t=t_{0}+\Delta t
$$

则**①**式：

$$
\textcolor{blue}{r\left ( t_{0}+\Delta t \right )}=r\left ( t_{0} \right )+{r}'\left ( t_{0} \right )\cdot \Delta t+\frac{\\{r}''\left ( t_{0} \right)\cdot \left ( \Delta t \right )^{2}}{2}+O\left ( \Delta t^{3} \right )
$$

令

$$
t=t_{0}-Delta t
$$

则**②**式：

$$
\textcolor{red}{r\left ( t_{0}-\Delta t \right )}=r\left ( t_{0} \right )-{r}'\left ( t_{0} \right )\cdot \Delta t+\frac{\\{r}''\left ( t_{0} \right)\cdot \left ( \Delta t \right )^{2}}{2}+O\left ( \Delta t^{3} \right )
$$

**①+②**式得：

$$
\textcolor{blue}{r\left ( t_{0}+\Delta t \right )}=2r\left ( t_{0} \right )-\textcolor{red}{r\left ( t_{0}-\Delta t \right )}
+{r}''\left ( t_{0} \right)\cdot \left ( \Delta t \right )^{2}+O\left ( \Delta t^{3} \right )
$$

去掉泰勒余项进一步简化公式

$$
\textcolor{blue}{r\left ( t_{0}+\Delta t \right )}=2r\left ( t_{0} \right )-\textcolor{red}{r\left ( t_{0}-\Delta t \right )}
+{r}''\left ( t_{0} \right)\cdot \left ( \Delta t \right )^{2}
$$

处理$\textcolor{red}{\{r}'\'\left ( t_{0} \right)\cdot \left ( \Delta t \right )^{2}} $，一点点物理学小知识，位移对时间的一阶导数为此时刻的速度，速度对时间的一阶导数为此时刻的加速度

$$
{r}'\left ( t_{0} \right)=v\left(t_{0}\right)
$$

$$
v'\left(t_{0}\right)=a\left(t_{0}\right)
$$

即最后得到公式

$$
\textcolor{blue}{r\left ( t_{0}+\Delta t \right )}=2r\left ( t_{0} \right )-\textcolor{red}{r\left ( t_{0}-\Delta t \right )}
+{a}\left ( t_{0} \right)\cdot \left ( \Delta t \right )^{2}
$$

#### 公式说明

由此可得到物体在$ t_{0}-\Delta t$、$ t_{0}$ 、$ t_{0}+\Delta t$ 这三个时刻的位置关系，将$ \Delta t$看作帧间隔,即得出结论：在已知物体的**上一帧位置**、**此帧位置**、**物体的加速度** ，可以预测物体的**下一帧位置**。

### 重力模拟（$\textcolor{red}{a\left(t_{0}\right)}$项的处理）
在通常情况下，物体运动过程中所受合力为重力，即：我们忽略其他力，在只考虑重力的情况下。可将上式中的$\textcolor{red}{a\left(t_{0}\right)}$视为重力加速度。即可在物体运动模拟中添加重力的影响。

### 约束模拟
下面我们将**点**升维成**线**，即在点与点中考虑添加约束（应力）。
在上面的所有步骤中，我们完成对点运动和重力的模拟，因在这里只要考虑点与点之间**应力**对绳子内部各个点的影响即可。
笔者在这里初步设想：
将**两个点之间**看作一根**弹簧**，同时设定好约束长度。
- 当两点之间的距离**超过**约束长度时，同时为两点添加一个沿绳子方向**向内**的力

- 当两点之间的距离**小于**约束长度时，同时为两点添加一个沿绳子方向**向外**的力
  基于此，运用**胡克定律** $\textcolor{red}{F=k\cdot\Delta x}$,计算加速度$\textcolor{red}{a'}$

$$
\textcolor{red}{\\{a'}=\frac{k\cdot \Delta x}{m}}
$$

  (注意：这里的$\textcolor{red}{\Delta x}$是指在这个时刻，**当前点**与**连接点**的距离，与**Verlet算法**中同一点**上一时刻**与**下一时刻**的**位置变化量**不同)

下一步单独计算**应力的位移**，方向为**沿绳方向**

$$
x=a\cdot\left ( \Delta t \right )^{2}=\textcolor{red}{\frac{k\cdot \Delta x\cdot\left ( \Delta t \right )^{2}}{m}}
$$

除此之外，由于绳索上的点（除**开端**和**结尾**外）所受的应力有**两部分**，我们还需考虑**上一个点**对此点的应力和**下一个点**对此点的应力。

关于这一点，在编写程序中我们只需要创建一个**类**将他们储存起来即可。

由此，我们几乎完成了**模拟绳索**的所有步骤。

## 代码实现

### 创建类

首先创建一个**Particle**类，和一个**Stick**类，模拟**点**和两点之间**绳子**的约束

注意：通过设置Bool变量**isLock**来标记点是否被**模拟算法**影响

```c#
 //创建粒子类
    public class Particle
    {
        public Vector3 position;//当前位置
        public Vector3 oldPosition;//上一帧位置
        public bool isLock = false;//是否锁定
        public Particle(Vector3 a)
        {
            position = a;
            oldPosition = a;
        }
    }
    //创建约束
    public class Stick
    {
        public Particle particle_1;
        public Particle particle_2;
        public Stick(Particle a, Particle b)
        {
            particle_1 = a;
            particle_2 = b;
        }
    }
```

### 初始化系统

因为归根结底要将算法应用于Unity，所以我将绳子的起始和结束分别关联到一个物体当中，设置好分段数目，然后通过计算自动分配点的位置

```c#
public GameObject startParticle;//起始位置
public GameObject endParticle;//结束位置
[SerializeField] private int particleCount = 10;//创建的粒子数目
private float stickLength;//约束的长度
private List<Particle> particles = new List<Particle>();
private List<Stick> sticks = new List<Stick>();


//初始化
private void InitSystem()
        {
            stickLength = (startParticle.transform.position - endParticle.transform.position).magnitude / (particleCount - 1);
            //计算约束长度
            Vector3 dir = (startParticle.transform.position - endParticle.transform.position).normalized;//开始点与结尾点的方向向量
            for (int i = 0; i < particleCount; i++)
            {
                particles.Add(new Particle(startParticle.transform.position - dir * stickLength * i));
            }
            for (int i = 0; i < particles.Count - 1; i++)
            {
                sticks.Add(new Stick(particles[i], particles[i + 1]));
            }
        }
```

### 编写模拟算法
根据上边的结论：
#### **Verlet 算法模拟**

```c#

            //Verlet 算法模拟
            var verletDel = new Vector3();
            for (int i = 0; i < particles.Count; i++)
            {
                if (particles[i].isLock == false)
                {
                    Vector3 temp = particles[i].position;
                    verletDel = particles[i].position - particles[i].oldPosition + new Vector3(0, -gravity, 0) * Time.fixedDeltaTime * Time.fixedDeltaTime;//Verlet 算法算法下的位置增量
                    particles[i].position += verletDel;
                    particles[i].oldPosition = temp;//更新oldPosition
                }
            }
```

#### **约束模拟**

```c#
                //约束
                foreach (var k in sticks)
                {
                    Vector3 dir = (k.particle_2.position - k.particle_1.position).normalized;//绳子方向的单位向量
                    float curLength = (k.particle_2.position - k.particle_1.position).magnitude;//当前节点长度
                    float deltaLength = curLength - stickLength;
                    var xDelta = ki * deltaLength * Time.fixedDeltaTime * Time.fixedDeltaTime;
                    
                        if (!k.particle_1.isLock)
                        {
                            k.particle_1.position += xDelta * dir;
                        }
                        if (!k.particle_2.isLock)
                        {
                            k.particle_2.position -= xDelta * dir;
                        }
                }
```

但是此时的**约束模拟**算法经实际验证还存在一些问题

我们遍历整个**List<**Sticks**>**时，是从顶端到末尾依次遍历，然后计算得出点的位置，一个一个的移动。

而实际上，绳子上**每个点**的移动，都会影响除这个点外的**所有点**的位置。

具体一点来讲，就像一串珠子，单独对某一个特定的珠子来看，影响其位置的仅仅只有与其**相邻**的两个珠子

但是整体来看，当我们**拿起**某个珠子时，因其连锁反应，这一串上面所有珠子都被**拿起**了。

因此，假设4个相邻位置的点A、B、C、D，

第一步：我们通过点**A、C**的位置确定**B**点的位置

第二步：我们通过点**B、D**的位置确定**C**点的位置

此时，**C点**的位置是一定会移动的，这C点的**位置改变**，又会导致第一步中使用**未移动之前C点位置信息**计算得来**B点**的位置错误。

这样来看我们永远无法正确模拟。

其实，可以使用多次计算来近似估计点的位置更新信息，在迭代多次后，每个点的移动距离几乎可以忽略不记，此时可以近似当作绳子的约束模拟完成。

加入迭代后代码如下：

```c#
public float ki = 10;//劲度系数

for (int i = 0; i < lterationStep; i++)
            {
                foreach (var k in sticks)
                {
                    Vector3 dir = (k.particle_2.position - k.particle_1.position).normalized;//绳子方向的单位向量
                    float curLength = (k.particle_2.position - k.particle_1.position).magnitude;//当前节点长度
                    float deltaLength = curLength - stickLength;
                    var xDelta = ki * deltaLength * Time.fixedDeltaTime * Time.fixedDeltaTime;
                        if (!k.particle_1.isLock)
                        {
                            k.particle_1.position += xDelta * dir;
                        }
                        if (!k.particle_2.isLock)
                        {
                            k.particle_2.position -= xDelta * dir;
                        }
                }
            }
```

此时分析代码，很容易得出，迭代次数越大，每个点与点的距离越接近我们设定好的**约束长度(stickLength)**

由此可以借助调整**迭代次数**，来模拟**弹性绳**和**非弹性绳**

#### 借助Unity自带组件lineRenderer渲染

```c#
private void Rendering()
        {
            for (int i = 0; i < particleCount; i++)
            {
                lineRenderer.SetPosition(i, particles[i].position);
            }
            endParticle.transform.position = particles[particles.Count - 1].position;//更新结束点位置
        }
```

至此，大功告成。

## 完整代码

```c#
using System.Collections.Generic;
using UnityEngine;
namespace CordSimulation
{
    [RequireComponent(typeof(LineRenderer))]//添加脚本时自动添加组件‘LineRenderer’
    public class Cord : MonoBehaviour
    {
        public GameObject startParticle;//起始位置
        public GameObject endParticle;//结束位置
        public bool isStartLock;
        public bool isEndLock;
        public int lineNumCornerVertices = 1;
        [SerializeField] private float gravity = 10;
        [SerializeField] private int particleCount = 10;//创建的粒子数目
        [SerializeField] private int lterationStep = 5;//迭代步数，越小越接近弹性绳
        public float ki = 2000;//劲度系数
        private float stickLength;//约束的长度
        private LineRenderer lineRenderer;
        private List<Particle> particles = new List<Particle>();
        private List<Stick> sticks = new List<Stick>();
        private void Awake()
        {
            lineRenderer = this.GetComponent<LineRenderer>();
            lineRenderer.positionCount = particleCount;//设置linerender分段
            lineRenderer.numCornerVertices = lineNumCornerVertices;//设置lineRender转角
        }
        void Start()
        {
            InitSystem();
        }
        //初始化系统
        private void InitSystem()
        {
            stickLength = (startParticle.transform.position - endParticle.transform.position).magnitude / (particleCount - 1);//计算约束长度
            Vector3 dir = (startParticle.transform.position - endParticle.transform.position).normalized;//开始点与结尾点的方向向量
            for (int i = 0; i < particleCount; i++)
            {
                particles.Add(new Particle(startParticle.transform.position - dir * stickLength * i));
            }
            for (int i = 0; i < particles.Count - 1; i++)
            {
                sticks.Add(new Stick(particles[i], particles[i + 1]));
            }
        }
        private void Simulation()
        {
            //实时更新绳子两端是否锁定
            particles[particles.Count - 1].isLock = isEndLock;
            particles[0].isLock = isStartLock;
            var verletDel = new Vector3();
            for (int i = 0; i < particles.Count; i++)
            {
                if (particles[i].isLock == false)
                {
                    Vector3 temp = particles[i].position;
                    verletDel = particles[i].position - particles[i].oldPosition + new Vector3(0, -gravity, 0) * Time.fixedDeltaTime * Time.fixedDeltaTime;//Verlet 算法算法下的位置增量
                    particles[i].position += verletDel;
                    particles[i].oldPosition = temp;//更新oldPosition
                }
            }
            //约束
            for (int i = 0; i < lterationStep; i++)
            {
                // //约束方式1：位置偏移
                // int j = 0;
                // foreach (var k in sticks)
                // {
                //     Vector3 dir = (k.particle_2.position - k.particle_1.position).normalized;//绳子方向的单位向量
                //     float curLength = (k.particle_2.position - k.particle_1.position).magnitude;//当前节点长度
                //     float deltaLength = curLength - stickLength;
                //     if (isStartLock)
                //     {
                //         sticks[0].particle_1 = new Particle(startParticle.transform.position);
                //         particles[0] = sticks[0].particle_1;
                //     }
                //     if (j == 0)
                //     {
                //         if (!k.particle_2.isLock)
                //         {
                //             k.particle_2.position -= deltaLength * dir;
                //         }
                //     }
                //     else if (j == sticks.Count - 1 && isEndLock)
                //     {
                //         if (!k.particle_1.isLock)
                //         {
                //             k.particle_1.position += deltaLength * dir;
                //         }
                //     }
                //     else if (j != 0)
                //     {
                //         if (!k.particle_1.isLock)
                //         {
                //             k.particle_1.position += 0.5f * deltaLength * dir;
                //         }
                //         if (!k.particle_2.isLock)
                //         {
                //             k.particle_2.position -= 0.5f * deltaLength * dir;
                //         }
                //     }
                //     j++;
                //     Debug.Log(j);
                // }

                //约束方式2
                foreach (var k in sticks)
                {
                    if (isStartLock)
                    {
                        sticks[0].particle_1 = new Particle(startParticle.transform.position);
                        particles[0] = sticks[0].particle_1;
                    }
                    Vector3 dir = (k.particle_2.position - k.particle_1.position).normalized;//绳子方向的单位向量
                    float curLength = (k.particle_2.position - k.particle_1.position).magnitude;//当前节点长度
                    float deltaLength = curLength - stickLength;
                    var xDelta = ki * deltaLength * Time.fixedDeltaTime * Time.fixedDeltaTime;
                    if (!k.particle_1.isLock)
                    {
                        k.particle_1.position += xDelta * dir;
                    }
                    if (!k.particle_2.isLock)
                    {
                        k.particle_2.position -= xDelta * dir;
                    }
                }
            }
        }
        private void FixedUpdate()
        {
            Simulation();
        }
        private void LateUpdate()
        {
            Rendering();
        }
        private void Rendering()
        {
            for (int i = 0; i < particleCount; i++)
            {
                lineRenderer.SetPosition(i, particles[i].position);
            }
            endParticle.transform.position = particles[particles.Count - 1].position;//更新结束点位置
        }
    }
    //创建粒子类
    public class Particle
    {
        public Vector3 position;//当前位置
        public Vector3 oldPosition;//上一帧位置
        public bool isLock = false;//是否锁定
        public Particle(Vector3 a)
        {
            position = a;
            oldPosition = a;
        }
    }
    //创建约束
    public class Stick
    {
        public Particle particle_1;
        public Particle particle_2;
        public Stick(Particle a, Particle b)
        {
            particle_1 = a;
            particle_2 = b;
        }
    }
}

```

注意：

```c#
private int lterationStep = 5;//迭代步数，越小越接近弹性绳
public float ki = 2000;//劲度系数
```

这两项不宜设置过大

**lterationStep**过大会导致性能开销增加

**ki**过大会导致 $\Delta t$​内点的位置增量变大，超过一定限度会使点永动，并无限拉伸撕裂。

注意：

代码中包含两种约束方式：

约束1为直接使用**位置**约束，其相关约束条件只有**lterationStep**

约束2为使用**胡克定律**位置约束

在约束2中调整的**ki**的值，能在更小**lterationStep**下，找到符合预期的绳索。

在约束1中因其直接使用位置约束，故不会出现约束2中**ki**值过大的无限拉伸撕裂情况。
