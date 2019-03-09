---
title: 基于Perlin Noise随机生成游戏地图
date: 2019-03-05 16:02:19
tags:
---



**摘要**

程序化内容生成（PCG）在游戏中的应用是使用随机数或伪随机数来实现游戏内容的程序化生成，其导致游戏内容更加不可预测，为游戏增加更多的可能性，为玩家带来更加持续的新鲜感。同时，这种方法也为游戏开发人员带来了许多优势，例如减少了内存消耗，辅助开发者更加迅速地进行游戏设计，以及对于基于人工的游戏开发作为一种补充。电子游戏中，地形地貌是游戏内容的重要组成部分，也会对游戏的整体美术风格与玩家的游戏体验造成很大影响。本文旨在提供一种顶视角2D像素风格游戏中实时生成无限地形地貌的可实行方案，希望能够提供一套基于Perlin noise的算法，通过输入有限个伪随机数种子，最终计算出较为美观的顶视角2D像素风格户外地貌，并且提供较强的多样性，使玩家在探索游戏地图时收获新鲜感与乐趣。

 

**关键词 :** 程序化内容生成, perlin噪声, 像素游戏 

 

**1.** **引言**

电子游戏中，地形地貌是游戏内容的重要组成部分，也会对游戏的整体美术风格与玩家的游戏体验造成很大影响。在以往的游戏中，地形地貌多为游戏设计与游戏开发人员事先设计好的内容，游戏一旦发布就已经固定，不具备任何随机性，而且不具备随着玩家的游戏进程实时生成，实时延展且无限延展的特性。近十年来，一大批内含程序化地形地貌生成的游戏涌现出来，如《Minecraft》，《Terraria》，《Don't Starve》，《Enter the Gungeon》，这些游戏中或多或少的用到了程序化内容生成的思想，使其游戏内容尤其是游戏地形地貌更加的丰富和随机，带给玩家更持久的新鲜感，继而提供了更大的重复可玩性。

程序化内容生成（PCG）在游戏中的应用是使用随机数或伪随机数来实现游戏内容的程序化生成，其导致游戏内容更加不可预测，为游戏增加更多的可能性，为玩家带来更加持续的新鲜感。同时，这种方法也为游戏开发人员带来了许多优势，例如减少了内存消耗，辅助开发者更加迅速地进行游戏设计，以及对于基于人工的游戏开发作为一种补充[1]。PCG可以应用在影响游戏玩法的各种元素之上：地形，地图，图层，故事，对话，任务，角色，规则，变化或武器。PCG一种重要的应用就是生成无限的游戏内容，它可以用于实时生成具有足够变化性的内容。在1980年代，《Rogue》游戏为PCG开辟了一条新的道路，它创造了对后世影响深远的贡献，即游戏中的冒险是通过算法来生成的，每次游玩，就会创造一个全新的冒险[2]。后来很多游戏开发者模仿《Rogue》使用PCG来生成其游戏内容，以至于后来形成了一个新的roguelike游戏类型。Julian Togeliu等人提出了PCG在游戏创作中的三大目标和九大挑战，三大目标即多级多内容PCG，基于PCG的游戏设计和生成完整游戏，而九大挑战之中则包含非通用原创性内容生成这一挑战，其中提到了如今的roguelike游戏之中的地牢生成缺乏创意性[3]。Mark Hendrikx等人提出了程序化内容生成在游戏中应用的几个层次，自下而上分别为游戏比特层、游戏空间层、游戏系统层、游戏情节层、游戏设计层以及衍生内容层，其中，游戏地形地貌的程序化生成包含在游戏空间层，是基础而重要的一层[4]。

然而，在顶视角2D像素风格游戏中，游戏地形地貌如何合理并且丰富地展现给玩家是一个重要的问题。目前的顶视角2D游戏中使用的程序化地图生成技术，多数为地牢游戏中地牢房间的随机生成，而户外地形地貌生成的现有实现方案多数缺乏美观性。如何在顶视角2D游戏中程序化生成有一定风格的户外地形地貌，是一个有待解决的问题。

