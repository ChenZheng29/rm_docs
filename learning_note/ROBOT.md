# ROBOT

## RSF

(Robotee Solution Framework)

### 确定要解决的问题和预期效果

一个清晰并且切合实际的目标

> Without a **clear definition of this need and the objectives**, the engineering design process cannot begin. Much time and many careers have been wasted in the pursuit of an un-defined target.

#### 有条理的做法

1. 汇总问题
2. 排列目标重要性顺序
3. 列出问题的约束条件



> -Get a clear picture of the **Parameters** of the problem.  ---   
>
>  -Make a list of the **Objectives** and rank them in order of **importance**.
>
> -Define the **Constraints** of the problem.
>
> -Many times a robot cannot do everything that a problem presents. It is important to **prioritize** and design a machine that can do the most things and do a few things very well.

### 调查和头脑风暴

甚至以前被别人尝试但是失败的方法才是金矿，以前的缺点可以被现在的情况填补

> Old ideas that failed are sometimes great research gold mines; that idea may have failed due to a lack of new technology that may exist now.

#### 做法

研究已有相同，相似的机器人

确定必须要解决的的设计的具体细节

确定可能的备选方案

规划项目时间线

> ### \- Explore other solutions to the same and similar problems,
>
>  – Identify specific details of the design which must be satisfied,
>  – Identify possible and alternative design solutions
>  – Plan and design an appropriate structure which includes drawings

先在纸上画，保持草图完整可用，方便后期以此为参考建模

> The first step is to **start sketching** to get the ideas on paper. Sketching and drawing by hand enables you to tap your creative side. It is important to have accurate and complete  sketches in order to translate the idea into hand or CAD drawings and  models. This phase also allows for **virtual prototyping**  or testing of the product in the computer. You can find potential, and  sometimes costly, flaws in a design before the real world mock-up is  constructed.
>
> **Draw and talk about ideas with in groups**. ==No ideas are bad ideas==. It is important to consider all approaches to a  problem. One that did not seem feasible or make sense in the beginning  might be the way to go in the end. Not too many projects go through  development on the first try or on the best idea at the time. **The final project usually consists of a collection of ideas**; some that were considered too risky, costly, or just plain crazy.
>
> **Solutions must be separated according to their pros and cons.** This activity is better accomplished in a group setting. **Brainstorming** encourages a maximum amount of input from different levels of experience and **different approaches** to the problem. Alternative solutions can be analyzed and cataloged  according to merit and possible use. After these ideas have been  distilled to a manageable number, the numbers must be crunched to  evaluate the probability and cost of a successful outcome, using the  individual solutions. Larger factors come into play here, such as common sense and instinct. If it doesn’t feel right, don’t do it.

### 仿真验证

在仿真里面做模型，来测试方法是否可行，如果不满足预期的效果，可能需要重新来一次这个流程

> The best way to know if a design will work in real-world conditions is to **build a prototype**.
>
> **Sketches and notes are required at this stage**. (May be you can  create prototypes using lego for this step. Once you  have created a lego prototype, take a digital picture of it.) Print out  the picture and jot your notes below the picture in your log book. Once  you have settled on a solution, go back over the list of specifications  you have made. Make sure that each specification is satisfied.
>
> If an initial design and prototype does  not fully solve the problem or specifications, meet the design  parameters, or stay within an acceptable cost, a designer may go “**back to the drawing board**” (or computer). The engineering design process has a loop to go back to the design and refine or redesign.
>
> Now it is the time to produce some  working drawings. These are the drawings that will assist you as you  begin constructing your robot. (Here again, lego and a digital camera  might be your best friend.) You may choose to do your drawings by hand  or you might want to use a draw program on the computer to assist you.

### 实物搭建

1. 需要考虑成本规划
2. 列出物资使用清单

### 重复测试

总结设计中的优点和缺点的陈述。它应该描述您在哪些方面取得了成功，哪些方面未能实现规范中规定的目标。

问题包括但不限于

> Here is a list of questions which will help you to prepare this statement.
>  • How well does the design function?
>  • Does the design look good?
>  • Is the product safe to use?
>  • Did I plan my work adequately?
>  • Did I find the construction straightforward or difficult?
>  • Were the most suitable materials used?
>  • Did it cost more or less than expected?
>  • How could I have improved my design?

### 回到第一步

# BIPEDAL Robots

## Swing Boy

### 解决的问题

- 在各个体育项目中，动作往往有规范和个人风格，在个人学习初期，往往是看视频然后把自己的动作视频和其他人的对比，看自己有那里不一样可以改进，然而视频的角度限制和衣物遮挡，让对比并不清晰明了；或者是请教身边做得好的人，但是会做的人并不一定可以表达清楚。在这个背景下，本机器人研发目的在于提高个人或者团体在学习体育规范动动作时学习效率和质量。

### 预期效果

#### 视觉

采用Jetson neno和摄像头对正面，侧面，上面三个方向的动作视频进行关节检测，分别得到双足机器人各个rool,pitch.yaw的三个方向的旋转角度，把初始状态设为直立。然后把各个关节转换坐标系到base_link这样就可以确定每个关节的位置了。

#### 双足本体

###### 硬件方面：

动力系统：

方案一：采用RM3508无刷电机+C620电调

方案二：购买云台电机自制FOC驱动板改装成伺服电机

方案三：购买关节电机

通讯系统：

主控和电机：nuc搭配usb to can用can总线读取电机数据

==主控和下位机==

能源系统：用DJI TB48S 27V电池

###### 结构方面：

传动：

方案一：踝关节电机移到膝关节

方案二：把膝关节和踝关节的pitch移到髋关节

### 调查

#### 参考：

LEO：可以飞的双足机器人

参考点：轻量化的腿部结构设计

> https://www.science.org/doi/epdf/10.1126/scirobotics.abf8136

