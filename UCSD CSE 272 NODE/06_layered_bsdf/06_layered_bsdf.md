# introduction
在现实生活中，材质不只是一层，很可能是多层，这几层结构共同决定了，材质的光学显现。
# 遇到的问题
如果层的间距不是很明显，层内的物理结构没有那么直观，导致其无法像一层玻璃这样，能够直接投射光线。于是找到一个方法对多层结构的光学现象进行拟合，是这个章节将要探讨的问题。
## 多层结构的具体例子
![](image/Pasted%20image%2020260604150123.png)
![](image/Pasted%20image%2020260604150136.png)
Why is wet material darker ? 原因：
![](image/Pasted%20image%2020260604150333.png)

# 第一个想法 Naive combination
和 disney_bsdf 2015 版本类似，只是给不同的层的 lob 增加一个各自的权重，没有考虑到层间散射折射。
![](image/Pasted%20image%2020260604151419.png)
![](image/Pasted%20image%2020260604151620.png)
于此同时，如果我想要给材质上涂一层水质，这种操作完全没有办法简单的实现。
# 解决问题的三种方法
![](image/Pasted%20image%2020260604151729.png)
## Adding Equation (Jakob 2014)
—— 使用矩阵形式精确求解层间多重散射
![](image/Pasted%20image%2020260604152057.png)
对于一簇光在打在一层 layer 的 material 上， 其能量在离开这层 layer 的形式如上，将这些能量加起来显然能够得到级数。
![](image/Pasted%20image%2020260604152327.png)
这个的问题是，T 与 R 通光线的入射角度相关，这里就引入一个离散化的矩阵，完成这件事。于是有
$$
\Phi^\uparrow(t) = \mathbf{R}_t \Phi^\downarrow(t) + \mathbf{T}_{bt} \Phi^\uparrow(b) \tag{1}
$$
$$
\Phi^\downarrow(b)  = \mathbf{R}_b \Phi^\uparrow(b) + \mathbf{T}_{tb} \Phi^\downarrow(t) \tag{2}
$$
其中

| 符号                                | 含义                           |
| --------------------------------- | ---------------------------- |
| $\boldsymbol{\Phi}^\downarrow(t)$ | 从**上方**射向层**顶部**的光通量向量       |
| $\boldsymbol{\Phi}^\uparrow(t)$   | 从层**顶部**向上射出的光通量向量           |
| $\boldsymbol{\Phi}^\downarrow(b)$ | 从层**顶部**穿透后到达**底部**下方的光通量向量  |
| $\boldsymbol{\Phi}^\uparrow(b)$   | 从层**底部**向层内射入的光通量向量（即从下方照射层） |
| $\mathbf{R}_t$                    | **顶部反射矩阵**                   |
| $\mathbf{T}_{tb}$                 | **顶部到底部透射矩阵**（从上到下）          |
| $\mathbf{R}_b$                    | **底部反射矩阵**                   |
| $\mathbf{T}_{bt}$                 | **底部到顶部透射矩阵**（从下到上）          |

进一步可求得两层的情况
![](image/Pasted%20image%2020260604155154.png)
![](image/Pasted%20image%2020260604155123.png)

| 弹跳次数   | 路径                                                  | 矩阵表达式                                                                                       |
| ------ | --------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| 0 (直接) | 光直接被层1顶部反射                                          | $\mathbf{R}_{t1}$                                                                           |
| 1      | 穿透层1 $\to$ 层2顶部反射 $\to$ 回穿层1                        | $\mathbf{T}_{bt1} \mathbf{R}_{t2} \mathbf{T}_{tb1}$                                         |
| 2      | 穿透层1 $\to$ 层2反射 $\to$ 层1底部反射 $\to$ 层2再反射 $\to$ 回穿层1 | $\mathbf{T}_{bt1} \mathbf{R}_{t2} \mathbf{R}_{b1} \mathbf{R}_{t2} \mathbf{T}_{tb1}$         |
| $n$    | 在层1底部和层2之间弹射 $n$ 次                                  | $\mathbf{T}_{bt1} \mathbf{R}_{t2} (\mathbf{R}_{b1} \mathbf{R}_{t2})^{n-1} \mathbf{T}_{tb1}$ |
其中，$T_{tb}$  为穿过整个层的完整操作
将所有路径求和：