本文旨在提供一种顶视角2D像素风格游戏中实时生成无限地形地貌的可实行方案，希望能够提供一套算法，通过输入有限个伪随机数种子，最终计算出较为美观的顶视角2D像素风格户外地貌，并且提供较强的多样性，使玩家在探索游戏地图时收获新鲜感与乐趣。

为了实现这些目标，本文选用Perlin噪声为主要的伪随机数生成算法[5]，实现一个二维Perlin噪声网格，并且在此基础上建立由数值到地形类型的映射关系。另外，本文将根据每个点计算出的Perlin噪声值来指定生成地图资源如树木石头的概率。最后，将在水域地形上创造碰撞体，在特定地形上设定摩擦系数，改变角色的移动速率。

 

**2.** **相关工作**

研究人员早已认识到程序化内容生成（PCG）在电子游戏创作中的重要性，之前有很多人从不同方面做出过相关的工作。在游戏开发中使用程序内容生成技术主要限于特定类型的游戏元素。PCG很少被用于生成整个游戏关卡，但是有一个值得注意的例外。地下城类型的游戏是一种在冒险和角色扮演游戏中经常遇到的游戏类型，其游戏关卡经常由PCG生成。由于其独特的游戏节奏，游戏玩法和游戏空间的组合，地牢关卡设计成为最适合展示PCG优势的地方。Roland van der Linden等人开展了关于生成地下城游戏关卡的程序方法的研究，他们总结了常见的做法，讨论了不同方法的优缺点，并确定了未来的一些有希望的挑战[6]。一般来说，当前的程序性地下城生成方法缺少的不是性能，而是更加强大的生成过程，以及对于生成过程更准确和更丰富的控制。这个领域仍然处于起步阶段，许多研究挑战仍然存在，例如，为设计师提高这些方法的直观性和可访问性。

关于将程序化内容生成应用于其他类型游戏地图生成领域，之前有人做出过一些实现方案。Lara-Cabrera等人为即时战略游戏《Planet Wars》提供了一个平衡的程序化地图生成器。该生成器使用进化策略生成和演化地图，并使用锦标赛系统评估这些地图的平衡质量，他们进行了几次实验，获得了一组可玩并且平衡的游戏地图[7]。Lawrence Johnson等人利用元胞自动机算法实现了洞穴关卡的无限生成，他们将基于元胞自动机的简单算法在无限洞穴游戏中进行评估，生成可玩且设计良好的基于隧道的地图。该算法具有非常低的计算成本，允许实时内容生成，并且所提出的地图在关卡设计方面提供了足够的灵活性[8]。Miguel Frade等人使用基于遗传算法的遗传地形程序技术自动演化地形，以获得所需的有足够可访问性的游戏地图[9]。Jean-David G´enevaux等人提出了一个框架，允许使用受水文学启发的概念快速直观地建模地形[10]。Houssam Hnaidi等人介绍了一种从一组参数化曲线生成地形的扩散方法，该曲线描述了地形特征，如山脊线，河床或悬崖。他们的方法为用户提供了基于矢量的直观的面向特征的地形控制。 不同类型的约束（例如高程，倾斜角和粗糙度）可以附加到曲线上，以便定义地形的形状，通过使用有效的多重网格扩散算法从曲线表示生成地形。 该算法可以在GPU上有效地实现，这允许用户以交互方式创建各种各样的景观[11]。

除了上述的地形地貌生成算法，还有许多研究者对于地图相关物的生成进行了一系列研究。Stefan Greuter提出了一种实时生成“伪无限”虚拟城市的方法。城市包含根据需要生成的几何变化的建筑物。建筑物生成参数由从建筑物位置导出的整数播种的伪随机数生成器创建。不同的建筑几何形状是从一组平面图中挑选出来。每个建筑物的楼层是通过在迭代过程中组合随机生成的多边形来创建的[12]。Soon Tee Teoh设计了一种算法模拟河流和海洋的侵蚀作用，来创建自然的河流、河流三角洲、沿海悬崖和海滩[13]。

