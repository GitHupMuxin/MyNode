# 特殊文件结构
## .ktx 文件
## KTX是**为GPU而生的纹理格式**，它：

1. **存储布局与GPU内存布局匹配**，减少加载时转换
    
2. **包含完整的纹理状态信息**，直接对应Vulkan图像创建参数
    
3. **支持硬件压缩格式**，节省内存带宽
    
4. **预生成Mipmap**，提升渲染质量并减少运行时开销
    
5. **跨平台兼容**，特别是KTX2+Basis Universal组合
# Function: loadAssets
## 加载内容
### empty.ktx
**占位符纹理/默认纹理**，它在渲染管线中扮演着关键的“安全网”和“标准化”角色。
	**1. 防止运行时崩溃（最重要的作用）**
	**2. 提供合理的默认值** : `empty.ktx` 不是纯黑或纯白纹理，而是**精心选择的“中性”值**，适合PBR渲染管线
	**3. 满足Vulkan的同步和状态要求**
### Scene
为场景即渲染的 model
### Environment
处理IBL的原生HDR图
### skybox
天空盒
#### Tips
🎯 **核心区别：光照数据(Environment) vs. 视觉背景(Skybox)**

它们确实**源自同一张HDR环境贴图**，但在渲染管线中被处理成**两种不同的资源**，服务于完全不同的目的：

| 维度        | **Environment（环境贴图）** | **Skybox（天空盒）**   |
| --------- | --------------------- | ----------------- |
| **技术用途**  | **光照计算数据源**           | **视觉背景渲染**        |
| **存储内容**  | 高动态范围（HDR）原始光能        | 低动态范围（LDR）色调映射后图像 |
| **纹理格式**  | 通常为立方体贴图（Cubemap）     | 立方体贴图（Cubemap）    |
| **处理方式**  | 预处理为辐照度图和预滤波图         | 可能应用色调映射、颜色校正     |
| **着色器访问** | 通过IBL计算采样             | 直接采样作为背景          |
| **更新频率**  | 低频（场景光照变化时才变）         | 可能高频（如动态时间、天气）    |

## Function: readDirectory
该函数的目的是读取某个目录下某个后缀的文件路径，保存到结构体 filelist, 最后一个形参是判断是否需要递归读取
## textures.empty.loadFromFile
读取从 **.ktx** 存入 **textures** 结构，需要用到 **gli** 库
通过 **stagingBuffer** 利用 **gli** 中读取数据到了数据，而后通过 **queue** 将 buffer 中的数据存入到 **VkImage** 中，顺便为此 image 创建一个 **VkSampler** 和 **VkImageView**
## Function: loadScene
 1. Function: models.scene.loadFromFile
 2. Function: createMaterialBuffer
 3. Function: createMeshDataBuffer
## models.skybox.loadFromFile
## loadEnvironment
# Function: generateBRDFLUT

# Function: prepareUniformBuffers
# Function: setupDescriptors
# Function: preparePipelines
# Function: updateOverlay