$$
\tilde{\mathbf{R}}_t = \mathbf{R}_{t1} + \mathbf{T}_{bt1} \left[ \sum_{n=0}^{\infty} (\mathbf{R}_{t2} \mathbf{R}_{b1})^n \right] \mathbf{R}_{t2} \mathbf{T}_{tb1}​
$$
利用矩阵的**Neumann 级数**求和公式（当谱半径小于1时）：
$$
\sum_{n=0}^{\infty} \mathbf{A}^n = (\mathbf{I} - \mathbf{A})^{-1}
$$
令 $\mathbf{A} = \mathbf{R}_{t2} \mathbf{R}_{b1}$，代入即得：
$$
\tilde{\mathbf{R}}_t = \mathbf{R}_{t1} + \mathbf{T}_{bt1} (\mathbf{I} - \mathbf{R}_{t2} \mathbf{R}_{b1})^{-1} \mathbf{R}_{t2} \mathbf{T}_{tb1}
$$
其他三个组合矩阵可类比推出（注意矩阵乘法顺序不可交换）：
$$
\tilde{\mathbf{R}}_b = \mathbf{R}_{b1} + \mathbf{T}_{tb1} (\mathbf{I} - \mathbf{R}_{b2} \mathbf{R}_{t1})^{-1} \mathbf{R}_{b2} \mathbf{T}_{bt1}
$$
$$
\tilde{\mathbf{T}}_{tb} = \mathbf{T}_{tb2} (\mathbf{I} - \mathbf{R}_{b1} \mathbf{R}_{t2})^{-1} \mathbf{T}_{tb1}
$$
$$
\tilde{\mathbf{T}}_{bt} = \mathbf{T}_{bt2} (\mathbf{I} - \mathbf{R}_{t2} \mathbf{R}_{b1})^{-1} \mathbf{T}_{bt1}​​
$$

![](image/Pasted%20image%2020260604163345.png)
每次一层层叠加，如上述推导，可能导致叠加的速度慢的同时，累积误差。可以通过二叉合并（类似 merge 排序）得到更加稳定的结果。
![](image/Pasted%20image%2020260604164222.png)
同时如果每一层都考虑了介质的影响，则在叠加后自动完成了对介质的累积影响。
![](image/Pasted%20image%2020260604164750.png)
如何生成这种矩阵，则需要多次测量，类似 bsdf 一样。先构造一个薄层的 R 和 T 矩阵
![](image/Pasted%20image%2020260604164809.png)
将方向角分离可以减轻负担
这个方法有他自身的缺点
![](image/Pasted%20image%2020260604170454.png)
首先需要的一个大矩阵，也就是存储有问题。同时贴图会遇到难题。
## Belcour's Algorithm 
—— 用高斯瓣近似，可实时运行（已集成到 Unity HDRP)
取消了 R T 矩阵，改用 GGX lob 进行拟合。
R T 矩阵的本质，是已经算了积分求和的内容，本质是能量。
GGX 拟合的 BSDF 也是能量，但是却是积分前的能量。
GGX 瓣(lob) 为一个球面函数，其核心的参数为：
粗糙度$\alpha$ ：控制 lob 的宽窄
中心方向：lob 指向哪里
强度权重$a_k$ ：lob 的“面积”——对整个半球积分的总反射率或透射率。包含了菲涅尔项和几何项的平均效果。
各向异性参数：$\alpha_x$ 和 $\alpha_y$
![](image/Pasted%20image%2020260604221659.png)
在 ppt 的描述下，一个 GGX lob 被描述为 

|符号|名称|物理含义|
|---|---|---|
|$e$|energy（能量）|该瓣的总反射率或透射率，即对半球积分后的总功率|
|$\bar{\omega}$|mean（平均出射方向）|反射/透射光子的平均出射方向（通常只需仰角 $\bar{\theta}$，各向同性时方位角退化）|
|$\sigma$|variance（方差）|瓣的角扩散程度，即出射方向围绕 $\bar{\omega}$ 的散布宽度|
其中
### 能量 $e$
$$
e = \int_{\mathcal{H}^2} f_r(\omega_i, \omega_o)(\omega_o · \mathbf{n}) \ d\omega_o
$$
这就是该瓣的**方向反照率**（directional albedo）。对于一个 GGX 反射瓣：
$$
e = R(\theta_i; \alpha, F_0) = \int_{0}^{2\pi}\int_{0}^{\pi/2} \frac{D \cdot F \cdot G}{4\cos\theta_i \cos\theta_o} \cos\theta_o \sin\theta_o \, d\theta_o \, d\phi_o​
$$
物理上 $e \le 1$，代表能量守恒。
### 平均方向 $\bar{\omega}$（或 $\bar{\theta}$）