与之前的程序化地形地貌生成相比，本文的地形地貌生成方法的难点之一在于如何在顶视角2D像素风格地图上表现出立体感，给与玩家以较为拟真的体验。另一个难点在于如何丰富地形地貌的多样性，以使玩家保持长久的新鲜感与游戏趣味。

 

**3.** **实现方法**

3.1.Perlin noise

Perlin噪声是一个非常强大的算法，经常用于程序生成随机内容，在游戏和其他像电影等多媒体领域广泛应用。算法发明者Ken Perlin也因此算法获得奥斯卡科技成果奖。在游戏开发领域，柏林噪声可以用于生成波形，起伏不平的材质或者纹理。例如，它能用于程序生成地形、火焰燃烧特效、水和云等等。柏林噪声绝大部分应用在二维，三维层面上，但某种意义上也能拓展到四维。一维柏林噪声可用于生成横版卷轴地形、模拟手绘线条等，在二维层面可以通过插值生成地形，而三维柏林噪声则可以模拟海平面上起伏的波浪。

Perlin噪声算法可以用于生成更加连续而平滑的随机数，没有白噪声的毛刺感，所以是生成游戏中地形地貌的一个重要工具。

3.2.计算噪声图

要使用Perlin噪声生成游戏地形地貌，需要计算出一个二维Perlin噪声，具体的实现方式需要以下几步。第一步，本文先用一个伪随机数生成器生成一个伪随机数序列，这个伪随机数生成器可以使用经典的线性同余法。第二步，需要在游戏场景中创建一个二维网格，本文暂且把每个单元格的大小设置为100*100，这个大小可以按需调整。之后，本文给二维网格的每个点指定一个随机的二维梯度向量，这些随机向量两个维度的值都是从第一步中伪随机数序列中抽取得到，之后对这个向量进行单位化，这样就得到了一张每个网格点都对应一个指向某个随机方向的单位向量的二维网格。第三步，也是最重要的一步，开始计算场景中每个点的噪声值，即一个0到1之间的随机数。

关于这个计算方法，本文将任取一个点举例说明。想要计算一个点的噪声值，首先，要先找到他所在的单元格，并且确定该单元格四个角的坐标点，这一操作可以通过整型变量除法得到。例如本文取坐标为（157，-233）的这个点，就可以得到其所在单元格的四个角的坐标分别为：（100，-300）、（100，-200）、（200，-300）、（200，-200）。然后通过两点位置矢量相减的方法分别得到这四个角指向待计算点的四个向量，将这四个向量分别与其所在的角的网格点对应的随机单位向量点乘，得到四个值，将这四个值根据待计算点在单元格内部的相对于左下角的uv值进行双线性插值。所谓uv值，即为相对于左下角的局部坐标系的坐标与单元格大小的比值，例如点（157，-233）局部坐标uv值即为（0.57,0.67），根据u值进行两次线性插值，再对结果根据v值进行一次线性插值，即为双线性插值。这样所得到的最终结果已与所需要的结果非常接近，但是可以发现，这样的计算结果的取值范围为（                                                  ）。于是最后将结果乘以   之后加1再除以2，即得到本文所需要的取值范围（0,1）的噪声值。

3.3.配置地形数值映射表

虽然计算平滑且一阶导数连续的噪声值是本文地形地貌生成算法的重要一步，但是仅仅得到游戏场景中每个点的噪声值还是不够的，还需要建立噪声值与地形地貌之间的映射关系，才可以指导在屏幕上渲染什么样式的地形地貌贴图。本文将（0,1）这个范围根据需要分为若干个小区间，每一个小区间映射到一种类型的地形地貌。例如（0.2,0.35）映射为草地，那么当某一点的噪声值落在（0.2,0.35）区间内的时候，即可以根据其草地的特性为其贴上相应的地砖或是绘制相应的颜色。除此以外，还可以给每一种地形设置摩擦系数，以限制玩家角色在其上的奔跑速度，或者设置游戏中资源如在此种地形之上的生成概率，以控制资源在不同地形上的分布。

 

