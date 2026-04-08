# 课程PPT链接： 

## Next Event Estimation（下一事件估计）
![Pasted-image-20260408163458.png](02_lajolla/02_lajolla/Pasted-image-20260408163458.png)
这个内容应该是对光源进行重要性采样，在GAMES101 光线追踪的课程中有详细的讲解。
这边对照了两个采样方式即Next event estimation 和 cosine weighted hemisphere
![Pasted-image-20260408163758.png](02_lajolla/02_lajolla/Pasted-image-20260408163758.png)
在小光源的情况下，NEE的效果会比较好
![Pasted-image-20260408163848.png](02_lajolla/02_lajolla/Pasted-image-20260408163848.png)
当接近光源时，则余弦加权采样的效果会比较好
**原因**
![Pasted-image-20260408164037.png](02_lajolla/02_lajolla/Pasted-image-20260408164037.png)
在NEE的公式中，需要除以r²，即除以距离的平方，当着色点距离光源很近时，分母上的 r² 会得到一个接近无限大的值（NaN）。
同时在进行 glancing angle 的采样时，可能会导致自遮挡，而NEE在上述场景中更加容易有glancing angle 的采样情况发生。
## Multiple importance sampling（多重重要性采样）
文中介绍了将NEE和余弦加权两种采样合并的多重重要性采样
![Pasted-image-20260408165006.png](02_lajolla/02_lajolla/Pasted-image-20260408165006.png)多重重要性采样会中和两种采样方式的优点
![Pasted-image-20260408165153.png](02_lajolla/02_lajolla/Pasted-image-20260408165153.png)
## 为何大多数模型的使用三角形作为图元
### 如何处理光线和三角形的相交
在GAMES101 光线追踪课程里面有更加详细的讲述。
### 如何快速判断光线于三角形是否可能相交
这里开始讲述包围盒(Bounding Volumes Hierarchy, BVH)
同样在GAMES101 光线追踪课程里面有更加详细的讲述。

### 如何将分布在模型的三维的颜色映射到一个二维的贴图
这个问题有点困难，而且是我没接触过的，PPT给出了一篇论文（Computational Peeling Art Design SIGGRAPH 2019），简述如何通过2D贴图反过来映射到三维模型上面
论文链接： http://staff.ustc.edu.cn/~fuxm/projects/Peeling/index.html
### 如何采样得到颜色
通过贴图和计算三角形的重心坐标进行采样

### Texture mapping: pros and cons(纹理映射的好处于坏处)
#### Pros：不同采样率（Separate Sampling Rates）
几何体的精细度（顶点密度/三角形数量）与表面颜色的精细度（纹理像素/纹素）彻底解耦。
通过纹理映射在一定程度上可以减少三角形数量，不需要无限精细的几何表示，只需要贴上一张贴图即可在“看起来”是正确的（例子一面凹凸不平且有特定花纹的墙）。
### Pros：更容易过滤（Much Easier to Filter）
在**2D 图像空间**做卷积比在**3D 不规则网格**上做平滑容易得多。
**几何过滤难**：如果想给一个粗糙的雕塑“磨皮”（低通滤波），需要移动顶点位置、改变拓扑，这是复杂的几何处理算法。
**纹理过滤易**：只需利用 **Mipmap** 或 **EWA 滤波**，根据光线微分（`Ray Differential`）或屏幕像素覆盖率，在预计算的图像金字塔里做**三线性插值**或**各向异性采样**。
####  Cons：UV 映射很难（UV Mapping is Hard）
将一个**三维表面**无扭曲地摊平到**二维平面**，且要求接缝处连续、纹理密度均匀，是一个数学上的**全局最优化难题**。就好像你无法摊平3D的地球表面做出来一个世界地图。
**失真（Distortion）**：球面映射必然导致极点处纹理挤压（Pinching）。 
**接缝（Seams）**：为了摊平封闭网格，必须切开网格。切开处会导致纹理滤波在边界处**无法正确跨缝采样**，需要依赖纹理集（Texture Atlas）或复杂的边界填充算法。
**手动劳动**：绝大多数美术资产至今仍依赖**手动展 UV**，这是一个极其枯燥且容易出错的环节，直接拉高了 CG 内容制作的成本。
![Pasted-image-20260408193831.png](02_lajolla/02_lajolla/Pasted-image-20260408193831.png)
### Pixel到纹理的映射可能是一片有面积的纹理，而不是某一点
这个在GAMES101中关于纹理的各项异性和mipmap的讨论中有说
至于文章中的二维函数的一阶泰勒展开，可理解为，通过屏幕空间的相邻像素的纹理坐标的变化率（导数），去估计一个Pixel在纹理上的跨度，根据该跨度进行不同LOD的Mipmap的选择。也可以在不同的Mipmap上进行三线性插值（Trilinear Interpolation）。
![Pasted-image-20260408193803.png](02_lajolla/02_lajolla/Pasted-image-20260408193803.png)

### 对于三角形来说，其构成的模型肯定是有棱有角的，对于这种情况，可以在三角形里面用重心坐标插值来得到一个较为圆滑的过度
![Pasted-image-20260408193740.png](02_lajolla/02_lajolla/Pasted-image-20260408193740.png)

### **用于计算光照的“着色法线”与用于判断光线遮挡的“真实几何表面”指向了不同的方向**。
**真实几何法线（`geometry_normal`）**：由三角形的三个顶点严格定义，垂直于那个唯一的平面。它决定了**光线在哪里反弹、物体是否被遮挡**。
**着色法线（`shading_frame.n`）**：通过**顶点法线插值**或**法线贴图**扰动后得到的法线。它让只有 12 个三角形的模型看起来像有 12 万个三角形的圆润曲面。
![Pasted-image-20260408193903.png](02_lajolla/02_lajolla/Pasted-image-20260408193903.png)
2021年《Ray Tracing Gems II》收录的一篇论文《Hacking the Shadow Terminator》，尝试解决这个问题。论文将通过一个阴影函数生成的概率作为额外的因子，柔和地衰减掉原本不该出现的光照。这样一来，原本突兀、锯齿状的明暗交界线，就会变成平滑、柔和的自然过渡，从而在极低的计算成本下“黑掉”（Hack）了问题，实现了高质量的渲染效果[](https://www.semanticscholar.org/paper/Hacking-the-Shadow-Terminator-Hanika/379081068ff6d8d95aa7c7f5aa2b8e719a80dc22?sort=is-influential)。
## 环境贴图
在GAMES101和 Learn OpenGL 里面都有详细的说明
![Pasted-image-20260408194157.png](02_lajolla/02_lajolla/Pasted-image-20260408194157.png)
# 数学
这章的数学在GAMES101都有详细描述，在上文已经标记了。