对于镜面反射瓣，$\bar{\omega}$ 就是理想镜面反射方向 $\omega_r$ 。对于粗糙 GGX 瓣，分布的中心仍在 $\omega_r$​ 方向附近，因此：

$\bar{\theta} \approx \theta_r = \theta_i \quad \text{（反射律）}$ 

更精确地说，$\bar{\omega}$ 是该瓣条件分布（在已知 $\omega_i$​ 下）的一阶矩：

$$
\bar{\omega} = \frac{1}{e} \int_{\mathcal{H}^2} \omega_o \, f_r(\omega_i, \omega_o) \, (\omega_o \cdot \mathbf{n}) \, d\omega_o​
$$
### 方差 $\sigma$

$\sigma$ 控制瓣的**宽度**。对于 GGX 分布，$\sigma$ 与粗糙度 $\alpha$ 直接关联：
$$
\sigma^2 \propto \alpha^2
$$
在实际实现中，Belcour 通常直接取：
$$
\sigma = \alpha
$$
或 $\sigma = k \cdot \alpha$，具体的比例常数取决于所用的映射约定。
## 目标：算出来多层 BSDF
对于一个光学路径,使用一个 lob (球面函数)去描述其行为. 
### 对于一层
每个瓣都用一个三元组表示：
$$
\text{Lobe} = (e,\ \bar{\omega},\ \sigma)
$$
- $e$：瓣的总能量（方向半球积分），即给定入射方向下的反射率/透射率
- $\bar{\omega}$：瓣的平均出射方向（一阶矩）
- $\sigma$：瓣在方向空间中的角标准差

对于 GGX 分布，可近似取：
$$
\sigma \approx \alpha \quad \text{（粗糙度）}
$$
### 对于两层 
 入射 $ω_i$
    │
    ▼
 瓣 $A  (e_A, ω̄_A, σ_A)$
    │
    ▼  中间方向 $ω_m$ (随机变量)
 瓣 $B  (e_B, ω̄_B, σ_B)$
    │
    ▼
 出射 $ω_o$
 
| 参数  | 规则                                                                  | 理由            |
| --- | ------------------------------------------------------------------- | ------------- |
| 能量  | $e_{AB} = e_A \cdot e_B$                                            | 两次独立事件概率相乘    |
| 均值  | $\bar{\omega}_{AB} = \frac{1}{2} (\bar{\omega}_A + \bar{\omega}_B$) | 偏转角相加（在斜率空间中） |
| 方差  | $\sigma_{AB}^2 = \sigma_A^2 + \sigma_B^2$                           | 独立随机变量的方差可加性  |

## 3. 路径展开与 Addding 级数的逐项压缩

现在将级数路径逐项写出，并直接应用串联规则。

|项|光学路径|瓣数|
|---|---|---|
|$P_0$|直接在顶层反射，没进入介质|1 个等效瓣|
|$P_1$|进入 → 底层反射 → 出来|1 个等效瓣|
|$P_2$|进入 → 底层反射 → 顶层内反射 → 底层反射 → 出来|1 个等效瓣|
|$P_3$|来回弹 3 次底层反射|1 个等效瓣|
|...|...|...|
### 路径 0（直接反射）

级数第一项：
$$
P_0 = R_t \quad \text{参数: } (e_0,\bar{\omega}_0,\sigma_0) = (e_R,\bar{\omega}_R,\sigma_R)
$$
### 路径 1（穿透 → 底反射 → 穿出）