**4.** **结果与展望**

本文在Unity引擎中利用引擎中的Tilemap系统实现了该算法，计算出一张足够大的二维Perlin噪声网格以键值对的形式保存在C#的字典结构中，然后根据这些噪声值在Tilemap系统中给各点贴上了地砖Tiles，并且根据设定的概率值添加了树木和石头等游戏中需要用到的资源，最后形成了比较好的效果。

在这个基于Perlin Noise生成游戏地图的算法基础之上，还有很多可以改进的点，比如更好的美术效果，在地图上加入剧情引导等。如何用PCG技术随机生成更具风格化与可玩性的游戏地图，还有待之后的研究不断进行探索。

 

参考文献

[1]  Shaker N, Togelius J, Nelson M J. Procedural Content Generation in Games[M]. Cham: Springer International Publishing, 2016.

[2]  Amato A. Procedural Content Generation in the Game Industry[A]. 见: O. Korn, N. Lee. Game Dynamics[M]. Cham: Springer International Publishing, 2017: 15–25.

[3]  Togelius J, Champandard A J, Lanzi P L, et al. Procedural content generation: Goals, challenges and actionable steps[C]//Dagstuhl Follow-Ups. Schloss Dagstuhl-Leibniz-Zentrum fuer Informatik, 2013, 6.

[4]  Hendrikx M, Meijer S, Van Der Velden J等. Procedural content generation for games: A survey[J]. ACM Transactions on Multimedia Computing, Communications, and Applications, 2013, 9(1): 1–22.

[5]  Liu D, Yan S, Ji R-R等. Image retrieval with query-adaptive hashing[J]. ACM Transactions on Multimedia Computing, Communications, and Applications, 2013, 9(1): 1–16.

[6]  van der Linden R, Lopes R, Bidarra R. Procedural Generation of Dungeons[J]. IEEE Transactions on Computational Intelligence and AI in Games, 2014, 6(1): 78–89.

[7]  Lara-Cabrera R, Cotta C, Fernández-Leiva A J. Procedural map generation for a RTS game[C]//13th International GAME-ON Conference on Intelligent Games and Simulation. 2012: 53-58.

[8]  Johnson L, Yannakakis G N, Togelius J. Cellular automata for real-time generation of infinite cave levels[A]. Proceedings of the 2010 Workshop on Procedural Content Generation in Games - PCGames ’10[C]. Monterey, California: ACM Press, 2010: 1–4.

[9]  Frade M, de Vega F F, Cotta C. Evolution of Artificial Terrains for Video Games Based on Accessibility[A]. 见: C. Di Chio, S. Cagnoni, C. Cotta等. Applications of Evolutionary Computation[M]. Berlin, Heidelberg: Springer Berlin Heidelberg, 2010, 6024: 90–99.

[10] Génevaux J-D, Galin É, Guérin E等. Terrain generation using procedural models based on hydrology[J]. ACM Transactions on Graphics, 2013, 32(4): 143.

[11] Hnaidi H, Guérin E, Akkouche S等. Feature based terrain generation using diffusion equation[J]. Computer Graphics Forum, 2010, 29(7): 2179–2186.

[12] Greuter S, Parker J, Stewart N, et al. Real-time procedural generation ofpseudo infinite'cities[C]//Proceedings of the 1st international conference on Computer graphics and interactive techniques in Australasia and South East Asia. ACM, 2003: 87-ff.

[13] Teoh S T. River and Coastal Action in Automatic Terrain Generation[C]//CGVR. 2008: 3-9.

 

 