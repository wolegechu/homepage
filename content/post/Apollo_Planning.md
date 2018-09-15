+++
date = "2018-09-14T23:00:00"
draft = false
tags = ["Self-driving", "star"]
title = "Apollo Planning 模块和其路径规划方法简析"
math = true
outurl = ""
summary = """
Apollo 的 `Planning` 模块以 `Routing` 模块产生一条当前位置到终点的理想路径作为输入，结合当前场景下的路况信息（信号灯、短期内障碍物的运动轨迹等），生成短期内的一条驾驶路径。
"""

[header]
image = ""
caption = ""

+++

Apollo 的 `Planning` 模块以 `Routing` 模块产生一条当前位置到终点的理想路径作为输入，结合当前场景下的路况信息（信号灯、短期内障碍物的运动轨迹等），生成短期内的一条驾驶路径。

在展开之前先介绍下 Apollo 主要采用的 Frenet 坐标系，见下图，整个坐标系基于一条光滑的参考线（右图中红线），然后将汽车的坐投影到参考线上，得到一个参考线上的投影点（图中蓝色点）。从参考线起点到投影点的路径长度就是汽车在 Frenet 坐标系下的纵向偏移量，用 `S` 表示。而投影点到汽车位置的距离则是汽车在Frenet坐标系下的横向偏移量，用 `L` 表示。通过汽车的朝向、速度、加速度也可以计算出 Frenet 坐标系下，横向和纵向偏移量的一阶导和二阶导。

![](/img/post/apollo_frenet.png)

Planning 模块的主要流程如下（最下面有大致流程图）：

- 从 Routing 模块获取全局路径
- 使用全局路径通过 `Planning and Control (PnC) Map` 生成多条未经平滑的参考线，范围为车辆后方 30m 至前方 150～250m
- 进行参考线平滑（也包括了参考线复用、拼接、收缩等操作）
- `Frame` 类初始化使用 `PnC Map` 生成的参考线
- 融合从感知模块获取的障碍物信息 `PathObstacle` 和障碍物（以及交规）对路径的决策影响 `PathDecision` 构成 `ReferenceLineInfo` 类
- 基于参考线信息和其中障碍物未来时刻的轨迹信息，规划器（`EM planner` 或 `Lattice planner`）会生成一条未来短距离内（如 40m）的最优规划路径 `PathData` (S-L 序列)，以及短时间内（如 8s, 150m）的速度规划序列 `SpeedData` (S-T 序列)
- 融合两者得到**短时间内最优路径** （S-L-T序列）交由 `Control` 模块处理。

`PnC Map` 主要做的事是生成参考线，这里参考线可以直接看作是车道中心线，通常会对当前车道和当前车道可驶入的邻接车道（该邻接车道也需存在于全局规划的路径中）分别生成参考线供后续模块使用。

在 `Frame` 类中，因为 `PnC Map` 给出的参考线实质上是一系列的点，所以会首先对每条参考线进行光滑处理，这里也涉及到了跨车道（如路口、分流、汇流等）情况下的参考线拼接。然后从感知模块获取障碍物信息进行滞后预测（lagged prediction），即除了使用 `Prediction` 模块发布的最新一次信息，还会考虑障碍物的历史轨迹预测数据以供更精确地预测。再将障碍物的轨迹信息加入到参考线中，并判断每个障碍物对自身轨迹规划产生的决策影响（纵向：忽略，停车，超车，减速，跟车；横向：忽略，微调，绕行）。

![](/img/post/apollo_planning_flowchart.png)

这里只是简单介绍了 Apollo 中的路径规划过程，更多细节可以参见 Github 中已开源的代码，为了更好地了解也可以进一步阅读下面的参考资料。

**Reference**

1. https://github.com/wolegechu/Apollo-Note (Forked from [YannZyl/Apollo-Note](https://github.com/YannZyl/Apollo-Note))
2. [ApolloAuto/apollo](https://github.com/ApolloAuto/apollo)
3. [Apollo 3.0 技术指南 - Planning 模块](https://github.com/ApolloAuto/apollo/blob/master/modules/planning/README.md)
4. [Apollo3.0 PnC更新以及车辆开放平台介绍](https://mp.weixin.qq.com/s?__biz=MzI1NjkxOTMyNQ==&mid=2247485310&idx=1&sn=830f95c8fe1c0fe300ab4baa735361d5&chksm=ea1e150cdd699c1a149879f5d044b6ed866d0ae5d30c59aa571cdbcad2cad738c964c397fcdd&mpshare=1&scene=1&srcid=0914fHtF0ADvSZWvZ7WSioeu&rd2werd=1#wechat_redirect)
5. [Lattice Planner规划算法](https://mp.weixin.qq.com/s?__biz=MzI1NjkxOTMyNQ==&mid=2247485354&idx=1&sn=c8f65591fe727904a8e1e440fb6827b9&chksm=ea1e15d8dd699cce46d296ede37177ccc8cc2f6c275ebd5c5b2f8008248f7d64de201e6ee1bf&mpshare=1&scene=1&srcid=0914R7kJdKmCmX6XxA0NTUXp&rd2werd=1#wechat_redirect)