第二项：
$$
P_1 = T_t \star R_b \star T_t'
$$
根据串联规则（三个瓣）：
$$
\begin{aligned} 
e_1 &= e_T \cdot e_{R,b} \cdot e_T' \\[4pt] 
\bar{\omega}_1 &= \frac{1}{3}(\bar{\omega}_T + \bar{\omega}_{R,b} + \bar{\omega}_T') \\[4pt] 
\sigma_1^2 &= \sigma_T^2 + \sigma_{R,b}^2 + \sigma_T'^{\,2} \end{aligned}
$$
### 路径 2（一次额外内反射）

第三项：
$$
P_2 = T_t \star R_b \star R_t' \star R_b \star T_t'
$$
五个瓣串联：
$$
\begin{aligned} 
e_2 &= e_T \cdot e_{R,b} \cdot e_R' \cdot e_{R,b} \cdot e_T' \\[4pt]
\bar{\omega}_2 &= \bar{\omega}_T + 2\,\bar{\omega}_{R,b} + \bar{\omega}_R' + \bar{\omega}_T' \\[4pt]
\sigma_2^2 &= \sigma_T^2 + 2\,\sigma_{R,b}^2 + \sigma_R'^{\,2} + \sigma_T'^{\,2} 
\end{aligned}​​
$$
### 路径 $k$（$k$ 次底层反射）

一般形式：
$$
P_k = T_t \star (R_b \star R_t')^{\star (k-1)} \star R_b \star T_t'
$$
应用串联规则：
$$
\begin{aligned} 
e_k &= e_T e_T' \, e_{R,b} \left(e_R' e_{R,b}\right)^{k-1} \\[4pt]
\bar{\omega}_k &= \bar{\omega}_T + \bar{\omega}_T' + k\,\bar{\omega}_{R,b} + (k-1)\,\bar{\omega}_R' \\[4pt]
\sigma_k^2 &= \sigma_T^2 + \sigma_T'^{\,2} + k\,\sigma_{R,b}^2 + (k-1)\,\sigma_R'^{\,2} 
\end{aligned}
$$

### base case
只需要预计算一个 energy e 其他的都是 GGX 的内容(均值方向 $\omega$ : 反射/折射方向，和方差 $\sigma$ ：由粗糙度映射)。

**同时，这个实现可以运用在实时渲染当中**
## Monte Carlo Simulation (Guo / Gamboa)
——在材料内部求解简化的一维渲染方程
在材料内部进行蒙特卡洛积分
![](image/Pasted%20image%2020260605162459.png)
#### Detail 1 
> 在 BSDF 的假设下，光线从哪一点进入、从哪一点退出——这两个点的水平距离**不产生可观测差异**。

为什么？因为 BSDF 本身就是 "入射点 = 出射点" 的局部反射模型。层内的水平位移不影响最终的散射方向分布。

示意图再次出现：光子在层间弹跳时的横向偏移，可以被**忽略**。
#### Detail 2
原本的问题是一个复杂的 4D BSDF（2D 入射 × 2D 出射）。水平位移可忽略后，内部状态仅仅由**高度（深度）+ 方向**描述，在这个简化空间里做路径追踪，维度降低、采样效率大大提高。
 
### 相比 Belcour 方法的优势（第 52 页）

幻灯片指出 Belcour 方法的两个局限：

1. **不支持多层高分辨率法线贴图**——Belcour 的算法假定每层界面是平坦的，无法正确建模法线贴图带来的局部方向变化
2. **分层 BSDF 的 mipmapping** 仍未解决——这被标注为一个有趣的研究方向


# 有趣的近期研究：

## Belcour 算法的"逆问题"

幻灯片提到：

> **the inverse version of Belcour's algorithm**

这是指 **逆渲染（Inverse Rendering** 方向的工作。Belcour 算法解决的是正问题：给定每层材质参数（粗糙度、折射率等），求出组合后的 BSDF。而"逆版本"则是反过来——**从一张照片或测量数据中，反向推断出各层的物理参数**。

这在应用层面意义重大：

- 相机拍一张实物照片 → 自动分解为涂层厚度、基底粗糙度、各层折射率等
- 材质扫描、数字资产生成、外观捕获都可受益

---

## 其他研究

- **基于神经网络的层次材质表示**：用小型神经网络替代矩阵或高斯近似，学习紧凑的分层 BSDF 编码
- **分层材质的空间变化（SVBRDF）**：将分层 BSDF 扩展到随空间位置变化，支持更丰富的纹理表现
- **时变 / 可风化分层材质**：模拟层的厚度随时间演变（如油漆剥落、氧化）

---

## 散射介质中的分层 BSDF

幻灯片明确指出：

> **a layered-BSDF that focuses on scattering medium**

前面三种方法主要处理**界面间的弹跳**（dielectric 层 + diffuse 层），层与层之间是真空或透明介质。但如果层间充满**参与介质**（如浑浊液体、雾气、半透明涂层），就需要同时处理：

- 界面处的 Fresnel 反射/折射
- 介质内部的体积散射（吸收 + 散射系数）

这项研究将 adding-doubling 或蒙特卡洛方法推广到**体积散射 + 界面反射耦合**的场景。实际应用包括：

- 带颜料的清漆层
- 浑浊水面上的油膜
- 半透明皮肤层（表皮 + 真皮散射）