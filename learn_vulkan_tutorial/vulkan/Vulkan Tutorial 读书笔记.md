本文的主要内容是个人在学习Vulkan Tutorial时对Vulkan的不同结构体的概念以及见解，，可以用来当作另一个视角下的Vulkan Tutorial。（如有错误，还请指正）代码沿用Vulkan Tutorial 例程的代码。

首先配好vulkan的编程环境，这个内容在不同的.操作系统以及不同的编译器甚至IDE都有差别，并非本文主要内容，还请读者自行配置。
我们会从Instance开始。
# Instance
本章节目标：
**1.**  创建一个`VkInstance`([[#Creating an instance]])
**2.** 解决有关**VK_ERROR_INCOMPATIBLE_DRIVER**的报错([[#Encountered VK_ERROR_INCOMPATIBLE_DRIVER]])
**3.** 检查是否支持特定拓展([[#Checking for extension support]])
**4.** 推出程序前释放`VkInstance`([[#Cleaning up]])
## Creating an instance
首先我们要构造VkInstance 实例.         
There is no global state in Vulkan and all per-application state is stored in a `VkInstance` object. Creating a `VkInstance` object initializes the Vulkan library and allows the application to pass information about itself to the implementation.
也就是说,一个程序可以对应多个VkInstance, 一个VkInstance可以运行不同的Vulkan程序.

首先，声明一个函数`void createInstance()`， 以及一个结构体`VkInstance`。
``` c++
class HelloTriangleApplication
{
	private:
		......
		VkInstance instance;
		void createInstance();
		......
}

void HelloTriangleApplication::createInstance()
{
    VkApplicationInfo appInfo{};
    appInfo.sType = VK_STRUCTURE_TYPE_APPLICATION_INFO;
    appInfo.pApplicationName = "Hello Triangle";
    appInfo.applicationVersion = VK_MAKE_VERSION(1, 0, 0);
    appInfo.pEngineName = "No Engine";
    appInfo.engineVersion = VK_MAKE_VERSION(1, 0, 0);
    appInfo.apiVersion = VK_API_VERSION_1_0;

    VkInstanceCreateInfo createInfo{};
    createInfo.sType = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO;
    createInfo.pApplicationInfo = &appInfo;

    uint32_t glfwExtensionCount = 0;
    const char** glfwExtensions;
    glfwExtensions = glfwGetRequiredInstanceExtensions(&glfwExtensionCount);

	//此处启用的拓展
    createInfo.enabledExtensionCount = glfwExtensionCount;
    createInfo.ppEnabledExtensionNames = glfwExtensions;

    createInfo.enabledLayerCount = 0;

    if (vkCreateInstance(&createInfo, nullptr, &instance) != VK_SUCCESS) {
        throw std::runtime_error("failed to create instance!");
    }
}

void HelloTriangleApplication::cleanup()
{
    vkDestroyInstance(instance, nullptr);
    glfwDestroyWindow(window);
    glfwTerminate();
}

```

### VkInstanceCreateInfo
```c++
// Provided by VK_VERSION_1_0
typedef struct VkInstanceCreateInfo {
    VkStructureType             sType;
    const void*                 pNext;
    VkInstanceCreateFlags       flags;
    const VkApplicationInfo*    pApplicationInfo;
    uint32_t                    enabledLayerCount;
    const char* const*          ppEnabledLayerNames;
    uint32_t                    enabledExtensionCount;
    const char* const*          ppEnabledExtensionNames;
} VkInstanceCreateInfo;
```
#### Members
- `sType` is a [VkStructureType](https://registry.khronos.org/vulkan/specs/latest/man/html/VkStructureType.html) value identifying this structure.
    
- `pNext` is `NULL` or a pointer to a structure extending this structure.
    
- `flags` is a bitmask of [VkInstanceCreateFlagBits](https://registry.khronos.org/vulkan/specs/latest/man/html/VkInstanceCreateFlagBits.html) indicating the behavior of the instance.
    
- `pApplicationInfo` is `NULL` or a pointer to a `VkApplicationInfo` structure. If not `NULL`, this information helps implementations recognize behavior inherent to classes of applications. [VkApplicationInfo](https://registry.khronos.org/vulkan/specs/latest/man/html/VkApplicationInfo.html) is defined in detail below.
    
- `enabledLayerCount` is the number of global layers to enable.
    
- `ppEnabledLayerNames` is a pointer to an array of `enabledLayerCount` null-terminated UTF-8 strings containing the names of layers to enable for the created instance. The layers are loaded in the order they are listed in this array, with the first array element being the closest to the application, and the last array element being the closest to the driver. See the [https://registry.khronos.org/vulkan/specs/latest/html/vkspec.html#extendingvulkan-layers](https://registry.khronos.org/vulkan/specs/latest/html/vkspec.html#extendingvulkan-layers) section for further details.
    
- `enabledExtensionCount` is the number of global extensions to enable.
    
- `ppEnabledExtensionNames` is a pointer to an array of `enabledExtensionCount` null-terminated UTF-8 strings containing the names of extensions to enable.

#### Example
```c++
VkInstanceCreateInfo createInfo{};
createInfo.sType = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO;
createInfo.pApplicationInfo = &appInfo;
```

注意到其需要一个pApplicationInfo,这便是我们要填充的另外一个结构体

### VkApplicationInfo
```c++
// Provided by VK_VERSION_1_0
typedef struct VkApplicationInfo {
    VkStructureType    sType;
    const void*        pNext;
    const char*        pApplicationName;
    uint32_t           applicationVersion;
    const char*        pEngineName;
    uint32_t           engineVersion;
    uint32_t           apiVersion;
} VkApplicationInfo;
```
#### Members
- `sType` is a [VkStructureType](https://registry.khronos.org/vulkan/specs/latest/man/html/VkStructureType.html) value identifying this structure.
    
- `pNext` is `NULL` or a pointer to a structure extending this structure.
    
- `pApplicationName` is `NULL` or is a pointer to a null-terminated UTF-8 string containing the name of the application.
    
- `applicationVersion` is an unsigned integer variable containing the developer-supplied version number of the application.
    
- `pEngineName` is `NULL` or is a pointer to a null-terminated UTF-8 string containing the name of the engine (if any) used to create the application.
    
- `engineVersion` is an unsigned integer variable containing the developer-supplied version number of the engine used to create the application.
    
- `apiVersion` **must** be the highest version of Vulkan that the application is designed to use, encoded as described in [https://registry.khronos.org/vulkan/specs/latest/html/vkspec.html#extendingvulkan-coreversions-versionnumbers](https://registry.khronos.org/vulkan/specs/latest/html/vkspec.html#extendingvulkan-coreversions-versionnumbers). The patch version number specified in `apiVersion` is ignored when creating an instance object. The variant version of the instance **must** match that requested in `apiVersion`.
#### Example
```c++
VkApplicationInfo appInfo{}; 
appInfo.sType = VK_STRUCTURE_TYPE_APPLICATION_INFO; 
appInfo.pApplicationName = "Hello Triangle"; 
appInfo.applicationVersion = VK_MAKE_VERSION(1, 0, 0); 
appInfo.pEngineName = "No Engine"; 
appInfo.engineVersion = VK_MAKE_VERSION(1, 0, 0); 
appInfo.apiVersion = VK_API_VERSION_1_0;
```

# Validation layers

## What are validation layers?
vulkan是一层很薄的封装，为了更好榨取性能，其做了非常少的有关验证的操作。故此，为了更方便我们开发和测试，我们需要开启validation layers来帮助我们进行vulkan程序的开发。
## Using validation layers
```c++
//通过一个flag来判断编译版本是否为debug版本，如果是那么就开启validation layers.
#ifdef NDEBUG
    const bool enableValidationLayers = false;
#else
    const bool enableValidationLayers = true;
#endif

```
随后我们要查看我们的硬件是否支持该层,于是添加函数`checkkValidationLayerSupport()`
```c++
class HelloTriangleApplication
{  
	......
    private:
        //validation layer
        bool checkValidationLayerSupport();        
	......
};

bool HelloTriangleApplication::checkValidationLayerSupport()
{
    uint32_t layerCount;
    vkEnumerateInstanceLayerProperties(&layerCount, nullptr);

    std::vector<VkLayerProperties> availableLayers(layerCount);
    vkEnumerateInstanceLayerProperties(&layerCount, availableLayers.data());

    for (const char* layerName : validationLayers)
    {
        bool layerFound = false;
        for (const auto& layerProperties : availableLayers)
        {
            if (strcmp(layerName, layerProperties.layerName) == 0)
            {
                layerFound = true;
                break;
            }
        }
        if (!layerFound)
            return false;
    }
    return true;
}

//在createInstance()开始的时候使用上面函数

void HelloTriangleApplication::createInstacne()
{
	if (enableValidationLayers && !checkValidationLayerSupport())
		throw std::runtime_error("validation layers requested, but not 
	    available!");
	......
}

```
在初始化VkInstance的时候我们就需要配置其启用的层了。于是在`createInstance()`添加一下代码。
```c++
    if(enableValidationLayers)
    {
        createInfo.enabledLayerCount = static_cast<uint32_t>(validationLayers.size());
        createInfo.ppEnabledLayerNames = validationLayers.data();
    }
    else
        createInfo.enabledLayerCount = 0;
```
## Message callback
这时候，我们可以指定其debug的信息输出的位置。这时候需要配置其callback函数。
在此之前我们需要启用`VK_EXT_debug_utils`extension
但同样的我们需要在`VkInstance`初始化的时候配置其启用的拓展。
于是我们写一个函数专门在函数`void createInstance()`来启用其拓展,同时获取glfw所需拓展。
```c++
class HelloTriangleApplication
{  
	......
    private:
        //validation layer
    ......     
        std::vector<const char*> getRequiredExtensions(); 
	......
};

std::vector<const char*> HelloTriangleApplication::getRequiredExtensions()
{
    uint32_t glfwExtensionCount = 0;
    const char** glfwExtensions;
    glfwExtensions = glfwGetRequiredInstanceExtensions(&glfwExtensionCount);
    std::vector<const char*> extensions(glfwExtensions, glfwExtensions + glfwExtensionCount);
    
    if (enableValidationLayers)
        extensions.push_back(VK_EXT_DEBUG_UTILS_EXTENSION_NAME);
    return extensions;
}
```
于是在`createInstance()`里面启用其拓展
```c++
void HelloTriangleApplication::createInstance()
{
    auto extensions = getRequiredExtensions();
    createInfo.enabledExtensionCount = static_cast<uint32_t>(extensions.size());
    createInfo.ppEnabledExtensionNames = extensions.data();
}
```
这时候需要其`callback()`函数了.
```c++
class HelloTriangleApplication
{  
	......
    private:
        //validation layer
    ......  
           static VKAPI_ATTR VkBool32 VKAPI_CALL debugCallback(VkDebugUtilsMessageSeverityFlagBitsEXT messageSeverity, VkDebugUtilsMessageTypeFlagsEXT messageType, const VkDebugUtilsMessengerCallbackDataEXT* pCallbackData, void* pUserData); 
	......
};

VKAPI_ATTR VkBool32 VKAPI_CALL HelloTriangleApplication::debugCallback(VkDebugUtilsMessageSeverityFlagBitsEXT messageSeverity, VkDebugUtilsMessageTypeFlagsEXT messageType, const VkDebugUtilsMessengerCallbackDataEXT* pCallbackData, void* pUserData)
{
    if (messageSeverity >= VK_DEBUG_UTILS_MESSAGE_SEVERITY_WARNING_BIT_EXT)
        std::cerr << "validation layer: " << pCallbackData->pMessage << std::endl;
    return VK_FALSE;
}
```
这时候我们需要`VkDebugUtilsMessengerEXT`来进行对`debugCallback()`来进行管理。创建函数`void setupDebugMessenger()`专门负责其初始化。并在函数`void initVulkan()`里面调用。

```c++
class HelloTriangleApplication
{  
	......
    private:
    ......     
        VkDebugUtilsMessengerEXT debugMessenger;
	......
		void setupDebugMessenger();
};

void HelloTriangleApplication::initVulkan()
{
	......
	setupDebugMessenger();
	......
}

```
但是其初始化的函数是一个扩展函数，我们必须获取构造该对象实例的函数的地址，以及销毁该对象实例的函数的地址。这时候填写两个函数辅助我们构造`VkDebugUtilsMessengerEXT`。
```C++
VkResult CreateDebugUtilsMessengerEXT(VkInstance instance, const VkDebugUtilsMessengerCreateInfoEXT* pCreateInfo, const VkAllocationCallbacks* pAllocator, VkDebugUtilsMessengerEXT* pDebugMessenger)
{
    auto func = (PFN_vkCreateDebugUtilsMessengerEXT) vkGetInstanceProcAddr(instance, "vkCreateDebugUtilsMessengerEXT");
    if (func != nullptr)
        return func(instance, pCreateInfo, pAllocator, pDebugMessenger);
    else
        return VK_ERROR_EXTENSION_NOT_PRESENT;

}

void DestroyDebugUtilsMessengerEXT(VkInstance instance, VkDebugUtilsMessengerEXT debugMessenger, const VkAllocationCallbacks* pAllocator)
{
    auto func = (PFN_vkDestroyDebugUtilsMessengerEXT) vkGetInstanceProcAddr(instance, "vkDestroyDebugUtilsMessengerEXT");
    if (func != nullptr)
        func(instance, debugMessenger, pAllocator);
}
```
同时再写一个辅助函数`populateDebugMessengerCreateInfo()`填充结构体`VkDebugUtilsMessengerCreateInfoEXT`。
```c++
class HelloTriangleApplication
{  
	......
    private:
	......   
		void populateDebugMessengerCreateInfo(VkDebugUtilsMessengerCreateInfoEXT& createInfo);
};

void HelloTriangleApplication::populateDebugMessengerCreateInfo(VkDebugUtilsMessengerCreateInfoEXT& createInfo)

{

    createInfo = {};

    createInfo.sType = VK_STRUCTURE_TYPE_DEBUG_UTILS_MESSENGER_CREATE_INFO_EXT;

    createInfo.messageSeverity = VK_DEBUG_UTILS_MESSAGE_SEVERITY_VERBOSE_BIT_EXT | VK_DEBUG_UTILS_MESSAGE_SEVERITY_WARNING_BIT_EXT | VK_DEBUG_UTILS_MESSAGE_SEVERITY_ERROR_BIT_EXT;

    createInfo.messageType = VK_DEBUG_UTILS_MESSAGE_TYPE_GENERAL_BIT_EXT | VK_DEBUG_UTILS_MESSAGE_TYPE_VALIDATION_BIT_EXT | VK_DEBUG_UTILS_MESSAGE_TYPE_PERFORMANCE_BIT_EXT;

    createInfo.pfnUserCallback = debugCallback;

}
```
这时候回去填充函数`setupDebugMessenger()`，并在cleanup()中销毁。
```c++
void HelloTriangleApplication::setupDebugMessenger()
{
    if (!enableValidationLayers)
        return ;
    VkDebugUtilsMessengerCreateInfoEXT createInfo;
    populateDebugMessengerCreateInfo(createInfo);
    if (CreateDebugUtilsMessengerEXT(instance, &createInfo, nullptr, &debugMessenger) != VK_SUCCESS)
        throw std::runtime_error("failed to set up debug messenger!");
}

void HelloTriangleApplication::cleanup()
{
	......
    if (enableValidationLayers)
        DestroyDebugUtilsMessengerEXT(instance, debugMessenger, nullptr);
    ......
}

```
### glfwGetRequiredInstanceExtensions();
This function returns an array of names of Vulkan instance extensions required by GLFW for creating Vulkan surfaces for GLFW windows. If successful, the list will always contain `VK_KHR_surface`, so if you don't require any additional extensions you can pass this list directly to the `VkInstanceCreateInfo` struct.
If Vulkan is not available on the machine, this function returns `NULL` and generates a [GLFW_API_UNAVAILABLE](https://natoune.github.io/glfwJS/variables/GLFW_API_UNAVAILABLE.html) error. Call [glfwVulkanSupported](https://natoune.github.io/glfwJS/functions/glfwVulkanSupported.html) to check whether Vulkan is at least minimally available.
If Vulkan is available but no set of extensions allowing window surface creation was found, this function returns `NULL`. You may still use Vulkan for off-screen rendering and compute work.
#### Parameters
- ##### count: number
    
    Where to store the number of extensions in the returned array. This is set to zero if an error occurred.
    
#### Returns string
An array of ASCII encoded extension names, or `NULL` if an [error](https://www.glfw.org/docs/3.3.2/intro_guide.html#error_handling) occurred.
## Debugging instance creation and destruction
这时候因为`setupDebugMessenger()`的调用在`createInstance()`后，这样会导致，如果在函数`createInstance()`出错的话，则其错误无法输出到所期望的地方，又或者说，直到`setupDebugMessenger()`调用后`debugCallback()`才会被调用。这时候有一个方式可以在`createInstance()`就能正确调用`debugCallback()`。
修改`createInstance()`如下。
```c++
void HelloTriangleApplication::createInstance()
{
    ......
    VkDebugUtilsMessengerCreateInfoEXT debugCreateInfo{};
    if(enableValidationLayers)
    {
        createInfo.enabledLayerCount = static_cast<uint32_t>(validationLayers.size());
        createInfo.ppEnabledLayerNames = validationLayers.data();
        populateDebugMessengerCreateInfo(debugCreateInfo);
        createInfo.pNext = (VkDebugUtilsMessengerCreateInfoEXT*) &debugCreateInfo;
    }
    else
    {
        createInfo.enabledLayerCount = 0;
        createInfo.pNext = nullptr;
    }
    ......
}
```


# Physical devices and queue families
在这一章节，我们需要选择一张物理显卡。不同的物理显卡支持的队列簇不一样，于是这就和队列簇(queue families)产生了联系。这时候我们会检查该物理显卡是否支持某个队列簇来帮我们选择显卡，同时我们还希望是一张独立显卡。
## Selecting a physical device
首先声明我们需要初始化的结构体`VkPhysicalDevice`。
然后声明我们的函数`void pickPhysicalDevice()`并在函数`void initVulkan()`中调用它。
为了方便我们选择不同类型的显卡，我们还需要声明一个函数`bool isDeviceSuitable()`。
```c++
class HelloTriangleApplication
{  

    private:
	   ...... 
	   VkPhysicalDevice physicalDevice = VK_NULL_HANDLE;
	   ......
       //Physical devices and queue families
	   void pickPhysicalDevice();
       bool isDeviceSuitable(VkPhysicalDevice device);
};

void HelloTriangleApplication::pickPhysicalDevice()
{

}

bool HelloTriangleApplication::isDeviceSuitable(VkPhysicalDevice device)
{

}

void HelloTriangleApplication::initVulkan()
{
	......
    pickPhysicalDevice();
}
```
填充函数`void pickPhysicalDevice()`
```c++
void HelloTriangleApplication::pickPhysicalDevice()
{
    uint32_t deviceCount = 0;
    vkEnumeratePhysicalDevices(instance, &deviceCount, nullptr);
    if (deviceCount == 0)
        throw std::runtime_error("failed to find GPUs with Vulkan support!");
    std::vector<VkPhysicalDevice> devices(deviceCount);
    vkEnumeratePhysicalDevices(instance, &deviceCount, devices.data());
    for (const auto& device : devices)
    {
        if(isDeviceSuitable(device))
        {
            physicalDevice = device;
            break;
        }
    }
    if (physicalDevice == VK_NULL_HANDLE)
        throw std::runtime_error("failed to find a suitable GPU!");
}
```
## Base device suitability checks
在这个章节我们会举一个对我们的显卡进行基础的suitability checks的例子。具体是查看显卡是否是独立显卡以及是否支持geometryShader。
于是填充`void isDeviceSuitable()`如下。
```c++
bool isDeviceSuitable(VkPhysicalDevice device)
{
	VkPhysicalDeviceProperties deviceProperties;
	VkPhysicalDeviceFeatures deviceFeatures;
	vkGetPhysicalDeviceProperties(device, &deviceProperties);
	vkGetPhysicalDeviceFeatures(device, &deviceFeatures);

	return deviceProperties.deviceType == VK_PHYSICAL_DEVICE_TYPE_DISCRETE_GPU && deviceFeatures.geometryShader;
}
```
或者你也可以通过给每个显卡计算其分数，最后选一个分数高的。(例如如果该显卡是独立显卡，则添加100分),但我们在后续并不会使用这个方式。我们后面使用的是下一章结所定义的函数。该章节可作为一个拓展知识，了解如何得到自己需要的显卡。
```c++
void pickPhysicalDevice() { ... // Use an ordered map to automatically sort candidates by increasing score 
	std::multimap<int, VkPhysicalDevice> candidates; 

	for (const auto& device : devices) 
	{ 
		int score = rateDeviceSuitability(device); 
		candidates.insert(std::make_pair(score, device)); 
	} 
	// Check if the best candidate is suitable at all 
	if (candidates.rbegin()->first > 0) 
	{ 
		physicalDevice = candidates.rbegin()->second; 
	} else { 
		throw std::runtime_error("failed to find a suitable GPU!"); 
	} 
}

int rateDeviceSuitability(VkPhysicalDevice device) 
{ 
	... 
	int score = 0; 
	// Discrete GPUs have a significant performance advantage 
	if (deviceProperties.deviceType == VK_PHYSICAL_DEVICE_TYPE_DISCRETE_GPU) 
	{ 
		score += 1000; 
	} 
	
	// Maximum possible size of textures affects graphics quality 
	score += deviceProperties.limits.maxImageDimension2D; 
	
	// Application can't function without geometry shaders 
	if (!deviceFeatures.geometryShader) 
	{ 
		return 0; 
	} 
	return score; 
}
```
## Queue families
我们需要一个辅助函数来帮我们找到不同显卡的所支持队列簇`uint32_t findQueueFamilies()`。同时声明一种结构体保存不同队列簇的下表`QueueFamilyIndices()`
tips:因为理论上uint32_t并没有特殊值表示某个队列簇下表不存在，于是我们使用std::optional<>。
这时候在`bool isDeviceSuitable()`中调用上述函数。
```c++
struct QueueFamilyIndices
{
    std::optional<uint32_t> graphicsFamily;
    bool isComplete()
    {
        return graphicsFamily.has_value();
    }
};
class HelloTriangleApplication
{  
    private:
	    ...... 
        //Physical devices and queue families
		......
        QueueFamilyIndices findQueuFamilies(VkPhysicalDevice device);
};

QueueFamilyIndices HelloTriangleApplication::findQueuFamilies(VkPhysicalDevice device)
{
    QueueFamilyIndices indices;
    uint32_t queueFamilyCount = 0;
    vkGetPhysicalDeviceQueueFamilyProperties(device, &queueFamilyCount, nullptr);
    std::vector<VkQueueFamilyProperties> queueFamilies(queueFamilyCount);
    vkGetPhysicalDeviceQueueFamilyProperties(device, &queueFamilyCount, queueFamilies.data());
    int i = 0;
    for (const auto& queueFamily : queueFamilies)
    {
        if (queueFamily.queueFlags & VK_QUEUE_GRAPHICS_BIT)
            indices.graphicsFamily = i;
        if (indices.isComplete())
            break;
        i++;
    }
    return indices;
}

bool HelloTriangleApplication::isDeviceSuitable(VkPhysicalDevice device)
{
    QueueFamilyIndices indices = findQueuFamilies(device);
    return indices.graphicsFamily.has_value();
}
```
# Logical device and queues
## Introduction
我们并不是直接控制`PhysicalDevice`的，而是通过`VkDevice //Logical device`来对物理设备操控。同时，你可以在同一个`PhysicalDevice`上，创建多个`VkDevice`即Logical device。
当然，在创建`VkDevice`时我们需要启用对应的`queue families`。
这时，首先添加类成员`VkDevice device`。
如后，添加声明`void createLogicalDevice()`，并在函数`void initVulkan()`中调用。
```c++
class HelloTriangleApplication
{  
    private:
		......
        VkDevice device;
		......
        //Logical device and queues
        void createLogicalDevice();
};

void HelloTriangleApplication::createLogicalDevice()
{

}

void HelloTriangleApplication::initVulkan()
{
	......
    createLogicalDevice();
}
```
## Specifying the queues to be created
这时候我们应该填充结构体`VkDeviceQueueCreateInfo`来启用我们的队列簇。我们还需要获取对应物理设备下该队列簇的下标，这时候可以同时上一个大章节有关`QueueFamilyIndices`了解其获取方式。
tips:你可以在一个队列簇下创建多个队列，但是在我们的例子中我们只创建一个，其实把不同线程产生的命令统一在主线程中单个队列一块提交，也就是说在简单的使用场景下，我们只需要一个队列。同时我们还必须设置队列的优先级，这里单个队列直接设为1.0就可以了(范围[0.0 - 1.0])。
```c++
void HelloTriangleApplication::createLogicalDevice()
{
    QueueFamilyIndices indices = findQueuFamilies(physicalDevice);
    VkDeviceQueueCreateInfo queueCreateInfo{};
    queueCreateInfo.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
    queueCreateInfo.queueFamilyIndex = indices.graphicsFamily.value();
    queueCreateInfo.queueCount = 1;

    float queuePriority = 1.0f;
    queueCreateInfo.pQueuePriorities = &queuePriority;
  

}
```
## Specifying used device features
这时候我们还可以启用不同的设备上的功能。但目前来说，我们并不需要其他功能，但是为了结构完整，我们简单加一行代码就行。
```c++
void HelloTriangleApplication::createLogicalDevice()
{
    QueueFamilyIndices indices = findQueuFamilies(physicalDevice);
    VkDeviceQueueCreateInfo queueCreateInfo{};
    queueCreateInfo.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
    queueCreateInfo.queueFamilyIndex = indices.graphicsFamily.value();
    queueCreateInfo.queueCount = 1;

    float queuePriority = 1.0f;
    queueCreateInfo.pQueuePriorities = &queuePriority;
	
	//new add
	VkPhysicalDeviceFeatures deviceFeatures{};
}
```
## Creating the logical device
创建logical device和之前的instance类似，需要填充`VkDeviceCreateInfo`结构体。并在`void cleanup()`中对`VkDevice`销毁。
```c++
void HelloTriangleApplication::createLogicalDevice()
{
    QueueFamilyIndices indices = findQueuFamilies(physicalDevice);
    VkDeviceQueueCreateInfo queueCreateInfo{};
    queueCreateInfo.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
    queueCreateInfo.queueFamilyIndex = indices.graphicsFamily.value();
    queueCreateInfo.queueCount = 1;

    float queuePriority = 1.0f;
    queueCreateInfo.pQueuePriorities = &queuePriority;

    VkPhysicalDeviceFeatures deviceFeatures{};
   // new add
    VkDeviceCreateInfo createInfo{};
    createInfo.sType = VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO;
    createInfo.pQueueCreateInfos = &queueCreateInfo;
    createInfo.queueCreateInfoCount = 1;
    createInfo.pEnabledFeatures = &deviceFeatures;
    createInfo.enabledExtensionCount = 0;
    if (enableValidationLayers)
    {
        createInfo.enabledLayerCount = static_cast<uint32_t>(validationLayers.size());
        createInfo.ppEnabledLayerNames = validationLayers.data();
    }
    else
        createInfo.enabledLayerCount = 0;
    if (vkCreateDevice(physicalDevice, &createInfo, nullptr, &device) != VK_SUCCESS)
        throw std::runtime_error("failed to create logical device!");
}

void HelloTriangleApplication::cleanup()
{
    vkDestroyDevice(device, nullptr);
	......
}
```
## Retrieving queue handles
我们还需要获取我们创建的queue的句柄即初始化`VkQueue`。并在`createLogicalDevice()`后获取其句柄。
```c++
class HelloTriangleApplication
{  
    private:
		......
        VkQueue graphicsQueue;
		......
};

void HelloTriangleApplication::createLogicalDevice()
{
	......
    vkGetDeviceQueue(device, indices.graphicsFamily.value(), 0, &graphicsQueue);\
}
```
# Window surface
vulkan并没有在电脑系统上面显示图片的能力。这部分的功能是通过拓展完成的。这时候我们应该启用一个扩展`VK_KHR_surface`，但我们早就启用的该拓展，因为在`glfwGetRequiredInstanceExtensions`的函数使用中就以及包含了该拓展。同时GLFW帮我们封装了跨平台的系统窗口的模块，我们直接使用其提供的功能。
surface是用于**连接渲染结果与显示窗口**的核心对象。
## Window surface creation
这时候我们应该初始化一个对象`VkSurfaceKHR`。并声明定义函数`void createSurface()`，在`void initVulkan()`中调用，在`void cleanup()`函数中销毁`VkSurfaceKHR`。同时在surface会影响我们选择显卡`VkPhysicalDevice`，则因该在`pickPhysicalDevice()`前调用。
```c++
class HelloTriangleApplication
{  
    private:
	    ......
        VkSurfaceKHR surface;
		......
        //Window surface
        void createSurface();
		......
};

void HelloTriangleApplication::initVulkan()
{
	......
    setupDebugMessenger();
    createSurface();
    pickPhysicalDevice();
	......
}

void HelloTriangleApplication::cleanup()
{
	......
    vkDestroySurfaceKHR(instance, surface, nullptr);
    vkDestroyInstance(instance, nullptr);
	......
}
```
## Querying for presentation support
因为在队列簇中`graphicsFamily`(负责绘制)和`presentFamily`(负责显示)可能不是同一个队列簇，于是我们应该修改结构体`QueueFamilyIndices`以及函数`findQueueFamilies`来找到两种队列簇。
```c++
struct QueueFamilyIndices
{
    std::optional<uint32_t> graphicsFamily;
    std::optional<uint32_t> presentFamily;

    bool isComplete()
    {
        return graphicsFamily.has_value() && presentFamily.has_value();
    }
};

QueueFamilyIndices HelloTriangleApplication::findQueuFamilies(VkPhysicalDevice device)
{
    QueueFamilyIndices indices;

    uint32_t queueFamilyCount = 0;
    vkGetPhysicalDeviceQueueFamilyProperties(device, &queueFamilyCount, nullptr);

    std::vector<VkQueueFamilyProperties> queueFamilies(queueFamilyCount);
    vkGetPhysicalDeviceQueueFamilyProperties(device, &queueFamilyCount, queueFamilies.data());

    int i = 0;
    for (const auto& queueFamily : queueFamilies)
    {
        if (queueFamily.queueFlags & VK_QUEUE_GRAPHICS_BIT)
            indices.graphicsFamily = i;

		//new add
        VkBool32 presentSupport = false;
        vkGetPhysicalDeviceSurfaceSupportKHR(device, i, surface, &presentSupport);
        if (presentSupport)
            indices.presentFamily = i;
            
        if (indices.isComplete())
            break;
        i++;
    }
    return indices;
}

```
## Creating the presentation queue
这时候应该在类中添加presentFamily的句柄，`VkQueue presentQueue`。这时候在构造逻辑设备的函数中添加`presentFamily`的配置。
```c++
class HelloTriangleApplication
{  
    private:
	    ......
        VkQueue presentQueue;
		......
};

void HelloTriangleApplication::createLogicalDevice()
{
    QueueFamilyIndices indices = findQueuFamilies(physicalDevice);

	//new add
    std::vector<VkDeviceQueueCreateInfo> queueCreateInfos;
    std::set<uint32_t> uniqueQueueFamilies = { indices.graphicsFamily.value(), indices.presentFamily.value() };

    float queuePriority = 1.0f;
    for (uint32_t queueFamily : uniqueQueueFamilies)
    {
        VkDeviceQueueCreateInfo queueCreateInfo{};
        queueCreateInfo.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
        queueCreateInfo.queueFamilyIndex = queueFamily;
        queueCreateInfo.queueCount = 1;
        queueCreateInfo.pQueuePriorities = &queuePriority;
        queueCreateInfos.push_back(queueCreateInfo);
    }
  
    VkPhysicalDeviceFeatures deviceFeatures{};
    
    VkDeviceCreateInfo createInfo{};
    createInfo.sType = VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO;

	//new add
    createInfo.queueCreateInfoCount = static_cast<uint32_t>(queueCreateInfos.size());
    createInfo.pQueueCreateInfos = queueCreateInfos.data();

    createInfo.pEnabledFeatures = &deviceFeatures;
    createInfo.enabledExtensionCount = 0;

    if (enableValidationLayers)
    {
        createInfo.enabledLayerCount = static_cast<uint32_t>(validationLayers.size());
        createInfo.ppEnabledLayerNames = validationLayers.data();
    }
    else
        createInfo.enabledLayerCount = 0;
  
    if (vkCreateDevice(physicalDevice, &createInfo, nullptr, &device) != VK_SUCCESS)
        throw std::runtime_error("failed to create logical device!");
  
    vkGetDeviceQueue(device, indices.graphicsFamily.value(), 0, &graphicsQueue);
	//new add
    vkGetDeviceQueue(device, indices.presentFamily.value(), 0, &presentQueue);

}
```

# Swap chain
显卡渲染以及pc上系统的显示之间的关系，是提供交换链(Swap chain)来进行配置的。这也是我们这一章节主要目标，配置好一个Swap chain
## Checking for swap chain support
首先vulkan并没有提供能够显示在PC显示器上面的接口，所以我们还是需要向glfw来获取其VK_KHR_swapchain的拓展。
首先同验证层一样，通过一个字符串数组来显示例举我们先要的设备的拓展。
然后，在选择设备的时候添加一个负责函数`bool checkDeviceExtensionSupport()`来专门检测其设备是拥有我们需要的拓展。并在`bool isDeviceSuitable()`里进行调用。
```c++
const std::vector<const char*> deviceExtensions = {
    VK_KHR_SWAPCHAIN_EXTENSION_NAME
};

class HelloTriangleApplication
{  
    private:
		......
        //Swap chain
        bool checkDeviceExtensionSupport(VkPhysicalDevice device);
};

bool HelloTriangleApplication::checkDeviceExtensionSupport(VkPhysicalDevice device)
{  
    uint32_t extensionCount;
    vkEnumerateDeviceExtensionProperties(device, nullptr, &extensionCount, nullptr);
    std::vector<VkExtensionProperties> availableExtensions(extensionCount);
    vkEnumerateDeviceExtensionProperties(device, nullptr, &extensionCount, availableExtensions.data());
    std::set<std::string> requiredExtensions(deviceExtensions.begin(), deviceExtensions.end());
    for (const auto& extension : availableExtensions)
        requiredExtensions.erase(extension.extensionName);
  
    return requiredExtensions.empty();
}

bool HelloTriangleApplication::isDeviceSuitable(VkPhysicalDevice device)
{
    QueueFamilyIndices indices = findQueuFamilies(device);
    bool extensionsSupported = checkDeviceExtensionSupport(device);

    return indices.graphicsFamily.has_value() && extensionsSupported;
}
```
## Enabling device extensions
在构造`VkDevice`也就是(logic device)时，需要指定其启用的拓展。即修改`void createLogicDevice()`如下。
```c++
void HelloTriangleApplication::createLogicalDevice()
{
	......
    VkDeviceCreateInfo createInfo{};
    createInfo.sType = VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO;
  
    createInfo.queueCreateInfoCount = static_cast<uint32_t>(queueCreateInfos.size());
    createInfo.pQueueCreateInfos = queueCreateInfos.data();

    createInfo.pEnabledFeatures = &deviceFeatures;
   //modify  
    createInfo.enabledExtensionCount = static_cast<uint32_t>(deviceExtensions.size());
    createInfo.ppEnabledExtensionNames = deviceExtensions.data();
	......
}
```
## Querying details of swap chain support
在这个章节我们需要填充一个结构体`SwapChainSupportDetails`，其保存了三个重要信息。
1. `VkSurfaceCapabilitiesKHR capabilities`  描述**表面的基础能力限制**，定义交换链创建的边界条件。[VkSurfaceCapabilitiesKHR(3)](https://registry.khronos.org/vulkan/specs/latest/man/html/VkSurfaceCapabilitiesKHR.html)
2. `std::vector<VkSurfaceFormatKHR> formats`枚举**表面支持的像素格式和颜色空间**，确保渲染输出与显示设备兼容[VkSurfaceFormatKHR(3)](https://registry.khronos.org/vulkan/specs/latest/man/html/VkSurfaceFormatKHR.html)
3. `std::vector<VkPresentModeKHR> presentModes`定义**图像从交换链到屏幕的呈现方式**，直接影响渲染流畅度和延迟。[VkPresentModeKHR(3)](https://registry.khronos.org/vulkan/specs/latest/man/html/VkPresentModeKHR.html)

我们还需声明一个函数`SwapChainSupportDetails querySwapChainSupport()`专门填充其结构。
并在函数`bool isDeviceSuitable()`中调用，检测其显卡设备是否存在能够支持的`formats`以及`presentModes`。(最起码要支持一种)
```c++
struct SwapChainSupportDetails
{
    VkSurfaceCapabilitiesKHR capabilities;
    std::vector<VkSurfaceFormatKHR> formats;
    std::vector<VkPresentModeKHR> presentModes;
};

class HelloTriangleApplication
{  
    private:
	    ......
        //Swap chain
        bool checkDeviceExtensionSupport(VkPhysicalDevice device);
        SwapChainSupportDetails querySwapChainSupport(VkPhysicalDevice device);
		......
};

SwapChainSupportDetails HelloTriangleApplication::querySwapChainSupport(VkPhysicalDevice device)
{
    SwapChainSupportDetails details;
    vkGetPhysicalDeviceSurfaceCapabilitiesKHR(device, surface, &details.capabilities);
    
    uint32_t formatCount;
    vkGetPhysicalDeviceSurfaceFormatsKHR(device, surface, &formatCount, nullptr);

    if (formatCount != 0)
    {
        details.formats.resize(formatCount);
        vkGetPhysicalDeviceSurfaceFormatsKHR(device, surface, &formatCount, details.formats.data());
    }

    uint32_t presentModeCount;
    vkGetPhysicalDeviceSurfacePresentModesKHR(device, surface, &presentModeCount, nullptr);

    if (presentModeCount != 0)
    {
        details.presentModes.resize(presentModeCount);
        vkGetPhysicalDeviceSurfacePresentModesKHR(device, surface, &presentModeCount, details.presentModes.data());
    }

    return details;
}

bool HelloTriangleApplication::isDeviceSuitable(VkPhysicalDevice device)
{
    QueueFamilyIndices indices = findQueuFamilies(device);
    bool extensionsSupported = checkDeviceExtensionSupport(device);
    
    bool swapChainAdequate = false;
    if (extensionsSupported)
    {
        SwapChainSupportDetails swapChainSupport = querySwapChainSupport(device);
        swapChainAdequate = !swapChainSupport.formats.empty() && !swapChainSupport.presentModes.empty();
    }
    
    return indices.isComplete() && extensionsSupported && swapChainAdequate;
}
```
## Choosing the right settings for the swap chain
在获取到其所支持的swapchain的三个重要信息后，我们需要对他进行选择。后续分三个小章节来定义不同的函数供我们使用。
### Surface format
定义函数`VkSurfaceFormatKHR chooseSwapSurfaceFormat();`
```c++
class HelloTriangleApplication
{  
    private:
	    ......
        //Swap chain
        ......
        VkSurfaceFormatKHR chooseSwapSurfaceFormat(const std::vector<VkSurfaceFormatKHR>& availableFormats);
		......
};

VkSurfaceFormatKHR HelloTriangleApplication::chooseSwapSurfaceFormat(const std::vector<VkSurfaceFormatKHR>& availableFormats)
{
    for (const auto& availableFormat : availableFormats)
    {
        if (availableFormat.format == VK_FORMAT_B8G8R8A8_SRGB && availableFormat.colorSpace == VK_COLOR_SPACE_SRGB_NONLINEAR_KHR)
            return availableFormat;
    }  
  
    return availableFormats[0];
}
```
### Presentation mode
定义函数`VkPresentModeKHR chooseSwapPresentMode()`
- `VK_PRESENT_MODE_IMMEDIATE_KHR`: 图片渲染完之后直接提交到显示屏，可能会导致画面撕裂。
- `VK_PRESENT_MODE_FIFO_KHR`: 使用一个队列去保存渲染完成的图片，这样可以保证画面不会被撕裂但，同时如果队列满了会导致程序阻塞。(可以理解为垂直同步)
- `VK_PRESENT_MODE_FIFO_RELAXED_KHR`: 并非严格的队列，在程序卡顿时，可以避免FIFO掉帧出现的卡顿感，但是增加撕裂。
- `VK_PRESENT_MODE_MAILBOX_KHR`: 并非严格的队列，可以避免程序阻塞，但队列满时，会对现有的图片进行替换。
```c++
class HelloTriangleApplication
{  
    private:
	    ......
        //Swap chain
        ......
        VkPresentModeKHR chooseSwapPresentMode(const std::vector<VkPresentModeKHR>& availablePresentModes);
		......
};

VkPresentModeKHR HelloTriangleApplication::chooseSwapPresentMode(const std::vector<VkPresentModeKHR>& availablePresentModes)
{
    for (const auto& availablePresentMode : availablePresentModes)
    {
        if (availablePresentMode == VK_PRESENT_MODE_MAILBOX_KHR)
            return availablePresentMode;
    }

    return VK_PRESENT_MODE_FIFO_KHR;
}
```
### Swap extent
其定义了交换链图像的分辨率（像素宽度和高度），即最终渲染到屏幕的图像尺寸。
定义函数`VkExtent2D chooseSwapExtent()`
```c++
class HelloTriangleApplication
{  
    private:
	    ......
        //Swap chain
        ......
        VkExtent2D chooseSwapExtent(const VkSurfaceCapabilitiesKHR& capabilities);
		......
};

VkExtent2D HelloTriangleApplication::chooseSwapExtent(const VkSurfaceCapabilitiesKHR& capabilities)
{
    if (capabilities.currentExtent.width != std::numeric_limits<uint32_t>::max())
        return capabilities.currentExtent;
    else
    {
        int width, height;
        glfwGetFramebufferSize(window, &width, &height);
        VkExtent2D actualExtent = {
            static_cast<uint32_t>(width),
            static_cast<uint32_t>(height)
        };
        actualExtent.width = std::clamp(actualExtent.width, capabilities.minImageExtent.width, capabilities.maxImageExtent.width);
        actualExtent.height = std::clamp(actualExtent.height, capabilities.minImageExtent.height, capabilities.maxImageExtent.height);
        return actualExtent;
    }
}
```
## Creating the swap chain
我们需要创建`VkSwapChainKHR`实例，对此我们需要填充`VkSwapchainCreateInfoKHR`[VkSwapchainCreateInfoKHR(3)](https://registry.khronos.org/vulkan/specs/latest/man/html/VkSwapchainCreateInfoKHR.html)。
创建函数`void createSwapChain()`并在`void initVulkan()`中调用。并使用我们前面章节获取的结构体对齐进行部分填充。并在`void cleanup()`中销毁。
需要注意的是：
```c++
class HelloTriangleApplication
{  
    private:
	    ......
		VkSwapchainKHR swapChain;
		......
        //Swap chain
        ......
        void createSwapChain();
		......
};

void HelloTriangleApplication::createSwapChain()
{
    SwapChainSupportDetails swapChainSupport = querySwapChainSupport(physicalDevice);
    
    VkSurfaceFormatKHR surfaceFormat = chooseSwapSurfaceFormat(swapChainSupport.formats);
    VkPresentModeKHR presentMode = chooseSwapPresentMode(swapChainSupport.presentModes);
    VkExtent2D extent = chooseSwapExtent(swapChainSupport.capabilities);

    uint32_t imageCount = swapChainSupport.capabilities.minImageCount + 1;
    if (swapChainSupport.capabilities.maxImageCount > 0 &&  imageCount > swapChainSupport.capabilities.maxImageCount)
        imageCount = swapChainSupport.capabilities.maxImageCount;
  
    VkSwapchainCreateInfoKHR createInfo{};
    createInfo.sType = VK_STRUCTURE_TYPE_SWAPCHAIN_CREATE_INFO_KHR;
    createInfo.surface = surface;
    createInfo.minImageCount = imageCount;
    createInfo.imageFormat = surfaceFormat.format;
    createInfo.imageColorSpace = surfaceFormat.colorSpace;
    createInfo.imageExtent = extent;
    createInfo.imageArrayLayers = 1;
    createInfo.imageUsage = VK_IMAGE_USAGE_COLOR_ATTACHMENT_BIT;

    QueueFamilyIndices indices = findQueuFamilies(physicalDevice);
    uint32_t queueFamilyIndices[] = { indices.graphicsFamily.value(), indices.presentFamily.value() };
    if (indices.graphicsFamily != indices.presentFamily)
    {
        createInfo.imageSharingMode = VK_SHARING_MODE_CONCURRENT;
        createInfo.queueFamilyIndexCount = 2;
        createInfo.pQueueFamilyIndices = queueFamilyIndices;
    }
    else
        createInfo.imageSharingMode = VK_SHARING_MODE_EXCLUSIVE;

    createInfo.preTransform = swapChainSupport.capabilities.currentTransform;
    createInfo.compositeAlpha = VK_COMPOSITE_ALPHA_OPAQUE_BIT_KHR;
    createInfo.presentMode = presentMode;
    createInfo.clipped = VK_TRUE;
    createInfo.oldSwapchain = VK_NULL_HANDLE;
    if (vkCreateSwapchainKHR(device, &createInfo, nullptr, &swapChain) != VK_SUCCESS)
        throw std::runtime_error("failed to create swap chain!");
}

void HelloTriangleApplication::initVulkan()
{
	......
    createSwapChain();
}

void HelloTriangleApplication::cleanup()
{
    vkDestroySwapchainKHR(device, swapChain, nullptr);
	......
}

```
## Retrieving the swap chain images
成功创建`swap chain`后我们还需要保存某些结构体以便后面使用，分别是`VkImage`,`VkFormat`,`VkExtent2D`。并在`void createSwapChain()`添加负责保存结构的代码。
```c++
class HelloTriangleApplication
{  
    private:
	    ......
	    VkSwapchainKHR swapChain;
        std::vector<VkImage> swapChainImages;
        VkFormat swapChainImageFormat;
        VkExtent2D swapChainExtent;
		......
};

void HelloTriangleApplication::createSwapChain()
{
	......
    if (vkCreateSwapchainKHR(device, &createInfo, nullptr, &swapChain) != VK_SUCCESS)
        throw std::runtime_error("failed to create swap chain!");
        
	//new add
    vkGetSwapchainImagesKHR(device, swapChain, &imageCount, nullptr);
    swapChainImages.resize(imageCount);
    vkGetSwapchainImagesKHR(device, swapChain, &imageCount, swapChainImages.data());
    swapChainImageFormat = surfaceFormat.format;
    swapChainExtent = extent;
}

```

# Image views
在Swap chain隐式构造了`VkImage`，这时为了使用其`VkImage`，我们需要构造对应的`VkImageView`，其描述了如何使用该图像。
- `VkImage` 仅表示**原始的图像内存块**（如交换链图像、纹理贴图、深度缓冲等），不包含如何解释其数据的元信息。
- `VkImageView` 则声明了：
    - **像素格式**（如 `VK_FORMAT_R8G8B8A8_SRGB`）
        
    - **图像类型**（1D/2D/3D/立方体贴图）
        
    - **子资源范围**（访问哪些 Mip 层级/数组层）  
        _👉 类比：_ `VkImage` 是一块未切割的生肉，`VkImageView` 是决定切块方式（切片/切丁）和烹饪方法（煎/烤）的食谱。
首先，添加`VkImageView`声明在类里面。
其次声明定义添加函数`void createImageViews()`，并在`void initVulkan()`中调用，并在`void cleanup()`中销毁。我们需要使用`VkImageViewCreateInfo`来配置其内容。[VkImageViewCreateInfo(3)](https://registry.khronos.org/vulkan/specs/latest/man/html/VkImageViewCreateInfo.html)
```c++
class HelloTriangleApplication
{  
    private:
		......
        std::vector<VkImageView> swapChainImageViews;
		......
        //Image view
        void createImageView();
		......
};

void HelloTriangleApplication::createImageView()
{
    swapChainImageViews.resize(swapChainImages.size());
    for (size_t i = 0; i < swapChainImages.size(); i++)
    {
        VkImageViewCreateInfo createInfo{};
        createInfo.sType = VK_STRUCTURE_TYPE_IMAGE_VIEW_CREATE_INFO;
        createInfo.image = swapChainImages[i];
        createInfo.viewType = VK_IMAGE_VIEW_TYPE_2D;
        createInfo.format = swapChainImageFormat;
        createInfo.components.r = VK_COMPONENT_SWIZZLE_IDENTITY;
        createInfo.components.g = VK_COMPONENT_SWIZZLE_IDENTITY;
        createInfo.components.b = VK_COMPONENT_SWIZZLE_IDENTITY;
        createInfo.components.a = VK_COMPONENT_SWIZZLE_IDENTITY;
        createInfo.subresourceRange.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
        createInfo.subresourceRange.baseMipLevel = 0;
        createInfo.subresourceRange.levelCount = 1;
        createInfo.subresourceRange.baseArrayLayer = 0;
        createInfo.subresourceRange.layerCount = 1;
        if (vkCreateImageView(device, &createInfo, nullptr, &swapChainImageViews[i]) != VK_SUCCESS)
            throw std::runtime_error("failed to create image views!");
    }
}

void HelloTriangleApplication::initVulkan()
{
	......
    createImageView();
}

void HelloTriangleApplication::cleanup()
{
    for (auto imageView : swapChainImageViews)
        vkDestroyImageView(device, imageView, nullptr);
	......
}
```
# Graphics Pipeline basics
该章节简单的加个函数定义即可，有关渲染管线的自行去了解。
添加`void createGraphicsPipeline()`声明，并在`void initVulkan()`中调用。
```c++
class HelloTriangleApplication
{  
    private:
	    ......
        //Graphics pipeline
        void createGraphicsPipeline();
		......
};

void HelloTriangleApplication::createGraphicsPipeline()
{

}

void HelloTriangleApplication::initVulkan()
{
	......
    createImageView();
    createGraphicsPipeline();
	......
}
```
# Shader modules
vulkan使用的是SPIR-V作为shader程序，这就需要我们首先编写好对应的GLSL或者HLSL后，通过编译器编译成对应的SPIR-V文件后，由vulkan提交给GPU。
## Vertex shade
新建一个文件，名为shader.vert。
```glsl
#version 450
layout(location = 0) out vec3 fragColor;

vec2 position[3] = vec2[](
    vec2(0.0, -0.5),
    vec2(0.5, 0.5),
    vec2(-0.5, 0.5)
);

vec3 colors[3] = vec3[](
    vec3(1.0, 0.0, 0.0),
    vec3(0.0, 1.0, 0.0),
    vec3(0.0, 0.0, 1.0)
);

void main()
{
    gl_Position = vec4(position[gl_VertexIndex], 0.0, 1.0);
    fragColor = colors[gl_VertexIndex];
}
```
## Fragment shader
新建一个文件，名为shader.frag。
```glsl
#version 450
layout (location = 0) in vec3 fragColor;
layout (location = 0) out vec4 outColor;

void main()
{
    outColor = vec4(fragColor, 1.0);  
}
```
## Compiling the shaders
通过glslc编译其shader
```bash
/home/user/VulkanSDK/x.x.x.x/x86_64/bin/glslc shader.vert -o vert.spv
/home/user/VulkanSDK/x.x.x.x/x86_64/bin/glslc shader.frag -o frag.spv
```
## Loading a shader
声明函数`std::vector<char> readFile()`来进行对shader文件的读取。并在函数`void createGraphicsPipeline()`中调用读取其SPIR-V文件。在这里我使用了辅助函数帮我找到我的文件路径，与你文件放的位置有关，你可以以自己的方式定义自己的辅助函数，只要最好的路径是对的就行。
```c++
class HelloTriangleApplication
{  
    private:
		......
        //Graphics pipeline
        void createGraphicsPipeline();
        static std::vector<char> readFile(const std::string& filename);
		......
};

std::vector<char> HelloTriangleApplication::readFile(const std::string& filename)
{
    std::ifstream file(filename, std::ios::ate | std::ios::binary);
    if (!file.is_open())
        throw std::runtime_error("failed to open file!");

    size_t fileSize = (size_t)file.tellg();
    std::vector<char> buffer(fileSize);
  
    file.seekg(0);
    file.read(buffer.data(), fileSize);
    file.close();

    return buffer;
}

void HelloTriangleApplication::createGraphicsPipeline()
{
    const std::string basePath = GetExecutionPath();

    auto vertShaderCode = readFile(basePath + "/assets/shaders/vert.spv");
    auto fragShaderCode = readFile(basePath + "/assets/shaders/frag.spv");

}

```
## Creating shader modules
我们需要创建`VkShaderModule`来传递shader代码给GPU。
这时候我们声明函数`VkShaderModule createShaderModule()`辅助函数来方便我们构造对应的`VkShaderModule`。并在`void createGraphicsPipeline()`中调用。并在传输给GPU后销毁，即在该函数末尾销毁。
```c++
class HelloTriangleApplication
{  
    private:
		......
        //Graphics pipeline
        void createGraphicsPipeline();
        static std::vector<char> readFile(const std::string& filename);
        VkShaderModule createShaderModule(const std::vector<char>& code);
		......
};


VkShaderModule HelloTriangleApplication::createShaderModule(const std::vector<char>& code)
{
    VkShaderModuleCreateInfo createInfo{};
    createInfo.sType = VK_STRUCTURE_TYPE_SHADER_MODULE_CREATE_INFO;
    createInfo.codeSize = code.size();
    createInfo.pCode = reinterpret_cast<const uint32_t*>(code.data());

    VkShaderModule shaderModule;
    if (vkCreateShaderModule(device, &createInfo, nullptr, &shaderModule) != VK_SUCCESS)
        throw std::runtime_error("failed to create shader module!");

    return shaderModule;
}

void HelloTriangleApplication::createGraphicsPipeline()
{
    const std::string basePath = GetExecutionPath();

    auto vertShaderCode = readFile(basePath + "/assets/shaders/vert.spv");
    auto fragShaderCode = readFile(basePath + "/assets/shaders/frag.spv");
  
    VkShaderModule vertShaderModule = createShaderModule(vertShaderCode);
    VkShaderModule fragShaderModule = createShaderModule(fragShaderCode);



    vkDestroyShaderModule(device, fragShaderModule, nullptr);
    vkDestroyShaderModule(device, fragShaderModule, nullptr);
}

```
## Shader stage creation
这时候便可以在构造pipeline的时候绑定对应的阶段的shader了。于是在`void createGraphicsPipeline()`中添加如下代码。
```c++
void HelloTriangleApplication::createGraphicsPipeline()
{
	......
    VkShaderModule vertShaderModule = createShaderModule(vertShaderCode);
    VkShaderModule fragShaderModule = createShaderModule(fragShaderCode);
	//new add
    VkPipelineShaderStageCreateInfo vertShaderStageInfo{};
    vertShaderStageInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO;
    vertShaderStageInfo.stage = VK_SHADER_STAGE_VERTEX_BIT;
    vertShaderStageInfo.module = vertShaderModule;
    vertShaderStageInfo.pName = "main";
  
    VkPipelineShaderStageCreateInfo fragShaderStageInfo{};
    fragShaderStageInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO;
    fragShaderStageInfo.stage = VK_SHADER_STAGE_FRAGMENT_BIT;
    fragShaderStageInfo.module = fragShaderModule;
    fragShaderStageInfo.pName = "main";

    VkPipelineShaderStageCreateInfo shahderStages[] = { vertShaderStageInfo, fragShaderStageInfo };



    vkDestroyShaderModule(device, fragShaderModule, nullptr);

    vkDestroyShaderModule(device, fragShaderModule, nullptr);

}
```
# Fixed functions
在以前的API大多会为渲染管线提供一个默认的状态。但在Vulkan中需要你自己去显示的配置其中各个部分，
## Dynamic state
动态状态（Dynamic State）机制允许在渲染过程中动态修改某些管线状态，而不是在创建管线时固定这些状态。
在函数`void createGraphicsPipeline()`函数里面。
首先声明一个`VkDynamicState`数组，指定哪些内容可以被动态的修改。并填充`VkPipelineDynamicStateCreateInfo`。
```c++
void HelloTriangleApplication::createGraphicsPipeline()
{
	......
    VkPipelineShaderStageCreateInfo shahderStages[] = { vertShaderStageInfo, fragShaderStageInfo };

    std::vector<VkDynamicState> dynamicStates = {
        VK_DYNAMIC_STATE_VIEWPORT,
        VK_DYNAMIC_STATE_SCISSOR
    };
    VkPipelineDynamicStateCreateInfo dynamicState{};
    dynamicState.sType = VK_STRUCTURE_TYPE_PIPELINE_DYNAMIC_STATE_CREATE_INFO;
    dynamicState.dynamicStateCount = static_cast<uint32_t>(dynamicStates.size());
    dynamicState.pDynamicStates = dynamicStates.data();


    vkDestroyShaderModule(device, fragShaderModule, nullptr);
    vkDestroyShaderModule(device, fragShaderModule, nullptr);

}
```
## Vertex input
关于Vertex input我们需要配置`VkPipelineVertexInputStateCreateInfo`结构体。但我们目前没有外部输入的Vertex所以简单添加如下代码即可。
```c++
void HelloTriangleApplication::createGraphicsPipeline()
{
	......
    VkPipelineShaderStageCreateInfo shahderStages[] = { vertShaderStageInfo, fragShaderStageInfo };
    
    VkPipelineVertexInputStateCreateInfo vertexInputInfo{};
    vertexInputInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_VERTEX_INPUT_STATE_CREATE_INFO;
    vertexInputInfo.vertexBindingDescriptionCount = 0;
    vertexInputInfo.vertexAttributeDescriptionCount = 0;
	......
}
```
## Input assembly
这一章节的主要目的是配置结构体`VkPipelineInputAssemblyStateCreateInfo`，其规定了两样东西：如何将顶点组装成几何图元以及是否允许在绘制过程中重启图元序列
我们目前只是按照顶点顺序画一个三角形，所以简单按照如下填充便好。
```c++
void HelloTriangleApplication::createGraphicsPipeline()
{
	......
    VkPipelineVertexInputStateCreateInfo vertexInputInfo{};
    vertexInputInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_VERTEX_INPUT_STATE_CREATE_INFO;
    vertexInputInfo.vertexBindingDescriptionCount = 0;
    vertexInputInfo.vertexAttributeDescriptionCount = 0;

    VkPipelineInputAssemblyStateCreateInfo inputAssembly{};
    inputAssembly.sType = VK_STRUCTURE_TYPE_PIPELINE_INPUT_ASSEMBLY_STATE_CREATE_INFO;
    inputAssembly.topology = VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST;
    inputAssembly.primitiveRestartEnable = VK_FALSE;
	......
}
```
## Viewports and scissors
`Viewports`将归一化设备坐标（NDC）转换为帧缓冲区坐标，定义几何图形在屏幕上的位置和深度范围。在`void createGraphicsPipeline()`添加如下代码。
我们需要填充结构体`VkPipelineViewportStateCreateInfo`。
```c++
void HelloTriangleApplication::createGraphicsPipeline()
{
	......
    VkPipelineInputAssemblyStateCreateInfo inputAssembly{};

    inputAssembly.sType = VK_STRUCTURE_TYPE_PIPELINE_INPUT_ASSEMBLY_STATE_CREATE_INFO;
    inputAssembly.topology = VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST;
    inputAssembly.primitiveRestartEnable = VK_FALSE;

    VkPipelineViewportStateCreateInfo viewportState{};
    viewportState.sType = VK_STRUCTURE_TYPE_PIPELINE_VIEWPORT_STATE_CREATE_INFO;
    viewportState.viewportCount = 1;
    viewportState.scissorCount = 1;
	......
}
```
### Viewports Example
```c++
VkViewport viewport = {
    .x = 100.0f,       // 从屏幕左侧100像素开始
    .y = 200.0f,       // 从屏幕底部200像素开始（Vulkan坐标系Y向上）
    .width = 400.0f,   // 宽度400像素
    .height = 300.0f,  // 高度300像素
    .minDepth = 0.0f,  // 最小深度
    .maxDepth = 1.0f   // 最大深度
};
```
- NDC坐标 `(-1, -1)` → 屏幕坐标 `(100, 200)`
    
- NDC坐标 `(1, 1)` → 屏幕坐标 `(500, 500)`

### Scissors Example
`scissors`定义 **像素级别的裁剪区域**，只有在此矩形内的像素才会被光栅化，之外的像素直接丢弃。
```c++
VkRect2D scissor = {
    .offset = {50, 50},   // 左下角坐标 (50,50)
    .extent = {300, 200}  // 宽300像素，高200像素
};
```
- 只有 `(50,50)` 到 `(350,250)` 区域内的像素会被处理
## Rasterizer
这一章节是配置光栅化阶段的操作，即执行fragment shader的阶段。我们需要填充结构体`VkPipelineRasterizationStateCreateInfo`来配置其流程。在函数`void createGraphicsPipeline()`中添加如下代码。
```c++
void HelloTriangleApplication::createGraphicsPipeline()
{
	......
    VkPipelineViewportStateCreateInfo viewportState{};
    viewportState.sType = VK_STRUCTURE_TYPE_PIPELINE_VIEWPORT_STATE_CREATE_INFO;
    viewportState.viewportCount = 1;
    viewportState.scissorCount = 1;

    VkPipelineRasterizationStateCreateInfo rasterizer{};
    rasterizer.sType = VK_STRUCTURE_TYPE_PIPELINE_RASTERIZATION_STATE_CREATE_INFO;
    rasterizer.depthClampEnable = VK_FALSE;
    rasterizer.rasterizerDiscardEnable = VK_FALSE;
    rasterizer.polygonMode = VK_POLYGON_MODE_FILL;
    rasterizer.lineWidth = 1.0f;
    rasterizer.cullMode = VK_CULL_MODE_BACK_BIT;
    rasterizer.depthBiasEnable = VK_FALSE;
	......
}
```
## Multisampling
在此章节我们要填充结构体`VkPipelineMultisampleStateCreateInfo`来配置多重采样，其作用是可以抗锯齿。于是在`void createGraphicsPipeline()`添加如下代码。
```c++
void HelloTriangleApplication::createGraphicsPipeline()
{
	......
    VkPipelineRasterizationStateCreateInfo rasterizer{};
    rasterizer.sType = VK_STRUCTURE_TYPE_PIPELINE_RASTERIZATION_STATE_CREATE_INFO;
    rasterizer.depthClampEnable = VK_FALSE;
    rasterizer.rasterizerDiscardEnable = VK_FALSE;
    rasterizer.polygonMode = VK_POLYGON_MODE_FILL;
    rasterizer.lineWidth = 1.0f;
    rasterizer.cullMode = VK_CULL_MODE_BACK_BIT;
    rasterizer.depthBiasEnable = VK_FALSE;
  
    VkPipelineMultisampleStateCreateInfo multisampling{};
    multisampling.sType = VK_STRUCTURE_TYPE_PIPELINE_MULTISAMPLE_STATE_CREATE_INFO;
    multisampling.sampleShadingEnable = VK_FALSE;
    multisampling.rasterizationSamples = VK_SAMPLE_COUNT_1_BIT;
	......
}
```
## Depth and stencil testing
关于深度和模板，我们暂且不需要特殊配置。
## Color blending
这一章节我们需要填充结构体`VkPipelineColorBlendAttachmentState`定义单个颜色附件的混合行为。如果有多个颜色附件（如 MRT），需要为每个附件单独配置。
以及`VkPipelineColorBlendStateCreateInfo`定义整个混合阶段的全局设置，包括所有颜色附件的混合状态。，来完成在fragment shader结束后有关颜色混合的配置。
```c++
void HelloTriangleApplication::createGraphicsPipeline()
{
	......
    VkPipelineMultisampleStateCreateInfo multisampling{};
    multisampling.sType = VK_STRUCTURE_TYPE_PIPELINE_MULTISAMPLE_STATE_CREATE_INFO;
    multisampling.sampleShadingEnable = VK_FALSE;
    multisampling.rasterizationSamples = VK_SAMPLE_COUNT_1_BIT;

    VkPipelineColorBlendAttachmentState colorBlendAttachment{};
    colorBlendAttachment.colorWriteMask = VK_COLOR_COMPONENT_R_BIT | VK_COLOR_COMPONENT_G_BIT | VK_COLOR_COMPONENT_B_BIT | VK_COLOR_COMPONENT_A_BIT;
    colorBlendAttachment.blendEnable = VK_FALSE;

    VkPipelineColorBlendStateCreateInfo colorBlending{};
    colorBlending.sType = VK_STRUCTURE_TYPE_PIPELINE_COLOR_BLEND_STATE_CREATE_INFO;
    colorBlending.logicOpEnable = VK_FALSE;
    colorBlending.logicOp = VK_LOGIC_OP_COPY;
    colorBlending.attachmentCount = 1;
    colorBlending.pAttachments = &colorBlendAttachment;
    colorBlending.blendConstants[0] = 0.0f;
    colorBlending.blendConstants[1] = 0.0f;
    colorBlending.blendConstants[2] = 0.0f;
    colorBlending.blendConstants[3] = 0.0f;
	......
}
```
## Pipeline layout
这一章节我们需要填充结构体`VkPipelineLayout`。当你需要向vertex shader传递uniform属性的数据时，就需要通过该结构体配置，但目前来说我们并不需要。在`void createGraphicsPipeline()`添加如下代码。
```c++
class HelloTriangleApplication
{  
    private:
		......
        VkPipelineLayout pipelineLayout;
		......
};

void HelloTriangleApplication::createGraphicsPipeline()
{
	......
    VkPipelineDynamicStateCreateInfo dynamicState{};
    dynamicState.sType = VK_STRUCTURE_TYPE_PIPELINE_DYNAMIC_STATE_CREATE_INFO;
    dynamicState.dynamicStateCount = static_cast<uint32_t>(dynamicStates.size());
    dynamicState.pDynamicStates = dynamicStates.data();

    VkPipelineLayoutCreateInfo pipelineLayoutInfo{};
    pipelineLayoutInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_LAYOUT_CREATE_INFO;
    pipelineLayoutInfo.setLayoutCount = 0;
    pipelineLayoutInfo.pushConstantRangeCount = 0;

    if (vkCreatePipelineLayout(device, &pipelineLayoutInfo, nullptr, &pipelineLayout) != VK_SUCCESS)
        throw std::runtime_error("failed to create pipeline layout!");
	......
}
```
# Render passes
在`Graphics pipeline`创建之前，我们需要知道我们在`Graphics pipeline`需要使用到多少`framebuffer attachments`例如color and depth buffers 以及 samples。
## Setup
声明对象`VkRenderPass`，并声明函数`void createRenderPass()`。然后在`void initVulkan()`中在`createGraphicsPipeline()`前调用。
```c++
class HelloTriangleApplication
{  
    private:
	    ......
        //RenderPass
        void createRenderPass();
		......
};

void HelloTriangleApplication::createRenderPass()
{

}

void HelloTriangleApplication::initVulkan()
{
	......
    createImageView();
    createRenderPass();
    createGraphicsPipeline();
}
```
## Attachment description
我们这个例程只需要一个color buffer attachment 就可以了，于是在`void createRenderPass()`添加如下代码。需要填充结构体`VkAttachmentDescription`
```c++
void HelloTriangleApplication::createRenderPass()
{
    VkAttachmentDescription colorAttachment{};
    colorAttachment.format = swapChainImageFormat;
    colorAttachment.samples = VK_SAMPLE_COUNT_1_BIT;
    colorAttachment.loadOp = VK_ATTACHMENT_LOAD_OP_CLEAR;
    colorAttachment.storeOp = VK_ATTACHMENT_STORE_OP_STORE;
    colorAttachment.stencilLoadOp = VK_ATTACHMENT_LOAD_OP_DONT_CARE;
    colorAttachment.stencilStoreOp = VK_ATTACHMENT_STORE_OP_DONT_CARE;
    colorAttachment.initialLayout = VK_IMAGE_LAYOUT_UNDEFINED;
    colorAttachment.finalLayout = VK_IMAGE_LAYOUT_PRESENT_SRC_KHR;
}
```
## Subpasses and attachment references
一个render pass 可以由多个subpasses。以便完成其他操作比如后处理。但是我们只是画一个三角形，我们只需要单一subpass就行。
于是填充两个结构体`VkAttachmentReference`和`VkSubpassDescription`在函数`void createRenderPass()`中。
```C++
void HelloTriangleApplication::createRenderPass()
{
    colorAttachment.finalLayout = VK_IMAGE_LAYOUT_PRESENT_SRC_KHR;
	......
    VkAttachmentReference colorAttachmentRef{};
    colorAttachmentRef.attachment = 0;
    colorAttachmentRef.layout = VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL;
  
    VkSubpassDescription subpass{};
    subpass.pipelineBindPoint = VK_PIPELINE_BIND_POINT_GRAPHICS;
    subpass.colorAttachmentCount = 1;
    subpass.pColorAttachments = &colorAttachmentRef;
}
```
## Render pass
这时候可以回去声明结构体实例`VkRenderPass`，并在`void createRenderPass()`中填充最好的结构体`VkRenderPassCreateInfo`。并在`void cleanup()`中销毁。
```c++
class HelloTriangleApplication
{  
    private:
		......
        VkRenderPass renderPass;
        VkPipelineLayout pipelineLayout;
		......
};

void HelloTriangleApplication::createRenderPass()
{
	......
    VkRenderPassCreateInfo renderPassInfo{};
    renderPassInfo.sType = VK_STRUCTURE_TYPE_RENDER_PASS_CREATE_INFO;
    renderPassInfo.attachmentCount = 1;
    renderPassInfo.pAttachments = &colorAttachment;
    renderPassInfo.subpassCount = 1;
    renderPassInfo.pSubpasses = &subpass;
    if (vkCreateRenderPass(device, &renderPassInfo, nullptr, &renderPass) != VK_SUCCESS)
        throw std::runtime_error("failed to create render pass!");
}

void HelloTriangleApplication::cleanup()
{
    vkDestroyPipelineLayout(device, pipelineLayout, nullptr);
    vkDestroyRenderPass(device, renderPass, nullptr);
	......
}

```
# Conclusion
在这一章节我们终于可以整合前面章节的结构体构造GraphicsPipeline了。
- Shader stages: the shader modules that define the functionality of the programmable stages of the graphics pipeline 
- Fixed-function state: all of the structures that define the fixed-function stages of the pipeline, like input assembly, rasterizer, viewport and color blending
- Pipeline layout: the uniform and push values referenced by the shader that can be updated at draw time
- Render pass: the attachments referenced by the pipeline stages and their usage
这时候我们终于可以填充结构体`VkGraphicsPipelineCreateInfo`了。在`void createGraphicsPipeline()`中添加如下代码。并在`void cleanup()`中销毁`VkPipeline`。
```c++
class HelloTriangleApplication
{  
    private:
		......
        VkRenderPass renderPass;
        VkPipelineLayout pipelineLayout;
        VkPipeline graphicsPipeline;
		......
};
void HelloTriangleApplication::createGraphicsPipeline()
{
    if (vkCreatePipelineLayout(device, &pipelineLayoutInfo, nullptr, &pipelineLayout) != VK_SUCCESS)
        throw std::runtime_error("failed to create pipeline layout!");
        
	......
    VkGraphicsPipelineCreateInfo pipelineInfo{};
    pipelineInfo.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
    pipelineInfo.stageCount = 2;
    pipelineInfo.pStages = shaderStages;
    pipelineInfo.pVertexInputState = &vertexInputInfo;
    pipelineInfo.pInputAssemblyState = &inputAssembly;
    pipelineInfo.pViewportState = &viewportState;
    pipelineInfo.pRasterizationState = &rasterizer;
    pipelineInfo.pMultisampleState = &multisampling;
    pipelineInfo.pColorBlendState = &colorBlending;
    pipelineInfo.pDynamicState = &dynamicState;
    pipelineInfo.layout = pipelineLayout;
    pipelineInfo.renderPass = renderPass;
    pipelineInfo.subpass = 0;
    pipelineInfo.basePipelineHandle = VK_NULL_HANDLE;

    if (vkCreateGraphicsPipelines(device, VK_NULL_HANDLE, 1, &pipelineInfo, nullptr, &graphicsPipeline) != VK_SUCCESS)
        throw std::runtime_error("failed to create graphics pipeline!");
	......
}

void HelloTriangleApplication::cleanup()
{
    vkDestroyPipeline(device, graphicsPipeline, nullptr);
	......
}
```

# Framebuffers
这一章我们应该为每一个`VkImageView`来创建其对应的`VkFramebuffer`。我们需要填充结构体`VkFramebufferCreateInfo`。
声明成员`std::vector<VkFramebuffer>`以及函数`void createFramebuffer()`，并在`void initVulkan()`中调用，在`void cleanup()`中销毁。
```c++
class HelloTriangleApplication
{  
    private:
		......
        std::vector<VkImageView> swapChainImageViews;
		//new add
        std::vector<VkFramebuffer> swapChainFramebuffers;
		......
        //Framebuffer
        void createFramebuffers();
		......
};

void HelloTriangleApplication::createFramebuffers()
{
    swapChainFramebuffers.resize(swapChainImageViews.size());
    for (size_t i = 0; i < swapChainImageViews.size(); i++)
    {
        VkImageView attachments[] = {
            swapChainImageViews[i]
        };

        VkFramebufferCreateInfo framebufferInfo{};
        framebufferInfo.sType = VK_STRUCTURE_TYPE_FRAMEBUFFER_CREATE_INFO;
        framebufferInfo.renderPass = renderPass;
        framebufferInfo.attachmentCount = 1;
        framebufferInfo.pAttachments = attachments;
        framebufferInfo.width = swapChainExtent.width;
        framebufferInfo.height = swapChainExtent.height;
        framebufferInfo.layers = 1;
  
        if (vkCreateFramebuffer(device, &framebufferInfo, nullptr, &swapChainFramebuffers[i]) != VK_SUCCESS)
            throw std::runtime_error("failed to create framebuffer!");
    }
}

void HelloTriangleApplication::initVulkan()
{
	......
    createFramebuffers();
}

void HelloTriangleApplication::cleanup()
{
    for (auto framebuffer : swapChainFramebuffers)
        vkDestroyFramebuffer(device, framebuffer, nullptr);
	......
}

```
# Command buffers
这时候就应该创建我们的`Command buffers`了，如果我们需要提交一个`draw call`或者`memory transfers`都需要通过`Command buffers`。这个对象则是专门负责提交命令给Vulkan的。
## Command pools
在创建`Command buffers`之前，我们需要创建`Command pools`，因为我们需要通过`Command pools`来分配内存与`Command buffers`。这时我们需要填充`VkCommandPoolCreateInfo`。
首先声明成员`CommandPool`。
其次，声明函数`void createCommandPool()`并在`void initVulkan()`中调用，在`void cleanup()`中销毁。
```c++
class HelloTriangleApplication
{  
    private:
	    ......
        VkCommandPool commandPool;
		......
        //Command buffers
        void createCommandPool();
		......
};

void HelloTriangleApplication::createCommandPool()
{
    QueueFamilyIndices queueFamilyIndices = findQueuFamilies(physicalDevice);
    VkCommandPoolCreateInfo poolInfo{};
    poolInfo.sType = VK_STRUCTURE_TYPE_COMMAND_POOL_CREATE_INFO;
    poolInfo.flags = VK_COMMAND_POOL_CREATE_RESET_COMMAND_BUFFER_BIT;
    poolInfo.queueFamilyIndex = queueFamilyIndices.graphicsFamily.value();
    if (vkCreateCommandPool(device, &poolInfo, nullptr, &commandPool) != VK_SUCCESS)
        throw std::runtime_error("failed to create command pool!");
}

void HelloTriangleApplication::initVulkan()
{
	......
    createCommandPool();
}

void HelloTriangleApplication::cleanup()
{
    vkDestroyCommandPool(device, commandPool, nullptr);
	......
}
```
## Command buffer allocation
这时候便可以分配我们的`Command buffer`了。这时我们需要填充`VkCommandBufferAllocateInfo`。
首先声明成员`VkCommandBuffer`。
其次，声明函数`void createCommandBuffer()`并在`void initVulkan()`中调用。
```c++
class HelloTriangleApplication
{  
    private:
		......
        VkCommandPool commandPool;
		......
        void createCommandBuffer();
};

void HelloTriangleApplication::createCommandBuffer()
{
    VkCommandBufferAllocateInfo allocInfo{};
    allocInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_ALLOCATE_INFO;
    allocInfo.commandPool = commandPool;
    allocInfo.level = VK_COMMAND_BUFFER_LEVEL_PRIMARY;
    allocInfo.commandBufferCount = 1;

    if (vkAllocateCommandBuffers(device, &allocInfo, &commandBuffer) != VK_SUCCESS)
        throw std::runtime_error("failed to allocate command buffer!");
}

void HelloTriangleApplication::initVulkan()
{
	......
    createCommandBuffer();
}
```
## Command buffer recording
这一章节主要目的是，完成一个函数收集我们的命令提交给`CommandBuffer`去渲染。
声明函数`void recordCommandBuffer()`并开始录制命令。即填充结构体`VkCommandBufferBeginInfo`是命令缓冲的"总开关"。
```c++
class HelloTriangleApplication
{  
    private:
		......
        void recordCommandBuffer(VkCommandBuffer commandBuffer, uint32_t imageIndex);
};

void HelloTriangleApplication::recordCommandBuffer(VkCommandBuffer commandBuffer, uint32_t imageIndex)
{
    VkCommandBufferBeginInfo beginInfo{};
    beginInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_BEGIN_INFO;
    if (vkBeginCommandBuffer(commandBuffer, &beginInfo) != VK_SUCCESS)
        throw std::runtime_error("failed to begin recording command buffer!");
}
```
## Starting a render pass
该章节的目的是配置一个`renderPass`的开始。即我们需要填充`VkRenderPassBeginInfo`渲染流程的"启动器"。添加在函数`void recordCommandBuffer()`里。
```c++
void HelloTriangleApplication::recordCommandBuffer(VkCommandBuffer commandBuffer, uint32_t imageIndex)
{
	......
    VkRenderPassBeginInfo renderPassInfo{};
    renderPassInfo.sType = VK_STRUCTURE_TYPE_RENDER_PASS_BEGIN_INFO;
    renderPassInfo.renderPass = renderPass;
    renderPassInfo.framebuffer = swapChainFramebuffers[imageIndex];
    renderPassInfo.renderArea.offset = { 0, 0 };
    renderPassInfo.renderArea.extent = swapChainExtent;

    VkClearValue clearColor = { { { 0.0f, 0.0f, 0.0f, 1.0f } } };
    renderPassInfo.clearValueCount = 1;
    renderPassInfo.pClearValues = &clearColor;
    vkCmdBeginRenderPass(commandBuffer, &renderPassInfo, VK_SUBPASS_CONTENTS_INLINE);
}
```
## Basic drawing commands
这时候就是一个drawing的纤细的配置了，目前我们只需配置前文提到的`VkViewport`以及`VkRect2D`。添加在函数`void recordCommandBuffer()`里。
```c++
void HelloTriangleApplication::recordCommandBuffer(VkCommandBuffer commandBuffer, uint32_t imageIndex)
{
	......
    vkCmdBeginRenderPass(commandBuffer, &renderPassInfo, VK_SUBPASS_CONTENTS_INLINE);
		
        vkCmdBindPipeline(commandBuffer, VK_PIPELINE_BIND_POINT_GRAPHICS, graphicsPipeline);
        VkViewport viewport{};
        viewport.x = 0.0;
        viewport.y = 0.0;
        viewport.width = (float)swapChainExtent.width;
        viewport.height = (float)swapChainExtent.height;
        viewport.minDepth = 0.0f;
        viewport.maxDepth = 1.0f;
        vkCmdSetViewport(commandBuffer, 0, 1, &viewport);
        
        VkRect2D scissor{};
        scissor.offset = { 0, 0 };
        scissor.extent = swapChainExtent;
        vkCmdSetScissor(commandBuffer, 0, 1, &scissor);
        vkCmdDraw(commandBuffer, 3, 1, 0, 0);

}
```
## Finishing up
在末尾我们还需要结束录制。在函数`void recordCommandBuffer()`末尾添加如下代码。
```c++
void HelloTriangleApplication::recordCommandBuffer(VkCommandBuffer commandBuffer, uint32_t imageIndex)
{
        vkCmdDraw(commandBuffer, 3, 1, 0, 0);
	......
    vkCmdEndRenderPass(commandBuffer);
    
    if (vkEndCommandBuffer(commandBuffer) != VK_SUCCESS)
        throw std::runtime_error("failed to record command buffer!");
}
```
# Rendering and presentation
这时候便可以声明函数`void drawFrame()`在`void mainloop()`中调用了，这便是渲染的主要逻辑。
```c++
class HelloTriangleApplication
{  
    private:
		......
        //Rendering and presentation
        void drawFrame();
};

void HelloTriangleApplication::drawFrame()
{

}

void HelloTriangleApplication::mainLoop()
{
    while (!glfwWindowShouldClose(window)) {
        glfwPollEvents();
        drawFrame();

    }
}
```
## Outline of a frame
在此之前，但是我们并没有联系`swap chain` 、 `windows system`和`GPU`，三者之间的同步关系。在接下来几个章节我们就需要完成简单的依赖关系。
比如等待前一帧的渲染。
向`swap chain`获取一个`image`。
然后使用该`image`去录制`command buffer`。
然后`submit` `command buffer`中的命令
最后`present` `swap chain` 中的`image`。
## Synchronization
有两种同步变量，分别是`Semaphores`和`Fences`，其中`Semaphores`负责`GPU`内部同步，而`Fences`负责同步`CPU`与`GPU`。
### Semaphores
其主要的作用是同步不同`Queue`之间的提交
```c++
VkCommandBuffer A, B = ... // record command buffers VkSemaphore 
S = ... // create a semaphore 
// enqueue A, signal S when done - starts executing immediately 
vkQueueSubmit(work: A, signal: S, wait: None) 
// enqueue B, wait on S to start 
vkQueueSubmit(work: B, signal: None, wait: S)
```
### Fences
其作用是让`CPU`等待某个任务的完成。
```c++
VkCommandBuffer A = ... // record command buffer with the transfer VkFence 
F = ... // create the fence 
// enqueue A, start work immediately, signal F when done 
vkQueueSubmit(work: A, fence: F) 

vkWaitForFence(F) // blocks execution until A has finished executing 

save_screenshot_to_disk() // can't run until the transfer has finished
```
### What to choose?
对于`swap chain`其发生在`GPU`的操作我们使用`Semaphores`，对于主机等待上一帧绘制结束这种例子，我们会使用`Fences`因为这涉及到`CPU`与`GPU`间的同步。
## Creating the synchronization objects
现在开始创建两个`Semaphores`以及一个`Fences`。
首先声明两个`VkSemaphores`类成员，一个`VkFence`类成员。
然后声明函数`void createSyncObject()`，填充结构体`VkSemaphoreCreateInfo`， 以及`VkFenceCreateInfo`。
并在`void initVulkan()`中调用，`void cleanup()`中销毁。
```c++
class HelloTriangleApplication
{  
    private:
		......
        VkSemaphore imageAvailableSemaphore;
        VkSemaphore renderFinishedSemaphore;
        VkFence inFlightFence;
		......
        //Rendering and presentation
        void drawFrame();
        void createSyncObjects();
		......
};

void HelloTriangleApplication::createSyncObjects()
{
    VkSemaphoreCreateInfo semaphoreInfo{};
    semaphoreInfo.sType = VK_STRUCTURE_TYPE_SEMAPHORE_CREATE_INFO;
    VkFenceCreateInfo fenceInfo{};
    fenceInfo.sType = VK_STRUCTURE_TYPE_FENCE_CREATE_INFO;
    fenceInfo.flags = VK_FENCE_CREATE_SIGNALED_BIT;
  
    if(vkCreateSemaphore(device, &semaphoreInfo, nullptr, &imageAvailableSemaphore) != VK_SUCCESS ||
        vkCreateSemaphore(device, &semaphoreInfo, nullptr, &renderFinishedSemaphore) != VK_SUCCESS ||
        vkCreateFence(device, &fenceInfo, nullptr, &inFlightFence) != VK_SUCCESS
    )
        throw std::runtime_error("failed to create synchronization objects for a frame!");    
}

void HelloTriangleApplication::initVulkan()
{
    ......
    createSyncObjects();
}

void HelloTriangleApplication::cleanup()
{
    vkDestroySemaphore(device, renderFinishedSemaphore, nullptr);
    vkDestroySemaphore(device, imageAvailableSemaphore, nullptr);
    vkDestroyFence(device, inFlightFence, nullptr);
	......
}

```
## Waiting for the previous frame
在下一次执行`void drawFrame()`的主要内容前我们，需要等待上一帧的绘制结束，所以我们会等待我们的`VkFence`然后重置它。
```c++
void HelloTriangleApplication::drawFrame()
{
    vkWaitForFences(device, 1, &inFlightFence, VK_TRUE, UINT64_MAX);
    vkResetFences(device, 1, &inFlightFence);
}
```
## Acquiring an image from the swap chain
在上一帧结束后，我们还需向`swap chain`要一张`image`来进行命令的录制，以便下一帧绘制到该`image`上。
```c++
void HelloTriangleApplication::drawFrame()
{
	......
    uint32_t imageIndex;
    vkAcquireNextImageKHR(device, swapChain, UINT64_MAX, imageAvailableSemaphore, VK_NULL_HANDLE, &imageIndex);
}
```
## Recording the command buffer
随后变可以开始录制命令了。
```c++
void HelloTriangleApplication::drawFrame()
{
	......
    vkResetCommandBuffer(commandBuffer, /*VkCommandBufferResetFlagBits*/ 0);
    recordCommandBuffer(commandBuffer, imageIndex);
}
```
## Submitting the command buffer
随后便可以提交我们录制的命令了，并填充`VkSubmitInfo`。
```c++
void HelloTriangleApplication::drawFrame()
{
	......
    VkSubmitInfo submitInfo{};
    submitInfo.sType = VK_STRUCTURE_TYPE_SUBMIT_INFO;
    VkSemaphore waitSemaphores[] = { imageAvailableSemaphore };
    VkPipelineStageFlags waitStages[] = { VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT };
    submitInfo.waitSemaphoreCount = 1;
    submitInfo.pWaitSemaphores = waitSemaphores;
    submitInfo.pWaitDstStageMask = waitStages;
    submitInfo.commandBufferCount = 1;
    submitInfo.pCommandBuffers = &commandBuffer;
    VkSemaphore signalSemaphores[] = { renderFinishedSemaphore };
    submitInfo.signalSemaphoreCount = 1;;
    submitInfo.pSignalSemaphores = signalSemaphores;

    if (vkQueueSubmit(graphicsQueue, 1, &submitInfo, inFlightFence) != VK_SUCCESS)
        throw std::runtime_error("failed to submit draw command buffer!");
}
```
## Subpass dependencies
注意到，我们是在CPU端获取图像的，但这并不意味着GPU端可以获取到图像，要把两者分开来看。在GPU渲染管线执行前，需要保证GPU获取其image，因为在渲染管线开始阶段便会改变image的布局(layout)。所以我们可以通过dependencies来控制其流程。在`void createRenderPass()`中添加如下代码。
```c++
void HelloTriangleApplication::createRenderPass()
{
	......
    VkSubpassDescription subpass{};
    subpass.pipelineBindPoint = VK_PIPELINE_BIND_POINT_GRAPHICS;
    subpass.colorAttachmentCount = 1;
    subpass.pColorAttachments = &colorAttachmentRef;

    VkSubpassDependency denpendency{};
    denpendency.srcSubpass = VK_SUBPASS_EXTERNAL;
    denpendency.dstSubpass = 0;
    denpendency.srcStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT;
    denpendency.srcAccessMask = 0;
    denpendency.dstStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT;
    denpendency.dstAccessMask = VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT;

    VkRenderPassCreateInfo renderPassInfo{};
    renderPassInfo.sType = VK_STRUCTURE_TYPE_RENDER_PASS_CREATE_INFO;
    renderPassInfo.attachmentCount = 1;
    renderPassInfo.pAttachments = &colorAttachment;
    renderPassInfo.subpassCount = 1;
    renderPassInfo.pSubpasses = &subpass;
    renderPassInfo.dependencyCount = 1;
    renderPassInfo.pDependencies = &denpendency;
	......
}
```
## Presentation
我们还有显示的提交presentation的命令。
```c++
void HelloTriangleApplication::drawFrame()
{
    ......
    VkPresentInfoKHR presentInfo{};
    presentInfo.sType = VK_STRUCTURE_TYPE_PRESENT_INFO_KHR;
    presentInfo.waitSemaphoreCount = 1;
    presentInfo.pWaitSemaphores = signalSemaphores;
    VkSwapchainKHR swapChains[] = { swapChain };
    presentInfo.swapchainCount = 1;
    presentInfo.pSwapchains = swapChains;
    presentInfo.pImageIndices = &imageIndex;
  
    vkQueuePresentKHR(presentQueue, &presentInfo);
}
```
最后再添加上一行代码，需要结束应用时，等待全部全局变量结束，不然会导致某些被持有的资源失效。
```c++
void mainLoop() 
{ 
	while (!glfwWindowShouldClose(window)) 
	{ 
		glfwPollEvents(); 
		drawFrame(); 
	}
	vkDeviceWaitIdle(device); 
}
```
# Frames in flight
## Frames in flight
这一章节主要的目的是，想要CPU提交一个图片的时候去准备下一张图片，只需要保证图片不冲突，结合`swapchain`的功能就可以加速整个过程。不然，你虽然`swapchain`有多张图片，但你每次加工一张然后显示一张很明显性能上有问题。
首先初始化一个变量代表我们有多少帧的循环。
```c++
const int MAX_FRAMES_IN_FLIGHT = 2;
```
这时候关于`Commandbuffer`,`VkSemaphore`,`VkFence`的都需要区分不同帧使用的对象。所以原来的对于改成如下。并声明一个变量代表当前帧。
```c++
//old
class HelloTriangleApplication
{  
    private:
		......
        VkCommandBuffer commandBuffer;
        VkSemaphore imageAvailableSemaphore;
        VkSemaphore renderFinishedSemaphore;
        VkFence inFlightFence;
		......
};

//new 
class HelloTriangleApplication
{  
    private:
		......
        std::vector<VkCommandBuffer> commandBuffers;
        std::vector<VkSemaphore> imageAvailableSemaphores;
        std::vector<VkSemaphore> renderFinishedSemaphores;
        std::vector<VkFence> inFlightFences;
        uint32_t currentFrame = 0;
		......
};
```
这时便去修改函数`void createCommandBuffer()`，函数名修改为`void createCommandBuffers()`。内容修改如下。tips:`void initVulkan()`中的调用别忘了修改。
```c++
void HelloTriangleApplication::createCommandBuffers()
{
    commandBuffers.resize(MAX_FRAMES_IN_FLIGHT);
    VkCommandBufferAllocateInfo allocInfo{};
    allocInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_ALLOCATE_INFO;
    allocInfo.commandPool = commandPool;
    allocInfo.level = VK_COMMAND_BUFFER_LEVEL_PRIMARY;
    allocInfo.commandBufferCount = (uint32_t)commandBuffers.size();/
    if (vkAllocateCommandBuffers(device, &allocInfo, commandBuffers.data()) != VK_SUCCESS)
        throw std::runtime_error("failed to allocate command buffer!");
}
```
当然`void createSyncObjects()`也同样需要修改，如下：
```c++
void HelloTriangleApplication::createSyncObjects()
{
    imageAvailableSemaphores.resize(MAX_FRAMES_IN_FLIGHT);
    renderFinishedSemaphores.resize(MAX_FRAMES_IN_FLIGHT);
    inFlightFences.resize(MAX_FRAMES_IN_FLIGHT);
  
    VkSemaphoreCreateInfo semaphoreInfo{};
    semaphoreInfo.sType = VK_STRUCTURE_TYPE_SEMAPHORE_CREATE_INFO;
    VkFenceCreateInfo fenceInfo{};
    fenceInfo.sType = VK_STRUCTURE_TYPE_FENCE_CREATE_INFO;
    fenceInfo.flags = VK_FENCE_CREATE_SIGNALED_BIT;

    for (size_t i = 0; i < MAX_FRAMES_IN_FLIGHT; i++)
    {
        if(vkCreateSemaphore(device, &semaphoreInfo, nullptr, &imageAvailableSemaphores[i]) != VK_SUCCESS ||
            vkCreateSemaphore(device, &semaphoreInfo, nullptr, &renderFinishedSemaphores[i]) != VK_SUCCESS ||
            vkCreateFence(device, &fenceInfo, nullptr, &inFlightFences[i]) != VK_SUCCESS
        )
            throw std::runtime_error("failed to create synchronization objects for a frame!");    
    }
}
```
同时`void cleanup()`也需要更改对应的销毁。
```c++
void HelloTriangleApplication::cleanup()
{
    for (size_t i = 0; i < MAX_FRAMES_IN_FLIGHT; i++)
    {
        vkDestroySemaphore(device, renderFinishedSemaphores[i], nullptr);
        vkDestroySemaphore(device, imageAvailableSemaphores[i], nullptr);
        vkDestroyFence(device, inFlightFences[i], nullptr);
    }
	......
}
```
这时候便可以在`void drawFrame()`修改，使得不同帧下标使用的资源不同修改如下。
```c++

```
# Swap chain recreation

## Introduction
但窗口系统的surface发生变化时，例如大小上的变化。我们就必须重新生成`swap chain`。
## Recreating the swap chain
这时候定义一个函数`void recreateSwapChain()`。当然在重新生成前，我们必须销毁原有的相关 的资源，例如`SwapChain`,`ImageViews`,`Framebuffers`。也必须等待一些原有的资源释放(信号量)。
还需声明一个辅助函数`void cleanupSwapChain()`，并在`void cleanup()`中调用。
```c++
class HelloTriangleApplication
{  
    private:
		......
        //Swap chain recreation
        void recreateSwapChain();
        void cleanupSwapChain();
		......
};

void HelloTriangleApplication::cleanupSwapChain()
{
    for (size_t i = 0; i < swapChainFramebuffers.size(); i++)
        vkDestroyFramebuffer(device, swapChainFramebuffers[i], nullptr);

    for (size_t i = 0; i < swapChainImageViews.size(); i++)
        vkDestroyImageView(device, swapChainImageViews[i], nullptr);

    vkDestroySwapchainKHR(device, swapChain, nullptr);
}

void HelloTriangleApplication::recreateSwapChain()
{
    vkDeviceWaitIdle(device);
    cleanupSwapChain();
    createSwapChain();
    createImageViews();
    createFramebuffers();
}
```
## Suboptimal or out-of-date swap chain
在函数`vkAcquireNextImageKHR()`的返回值会告诉我们是否需要重新生成`swap chain`。所以修改`void drawFrame()`。
```c++
void HelloTriangleApplication::drawFrame()
{
    vkWaitForFences(device, 1, &inFlightFences[currentFrame], VK_TRUE, UINT64_MAX);
    vkResetFences(device, 1, &inFlightFences[currentFrame]);

    uint32_t imageIndex;
    VkResult result = vkAcquireNextImageKHR(device, swapChain, UINT64_MAX, imageAvailableSemaphores[currentFrame], VK_NULL_HANDLE, &imageIndex);
    if (result == VK_ERROR_OUT_OF_DATE_KHR)
    {
        recreateSwapChain();
        return;
    }
    else if (result != VK_SUCCESS && result != VK_SUBOPTIMAL_KHR)
    {  
        throw std::runtime_error("failed to acquire swap chain image!");
    }
	......
}
```
## Fixing a deadlock
因为我们重新生成`swapchain`是直接退出该函数的，所以会导致其`inFlightFences[currentFrrame]`不会被unsight。所以我们要修改一下顺序，不重新生成`swapchain`即能正常推命令时，才调用`vkResetFences()`。
```c++
void HelloTriangleApplication::drawFrame()
{
    vkWaitForFences(device, 1, &inFlightFences[currentFrame], VK_TRUE, UINT64_MAX);
    uint32_t imageIndex;
    VkResult result = vkAcquireNextImageKHR(device, swapChain, UINT64_MAX, imageAvailableSemaphores[currentFrame], VK_NULL_HANDLE, &imageIndex);
    if (result == VK_ERROR_OUT_OF_DATE_KHR)
    {
        recreateSwapChain();
        return;
    }
    else if (result != VK_SUCCESS && result != VK_SUBOPTIMAL_KHR)
    {  
        throw std::runtime_error("failed to acquire swap chain image!");
    }
    vkResetFences(device, 1, &inFlightFences[currentFrame]);
	......
}
```
## Handling resizes exsplicitly
在 Vulkan 中，`VK_ERROR_OUT_OF_DATE_KHR` 错误表示交换链与窗口表面不再兼容，通常发生在窗口大小变化后。但该错误**不被保证一定触发**
1. **双重检测机制**：
    
    - 显式监听窗口系统事件（如 GLFW 的 `glfwSetFramebufferSizeCallback`）
        
    - 同时检查 Vulkan API 调用返回的错误码
```c++
class HelloTriangleApplication
{  
    private:
	    ......
        uint32_t currentFrame = 0;
        bool framebufferResized = false;
		......
        static void framebufferResizeCallback(GLFWwindow* window, int width, int height);
		......
};
void HelloTriangleApplication::framebufferResizeCallback(GLFWwindow* window, int width, int height)
{
    auto app = reinterpret_cast<HelloTriangleApplication*>(glfwGetWindowUserPointer(window));
    app->framebufferResized = true;
}

void HelloTriangleApplication::initWindow()
{
    glfwInit();
  
    glfwWindowHint(GLFW_CLIENT_API, GLFW_NO_API);
    glfwWindowHint(GLFW_RESIZABLE, GLFW_FALSE);
  
    window = glfwCreateWindow(WIDTH, HEIGHT, "Vulkan", nullptr, nullptr);
    glfwSetWindowUserPointer(window, this);
    glfwSetFramebufferSizeCallback(window, framebufferResizeCallback);
}

void HelloTriangleApplication::drawFrame()
{
	......
    result = vkQueuePresentKHR(presentQueue, &presentInfo);
    if (result == VK_ERROR_OUT_OF_DATE_KHR || result == VK_SUBOPTIMAL_KHR || framebufferResized)
    {
        framebufferResized = false;
        recreateSwapChain();
    }
    else if (result != VK_SUCCESS)
        throw std::runtime_error("failed to present swap chain image!");

    currentFrame = (currentFrame + 1) % MAX_FRAMES_IN_FLIGHT;
}
```
## Handling minimization
处理特殊情况，最小化。这时候我们直接在`void recreateSwapChain()`阻塞其运行。
```c++
void HelloTriangleApplication::recreateSwapChain()
{
    int width = 0, height = 0;
    glfwGetWindowSize(window, &width, &height);
    while (width == 0 || height == 0)
    {
        glfwGetFramebufferSize(window, &width, &height);
        glfwWaitEvents();
    }
  
    vkDeviceWaitIdle(device);

    cleanupSwapChain();
  
    createSwapChain();
    createImageViews();
    createFramebuffers();

}
```


# Vertex input description
接下来四个章节都是与`Vertex input`相关的。
## Vertex shader
首先修改我们的vertex shader,让他从接受hard code 到从外部接受输入的顶点。
```c++
#version 450
layout(location = 0) in vec2 inPosition;
layout(location = 1) in vec3 inColor;

layout(location = 0) out vec3 fragColor;

void main() {
    gl_Position = vec4(inPosition, 0.0, 1.0);
    fragColor = inColor;
}
```
## Vertex data
引入外部库glm来存储向量的数据结构。
```c++
#include "glm/glm.hpp"

struct Vertex
{
    glm::vec2 pos;
    glm::vec3 color;
};

const std::vector<Vertex> vertices = {
    { { 0.0f, -0.5f }, { 1.0f, 0.0f, 0.0f } },
    { { 0.5f, 0.5f }, { 0.0f, 1.0f, 0.0f } },
    { { -0.5f, 0.5f }, { 0.0f, 0.0f, 1.0f } }
};
```
## Binding descriptions
这时候应该为我们的数据创建一个`VkVertexInputBindingDescription`，定义顶点数据在内存中的整体组织结构。在结构体`Vertex`中添加函数`static VkVertexInputBindingDescription getBindingDescription()`
```c++
struct Vertex
{
	......
    static VkVertexInputBindingDescription getBindingDescription();    
};

VkVertexInputBindingDescription Vertex::getBindingDescription()
{
    VkVertexInputBindingDescription bindingDescription{};
    bindingDescription.binding = 0;
    bindingDescription.stride = sizeof(Vertex);
    bindingDescription.inputRate = VK_VERTEX_INPUT_RATE_VERTEX;

    return bindingDescription;
}
```
## Attribute descriptions
然后在定义一个函数`static std::array<VkVertexInputAttributeDescription, 2> getAttributeDescriptions()`来填充`VkVertexInputAttributeDescription`其定义单个顶点属性的具体细节。
```c++
struct Vertex 
{
	......
    static std::array<VkVertexInputAttributeDescription, 2> getAttributeDescriptions(); 
};

std::array<VkVertexInputAttributeDescription, 2> Vertex::getAttributeDescriptions()
{
    std::array<VkVertexInputAttributeDescription, 2> attributeDescriptions{};

    attributeDescriptions[0].binding = 0;
    attributeDescriptions[0].location = 0;
    attributeDescriptions[0].format = VK_FORMAT_R32G32_SFLOAT;
    attributeDescriptions[0].offset = offsetof(Vertex, pos);

    attributeDescriptions[1].binding = 0;
    attributeDescriptions[1].location = 1;
    attributeDescriptions[1].format = VK_FORMAT_R32G32B32_SFLOAT;
    attributeDescriptions[1].offset = offsetof(Vertex, color);

    return attributeDescriptions;
}
```
## Pipeline vertex input
这时候应该修改GraphicsPipeline使其接受我们的顶点数据分布。
修改函数`void createGraphicsPipeline()`中`vertexInputInfo`，如下
```c++
void HelloTriangleApplication::createGraphicsPipeline()
{
	......
    VkPipelineVertexInputStateCreateInfo vertexInputInfo{};
    vertexInputInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_VERTEX_INPUT_STATE_CREATE_INFO;

    auto bindingDescription = Vertex::getBindingDescription();
    auto attributeDescriptions = Vertex::getAttributeDescriptions();

    vertexInputInfo.vertexBindingDescriptionCount = 1; 
    vertexInputInfo.vertexAttributeDescriptionCount = static_cast<uint32_t>(attributeDescriptions.size());
    vertexInputInfo.pVertexBindingDescriptions = &bindingDescription;
    vertexInputInfo.pVertexAttributeDescriptions = attributeDescriptions.data();
	......
```
# Vertex buffer creation
## Buffer creation
声明成员变量`VkBuffer`并声明函数`void createVertexBuffer()`在其中填充结构体`VkBufferCreateInfo`用于配置`VkBuffer`，并在`void initVulkan()`中，`void createCommandBuffers`前调用，并在`void cleanup()`中释放。
```c++
class HelloTriangleApplication
{   
		......
        VkBuffer vertexBuffer;
		......
        //Vertex buffer creation
        void createVertexBuffer();
		......
};

void HelloTriangleApplication::createVertexBuffer()
{
    VkBufferCreateInfo bufferInfo{};
    bufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
    bufferInfo.size = sizeof(vertices[0]) * vertices.size();
    bufferInfo.usage = VK_BUFFER_USAGE_VERTEX_BUFFER_BIT;
    bufferInfo.sharingMode = VK_SHARING_MODE_EXCLUSIVE;

    if (vkCreateBuffer(device, &bufferInfo, nullptr, &vertexBuffer) != VK_SUCCESS)
        throw std::runtime_error("failed to create vertex buffer!");
}

void HelloTriangleApplication::initVulkan()
{
    ......
    createVertexBuffer();
    createCommandBuffers();
    ......
}

void HelloTriangleApplication::cleanup()
{
	......
    vkDestroyBuffer(device, vertexBuffer, nullptr);
	......
}
```
## Memory requirements
之后我们需要为其分配内容，第一步就是需要一个对象`VkMemoryRequirements`，添加相关代码在`void createVertexBuffer()`中。同时显卡可以提供不同类型的内存，所以我们添加一个辅助函数来帮忙查找我们需要的内存`uint32_t findMemoryType()`
```c++
class HelloTriangleApplication
{  
    private:
		......
        uint32_t findMemoryType(uint32_t typeFilter, VkMemoryPropertyFlags properties);
		......
};

void HelloTriangleApplication::createVertexBuffer()
{
	......
    VkMemoryRequirements memRequirements;
    vkGetBufferMemoryRequirements(device, vertexBuffer, &memRequirements);
}

uint32_t HelloTriangleApplication::findMemoryType(uint32_t typeFilter, VkMemoryPropertyFlags properties)
{
    VkPhysicalDeviceMemoryProperties memProperties;
    vkGetPhysicalDeviceMemoryProperties(physicalDevice, &memProperties);

    for (uint32_t i = 0; i < memProperties.memoryTypeCount; i++)
    {
        if ((typeFilter & (1 << i)) && (memProperties.memoryTypes[i].propertyFlags & properties) == properties)
            return i;
    }

    throw std::runtime_error("failed to find suitable memory type!");
}
```
## Memory allocation
声明成员变量`VkDeviceMemory`,并回到`void createVertexBuffer()`并分配内存了，修改如下。
```c++
class HelloTriangleApplication
{  
    private:
	    ......
        VkBuffer vertexBuffer;
        VkDeviceMemory vertexBufferMemory;
		......
};

void HelloTriangleApplication::createVertexBuffer()
{
    VkBufferCreateInfo bufferInfo{};
    bufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
    bufferInfo.size = sizeof(vertices[0]) * vertices.size();
    bufferInfo.usage = VK_BUFFER_USAGE_VERTEX_BUFFER_BIT;
    bufferInfo.sharingMode = VK_SHARING_MODE_EXCLUSIVE;

    if (vkCreateBuffer(device, &bufferInfo, nullptr, &vertexBuffer) != VK_SUCCESS)
        throw std::runtime_error("failed to create vertex buffer!");

    VkMemoryRequirements memRequirements;
    vkGetBufferMemoryRequirements(device, vertexBuffer, &memRequirements);

    VkMemoryAllocateInfo allocInfo{};
    allocInfo.sType = VK_STRUCTURE_TYPE_MEMORY_ALLOCATE_INFO;
    allocInfo.allocationSize = memRequirements.size;
    allocInfo.memoryTypeIndex = findMemoryType(memRequirements.memoryTypeBits, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT);

    if (vkAllocateMemory(device, &allocInfo, nullptr, &vertexBufferMemory) != VK_SUCCESS)
        throw std::runtime_error("failed to allocate vertex buffer memory!");
    vkBindBufferMemory(device, vertexBuffer, vertexBufferMemory, 0);
}

void HelloTriangleApplication::cleanup()
{
	......
    vkDestroyBuffer(device, vertexBuffer, nullptr);
    vkFreeMemory(device, vertexBufferMemory, nullptr);
	......
}
```
## Filling the vertex buffer
接下来该填充其vertex buffer了。
```c++
void HelloTriangleApplication::createVertexBuffer()
{
	......
    void* data;
    vkMapMemory(device, vertexBufferMemory, 0, bufferInfo.size, 0, &data);
    memcpy(data, vertices.data(), (size_t)bufferInfo.size);
    vkUnmapMemory(device, vertexBufferMemory);
}
```
## Binding the vertex buffer
最后一步，在录制命令的时候bindVertexBuffer。
```c++
void HelloTriangleApplication::recordCommandBuffer(VkCommandBuffer commandBuffer, uint32_t imageIndex)
{
	......
        VkBuffer vertexBuffers[] = { vertexBuffer };
        VkDeviceSize offsets[] = { 0 };
        vkCmdBindVertexBuffers(commandBuffer, 0, 1, vertexBuffers, offsets);

        vkCmdDraw(commandBuffer, static_cast<uint32_t>(vertices.size()), 1, 0, 0);
    ......
}
```
# Staging buffer
Staging Buffer（暂存缓冲区是一个核心概念，它解决了 GPU 内存架构中的一个关键问题：高效安全地在 CPU 和 GPU 之间传输数据。同时只有通过staging buffer才能间数据保存在`GPU`的显存上。上一章节实际保存在`CPU`的`RAM`上。
## Transfer queue
Transfer queue 是接受传输内存的命令的。这便是我们接下来的章节有关`Staging buffer`如何在`CPU`传输给`GPU`所使用的队列。The good news is that any queue family with `VK_QUEUE_GRAPHICS_BIT` or `VK_QUEUE_COMPUTE_BIT` capabilities already implicitly support `VK_QUEUE_TRANSFER_BIT` operations.也就是我们的graphics queue也可以实现。所以我们暂时还是使用graphics queue
## Abstracting buffer creation
首先把创建一个`buffer`的功能抽象出来一个函数`void createBuffer()`。并使用该函数修改`void createVertexBuffer()`的实现。
```c++
void HelloTriangleApplication::createBuffer(VkDeviceSize size, VkBufferUsageFlags usage, VkMemoryPropertyFlags properties, VkBuffer& buffer, VkDeviceMemory& bufferMemory)
{
    VkBufferCreateInfo bufferInfo{};
    bufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
    bufferInfo.size = size;
    bufferInfo.usage = usage;
    bufferInfo.sharingMode = VK_SHARING_MODE_EXCLUSIVE;

    if (vkCreateBuffer(device, &bufferInfo, nullptr, &buffer) != VK_SUCCESS)
        throw std::runtime_error("failed to create buffer!");

    VkMemoryRequirements memRequirements;
    vkGetBufferMemoryRequirements(device, buffer, &memRequirements);
    
    VkMemoryAllocateInfo allocInfo{};
    allocInfo.sType = VK_STRUCTURE_TYPE_MEMORY_ALLOCATE_INFO;
    allocInfo.allocationSize = memRequirements.size;
    allocInfo.memoryTypeIndex = findMemoryType(memRequirements.memoryTypeBits, properties);

    if (vkAllocateMemory(device, &allocInfo, nullptr, &bufferMemory) != VK_SUCCESS)
        throw std::runtime_error("failed to allocate buffer memory!");

    vkBindBufferMemory(device, buffer, bufferMemory, 0);
}

void HelloTriangleApplication::createVertexBuffer() 
{ 
	VkDeviceSize bufferSize = sizeof(vertices[0]) * vertices.size(); 
	createBuffer(bufferSize, VK_BUFFER_USAGE_VERTEX_BUFFER_BIT, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, vertexBuffer, vertexBufferMemory); 
	void* data; 
	vkMapMemory(device, vertexBufferMemory, 0, bufferSize, 0, &data); 
	memcpy(data, vertices.data(), (size_t) bufferSize); 
	vkUnmapMemory(device, vertexBufferMemory); 
}
```
## Using a staging buffer
这时候又要一个负责函数负责copy buffer，于是创建函数`void copyBuffer()`，并在`void createVertexBuffer()`中使用。
```c++
void HelloTriangleApplication::copyBuffer(VkBuffer srcBuffer, VkBuffer dstBuffer, VkDeviceSize size)
{
    VkCommandBufferAllocateInfo allocInfo{};
    allocInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_ALLOCATE_INFO;
    allocInfo.level = VK_COMMAND_BUFFER_LEVEL_PRIMARY;
    allocInfo.commandPool = commandPool;
    allocInfo.commandBufferCount = 1;

    VkCommandBuffer commandBuffer;
    vkAllocateCommandBuffers(device, &allocInfo, &commandBuffer);

    VkCommandBufferBeginInfo beginInfo{};
    beginInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_BEGIN_INFO;
    beginInfo.flags = VK_COMMAND_BUFFER_USAGE_ONE_TIME_SUBMIT_BIT;

    vkBeginCommandBuffer(commandBuffer, &beginInfo);

        VkBufferCopy copyRegion{};
        copyRegion.size = size;
        vkCmdCopyBuffer(commandBuffer, srcBuffer, dstBuffer, 1, &copyRegion);

    vkEndCommandBuffer(commandBuffer);

    VkSubmitInfo submitInfo{};
    submitInfo.sType = VK_STRUCTURE_TYPE_SUBMIT_INFO;
    submitInfo.commandBufferCount = 1;
    submitInfo.pCommandBuffers = &commandBuffer;

    vkQueueSubmit(graphicsQueue, 1, &submitInfo, VK_NULL_HANDLE);
    vkQueueWaitIdle(graphicsQueue);

    vkFreeCommandBuffers(device, commandPool, 1, &commandBuffer);
}

void HelloTriangleApplication::createVertexBuffer()
{
    VkDeviceSize bufferSize = sizeof(vertices[0]) * vertices.size();

    VkBuffer stagingBuffer;
    VkDeviceMemory stagingBufferMemory;
    createBuffer(bufferSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT| VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, stagingBuffer, stagingBufferMemory);

    void* data;
    vkMapMemory(device, stagingBufferMemory, 0, bufferSize, 0, &data);
    memcpy(data, vertices.data(), (size_t)bufferSize);
    vkUnmapMemory(device, stagingBufferMemory);

    createBuffer(bufferSize, VK_BUFFER_USAGE_TRANSFER_DST_BIT | VK_BUFFER_USAGE_VERTEX_BUFFER_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, vertexBuffer, vertexBufferMemory);

    copyBuffer(stagingBuffer, vertexBuffer, bufferSize);

    vkDestroyBuffer(device, stagingBuffer, nullptr);
    vkFreeMemory(device, stagingBufferMemory, nullptr);
}
```
# Index buffer
## Index buffer creation
这一章节我们将使用Index buffer，故此修改Vertices 以及初始化indices结构。
在声明类成员`VkBuffer`以及`VkDeviceMemory`供indices使用。
同时声明函数`void createIndexBuffer()`，并在`void initVulkan()`中，与`void createVertexBuffer()`一起使用。最后，在`void cleanup()`中销毁。
```c++
const std::vector<Vertex> vertices = {
    { { -0.5f, -0.5f }, { 1.0f, 0.0f, 0.0f } },
    { { 0.5f, -0.5f }, { 0.0f, 1.0f, 0.0f } },
    { { 0.5f, 0.5f }, { 0.0f, 0.0f, 1.0f } },
    { { -0.5f, 0.5f }, { 1.0f, 1.0f, 1.0f } }
};

const std::vector<uint16_t> indices = {
    0, 1, 2, 2, 3, 0
};

class HelloTriangleApplication
{   
    private:
		......
        VkBuffer vertexBuffer;
        VkDeviceMemory vertexBufferMemory;

        VkBuffer indexBuffer;
        VkDeviceMemory indexBufferMemory;
		......
		//Index buffer
		void createIndexBuffer();
};

void HelloTriangleApplication::createIndexBuffer()
{
    VkDeviceSize bufferSize = sizeof(indices[0]) * indices.size();

    VkBuffer stagingBuffer;
    VkDeviceMemory stagingBufferMemory;
    createBuffer(bufferSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, stagingBuffer, stagingBufferMemory);

    void* data;
    vkMapMemory(device, stagingBufferMemory, 0, bufferSize, 0, &data);
    memcpy(data, indices.data(), (size_t)bufferSize);
    vkUnmapMemory(device, stagingBufferMemory);

    createBuffer(bufferSize, VK_BUFFER_USAGE_TRANSFER_DST_BIT | VK_BUFFER_USAGE_INDEX_BUFFER_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, indexBuffer, indexBufferMemory);

    copyBuffer(stagingBuffer, indexBuffer, bufferSize);

    vkDestroyBuffer(device, stagingBuffer, nullptr);
    vkFreeMemory(device, stagingBufferMemory, nullptr);
}

void HelloTriangleApplication::initVulkan()
{
    ......
    createVertexBuffer();
    createIndexBuffer();
    ......
}

void HelloTriangleApplication::cleanup()
{
	......
    vkDestroyBuffer(device, indexBuffer, nullptr);
    vkFreeMemory(device, indexBufferMemory, nullptr);
	......
}
```
## Using an index buffer
在`void recordCommandBuffer()`中使用`index buffer`。
```c++
void HelloTriangleApplication::recordCommandBuffer(VkCommandBuffer commandBuffer, uint32_t imageIndex)
{
    ......
        VkBuffer vertexBuffers[] = { vertexBuffer };
        VkDeviceSize offsets[] = { 0 };
        vkCmdBindVertexBuffers(commandBuffer, 0, 1, vertexBuffers, offsets);
        vkCmdBindIndexBuffer(commandBuffer, indexBuffer, 0, VK_INDEX_TYPE_UINT16);

	    vkCmdDrawIndexed(commandBuffer, static_cast<uint32_t>(indices.size()), 1, 0, 0, 0);
    ......
}
```
# Uniform buffers
# Descriptor layout and buffer
## Introduction
这章主要负责公共变量的设置，即uniform buffer。
首先添加结构体`UniformBufferObject`。
```c++
struct UniformBufferObject
{
    glm::mat4 model;
    glm::mat4 view;
    glm::mat4 proj;
};
```
## Vertex shader
这时我们要修改我们的vertex shader用来接受其uniform buffer的内容。
```glsl
#version 450

layout(binding = 0) uniform UniformBufferObject{
    mat4 model;
    mat4 view;
    mat4 proj;
}ubo;

layout(location = 0) in vec2 inPosition;
layout(location = 1) in vec3 inColor;

layout(location = 0) out vec3 fragColor;

void main()
{
    gl_Position = ubo.proj * ubo.view * ubo.model * vec4(inPosition, 0.0, 1.0);
    fragColor = inColor;
}
```
## Descriptor set layout
我们要初始化`VkDescriptorSetLayout`即要填充结构体`VkDescriptorSetLayoutBinding`以及`VkDescriptorSetLayoutCreateInfo`从而配置uniform buffer。
声明函数`void createDescriptorSetLayout()`并在`void initVulkan()`中`void createGraphicsPipeline()`前调用，在`void cleanup()`中清除。最后在`void createGraphicsPipeline()`中`VkPipelineLayoutCreateInfo`中传递其结构。
```c++
class HelloTriangleApplication
{   
    private:
        ......
        VkDescriptorSetLayout descriptorSetLayout;
        VkPipelineLayout pipelineLayout;
		......
        //uniform buffer
        void createDescriptorSetLayout();
		......
};

void HelloTriangleApplication::createDescriptorSetLayout()
{
    VkDescriptorSetLayoutBinding uboLayoutBinding{};
    uboLayoutBinding.binding = 0;
    uboLayoutBinding.descriptorCount = 1;
    uboLayoutBinding.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
    uboLayoutBinding.pImmutableSamplers = nullptr;
    uboLayoutBinding.stageFlags = VK_SHADER_STAGE_VERTEX_BIT;

    VkDescriptorSetLayoutCreateInfo layoutInfo{};
    layoutInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_LAYOUT_CREATE_INFO;
    layoutInfo.bindingCount = 1;
    layoutInfo.pBindings = &uboLayoutBinding;

    if(vkCreateDescriptorSetLayout(device, &layoutInfo, nullptr, &descriptorSetLayout) != VK_SUCCESS)
        throw std::runtime_error("failed to create descriptor set layout!");
}

void HelloTriangleApplication::createGraphicsPipeline()
{
	......
    VkPipelineLayoutCreateInfo pipelineLayoutInfo{};
    pipelineLayoutInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_LAYOUT_CREATE_INFO;
    pipelineLayoutInfo.setLayoutCount = 1;
    pipelineLayoutInfo.pSetLayouts = &descriptorSetLayout;
    pipelineLayoutInfo.pushConstantRangeCount = 0;

    if (vkCreatePipelineLayout(device, &pipelineLayoutInfo, nullptr, &pipelineLayout) != VK_SUCCESS)
        throw std::runtime_error("failed to create pipeline layout!");
	......
}
```
## Uniform buffer
同时uniform buffer 也属于buffer，所以我们还是需要通过`VkBuffer`和`VkDeviceMemory`来配置。
声明函数`void createUniformBuffers()`
```c++
class HelloTriangleApplication
{   
    private:
		......
        std::vector<VkBuffer> uniformBuffers;
        std::vector<VkDeviceMemory> uniformBuffersMemory;
        std::vector<void*> uniformBufferMapped;
		......
        //uniform buffer
        void createDescriptorSetLayout();
        void createUniformBuffers();
		......
};

void HelloTriangleApplication::createUniformBuffers()
{
    VkDeviceSize bufferSize = sizeof(UniformBufferObject);

    uniformBuffers.resize(MAX_FRAMES_IN_FLIGHT);
    uniformBuffersMemory.resize(MAX_FRAMES_IN_FLIGHT);
    uniformBuffersMemory.resize(MAX_FRAMES_IN_FLIGHT);

    for (size_t i = 0; i < MAX_FRAMES_IN_FLIGHT; i++)
    {
        createBuffer(bufferSize, VK_BUFFER_USAGE_UNIFORM_BUFFER_BIT, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, uniformBuffers[i], uniformBuffersMemory[i]);

        vkMapMemory(device, uniformBuffersMemory[i], 0, bufferSize, 0, &uniformBufferMapped[i]);
    }
}

void HelloTriangleApplication::initVulkan()
{
	......
    createVertexBuffer();
    createIndexBuffer();
    createUniformBuffers();
	......
}

void HelloTriangleApplication::cleanup()
{
	......
    for(size_t i = 0; i < MAX_FRAMES_IN_FLIGHT; i++)
    {
        vkDestroyBuffer(device, uniformBuffers[i], nullptr);
        vkFreeMemory(device, uniformBuffersMemory[i], nullptr);
    }
	......
}
```
## Updating uniform data
最后就是填充其uniform buffer里面的数据了。我们需要一个函数来进行update于是声明函数`void updateUniformBuffer()`。
```c++
class HelloTriangleApplication
{  
    private:
		......
        //uniform buffer
        void createDescriptorSetLayout();
        void createUniformBuffers();
        void updataUniformBuffer(uint32_t currentImage);
		......
};

void HelloTriangleApplication::updataUniformBuffer(uint32_t currentImage)
{
    static auto starTime = std::chrono::high_resolution_clock::now();

    auto currentTime = std::chrono::high_resolution_clock::now();
    float time = std::chrono::duration<float, std::chrono::seconds::period>(currentTime - starTime).count();

    UniformBufferObject ubo{};
    ubo.model = glm::rotate(glm::mat4(1.0f), time * glm::radians(90.0f), glm::vec3(0.0f, 0.0f, 1.0f));
    ubo.view = glm::lookAt(glm::vec3(2.0f, 2.0f, 2.0f), glm::vec3(0.0f, 0.0f, 0.0f), glm::vec3(0.0f, 0.0f, 1.0f));
    ubo.proj = glm::perspective(glm::radians(45.0f), swapChainExtent.width / (float)swapChainExtent.height, 0.1f, 10.0f);
    ubo.proj[1][1] *= -1;

    memcpy(uniformBufferMapped[currentFrame], &ubo, sizeof(ubo));
}
```
# Descriptor pool and sets
这一章节我们要创建`Descriptor sets`。
## Descriptor pool
在创建`Descriptor sets`之前我们需要`DescriptorPool`。声明成员变量`VkDescriptorPool`创建函数`void createDescriptorPool()`，并在`void initVulkan()`中调用。
```c++
class HelloTriangleApplication
{  
    private:
		......
        VkDescriptorPool descriptorPool;
		......
        //Descriptor pool and sets
        void createDescriptorPool();
		......
};

void HelloTriangleApplication::createDescriptorPool()
{
    VkDescriptorPoolSize poolSize{};
    poolSize.type = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
    poolSize.descriptorCount = static_cast<uint32_t>(MAX_FRAMES_IN_FLIGHT);

    VkDescriptorPoolCreateInfo poolInfo{};
    poolInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_POOL_CREATE_INFO;
    poolInfo.poolSizeCount = 1;
    poolInfo.pPoolSizes = &poolSize;
    poolInfo.maxSets = static_cast<uint32_t>(MAX_FRAMES_IN_FLIGHT);

    if (vkCreateDescriptorPool(device, &poolInfo, nullptr, &descriptorPool) != VK_SUCCESS)
        throw std::runtime_error("failed to create descriptor pool!");
}

void HelloTriangleApplication::initVulkan()
{
    ......
    createDescriptorPool();
	......
}
```
## Descriptor set
这时候便可以创建Descriptor set了，声明函数`void createDescriptorSets()`，并在`void initVulkan()`中调用。
```c++
class HelloTriangleApplication
{ 
    private:
		......
        VkDescriptorPool descriptorPool;
        std::vector<VkDescriptorSet> descriptorSets;
		......
        void createDescriptorSets();
		......
};

void HelloTriangleApplication::createDescriptorSets()
{
    std::vector<VkDescriptorSetLayout> layouts(MAX_FRAMES_IN_FLIGHT);
    VkDescriptorSetAllocateInfo allocInfo{};
    allocInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_ALLOCATE_INFO;
    allocInfo.descriptorPool = descriptorPool;
    allocInfo.descriptorSetCount = static_cast<uint32_t>(MAX_FRAMES_IN_FLIGHT);
    allocInfo.pSetLayouts = layouts.data();

    descriptorSets.resize(MAX_FRAMES_IN_FLIGHT);
    if (vkAllocateDescriptorSets(device, &allocInfo, descriptorSets.data()) != VK_SUCCESS)
        throw std::runtime_error("failed to allocate descriptor sets!");
    
    for (size_t i = 0; i < MAX_FRAMES_IN_FLIGHT; i++)
    {
        VkDescriptorBufferInfo bufferInfo{};
        bufferInfo.buffer = uniformBuffers[i];
        bufferInfo.offset = 0;
        bufferInfo.range = sizeof(UniformBufferObject);

        VkWriteDescriptorSet descriptorWrite{};
        descriptorWrite.sType = VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET;
        descriptorWrite.dstSet = descriptorSets[i];
        descriptorWrite.dstBinding = 0;
        descriptorWrite.dstArrayElement = 0;
        descriptorWrite.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
        descriptorWrite.descriptorCount = 1;
        descriptorWrite.pBufferInfo = &bufferInfo;

        vkUpdateDescriptorSets(device, 1, &descriptorWrite, 0, nullptr);
    }
}

void HelloTriangleApplication::initVulkan()
{
	......
    createDescriptorSets();
	......
}
```
## Using descriptor sets
最后在`void recordCommandBuffer()`中使用
```c++
void HelloTriangleApplication::recordCommandBuffer(VkCommandBuffer commandBuffer, uint32_t imageIndex)
{
		......
        vkCmdBindDescriptorSets(commandBuffer, VK_PIPELINE_BIND_POINT_GRAPHICS, pipelineLayout, 0, 1, &descriptorSets[currentFrame], 0, nullptr);
        ......
}
```
# Images
## Introduction
在前面，我们以及开始使用了Image这个对象了，但是确是swapchain自动构建的。这回我们要自己手动构建一个Image并存储texture。
## Image library
我们需要这个头文件`stb_image.h`，来处理我们texture的数据转换，例如从.jpg图像转换到我们数组里面保存。
## Loading an image
首先引用此头文件。声明函数`void createTextureImage()`，并在`void initVulkan()`中调用。(记得提前保存好所需的jpg文件并获取到其路径)
```c++
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image/stb_image.h"
class HelloTriangleApplication
{   
    private:
		......
        //Textrue mapping
        void createTextureImage();
		......
};

void HelloTriangleApplication::createTextureImage()
{
    int texWidth, texHeight, texChannels;
    std::string basePath(GetExecutionPath());
    stbi_uc* pixels = stbi_load((basePath + "/assets/textures/texture.jpg").c_str(), &texWidth, &texHeight, &texChannels, STBI_rgb_alpha);
    VkDeviceSize imageSize = texWidth * texHeight * 4;

    if (!pixels)
        throw std::runtime_error("failed to load texture image!");
}

void HelloTriangleApplication::initVulkan()
{
	......
    createCommandPool();
    createTextureImage();
    createVertexBuffer();
	......
}
```
## Staging buffer
同时作为`GPU`需要频繁使用的数据，我们需要将其传输到`GPU`上面。也就需要我们的`staging buffer`，最后还要将stb的数据释放。
```c++
void HelloTriangleApplication::createTextureImage()
{
    int texWidth, texHeight, texChannels;
    std::string basePath(GetExecutionPath());
    stbi_uc* pixels = stbi_load((basePath + "/assets/textures/texture.jpg").c_str(), &texWidth, &texHeight, &texChannels, STBI_rgb_alpha);
    VkDeviceSize imageSize = texWidth * texHeight * 4;

    if (!pixels)
        throw std::runtime_error("failed to load texture image!");

    VkBuffer stagingBuffer;
    VkDeviceMemory stagingBufferMemory;
    createBuffer(imageSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, stagingBuffer, stagingBufferMemory);

    void* data;
    vkMapMemory(device, stagingBufferMemory, 0, imageSize, 0, &data);
    memcpy(data, pixels, static_cast<size_t>(imageSize));
    vkUnmapMemory(device, stagingBufferMemory);

    stbi_image_free(pixels);
}
```
## Texture Image
首先新声明函数`void createImage()`以方便其他模块调用。
这会我们在类里面声明`VkImage`，`VkDeviceMemory`来供我们使用。随后填充结构体`VkImageCreateInfo`类配置我们的Texture Image。
```c++
class HelloTriangleApplication
{  
    private:
		......
        VkImage textureImage;
        VkDeviceMemory textureImageMemory;
	    ......
	    void createImage(uint32_t width, uint32_t height, VkFormat format, VkImageTiling tiling, VkImageUsageFlags usage, VkMemoryPropertyFlags properties, VkImage& image, VkDeviceMemory& imageMemory);
};

void HelloTriangleApplication::createImage(uint32_t width, uint32_t height, VkFormat format, VkImageTiling tiling, VkImageUsageFlags usage, VkMemoryPropertyFlags properties, VkImage& image, VkDeviceMemory& imageMemory)
{
    VkImageCreateInfo imageInfo{};
    imageInfo.sType = VK_STRUCTURE_TYPE_IMAGE_CREATE_INFO;
    imageInfo.imageType = VK_IMAGE_TYPE_2D;
    imageInfo.extent.width = width;
    imageInfo.extent.height = height;
    imageInfo.extent.depth = 1;
    imageInfo.mipLevels = 1;
    imageInfo.arrayLayers = 1;
    imageInfo.format = format;
    imageInfo.tiling = tiling;
    imageInfo.initialLayout = VK_IMAGE_LAYOUT_UNDEFINED;
    imageInfo.usage = usage;
    imageInfo.samples = VK_SAMPLE_COUNT_1_BIT;
    imageInfo.sharingMode = VK_SHARING_MODE_EXCLUSIVE;

    if(vkCreateImage(device, &imageInfo, nullptr, &image) != VK_SUCCESS)
        throw std::runtime_error("failed to create image!");

    VkMemoryRequirements memRequirements;
    vkGetImageMemoryRequirements(device, image, &memRequirements);

    VkMemoryAllocateInfo allocInfo{};
    allocInfo.sType = VK_STRUCTURE_TYPE_MEMORY_ALLOCATE_INFO;
    allocInfo.allocationSize = memRequirements.size;
    allocInfo.memoryTypeIndex = findMemoryType(memRequirements.memoryTypeBits, properties);

    if (vkAllocateMemory(device, &allocInfo, nullptr, &imageMemory) != VK_SUCCESS)
        throw std::runtime_error("failed to allocate image memory!");
    
    vkBindImageMemory(device, image, imageMemory, 0);
}

void HelloTriangleApplication::createTextureImage()
{
	......
    createImage(texWidth, texHeight, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_TILING_OPTIMAL, VK_IMAGE_USAGE_TRANSFER_DST_BIT | VK_IMAGE_USAGE_SAMPLED_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, textureImage, textureImageMemory);
}
```
## Layout transitions
在完成主要任务之前，我们先给录制命令的开始和末尾两段代码进行封装，以便以后使用。同时可以在函数`void copyBuffer()`中使用。
```c++
class HelloTriangleApplication
{  
    private:
		......
        VkCommandBuffer beginSingleTimeCommands();
        void endSingleTimeCommands(VkCommandBuffer commandBuffer);
		......
};

VkCommandBuffer HelloTriangleApplication::beginSingleTimeCommands()
{
    VkCommandBufferAllocateInfo allocInfo{};
    allocInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_ALLOCATE_INFO;
    allocInfo.level = VK_COMMAND_BUFFER_LEVEL_PRIMARY;
    allocInfo.commandPool = commandPool;
    allocInfo.commandBufferCount = 1;

    VkCommandBuffer commandBuffer;
    vkAllocateCommandBuffers(device, nullptr, &commandBuffer);

    VkCommandBufferBeginInfo beginInfo{};
    beginInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_BEGIN_INFO;
    beginInfo.flags = VK_COMMAND_BUFFER_USAGE_ONE_TIME_SUBMIT_BIT;

    vkBeginCommandBuffer(commandBuffer, &beginInfo);

    return commandBuffer;
}

void HelloTriangleApplication::endSingleTimeCommands(VkCommandBuffer commandBuffer)
{
    vkEndCommandBuffer(commandBuffer);

    VkSubmitInfo submitInfo{};
    submitInfo.sType = VK_STRUCTURE_TYPE_SUBMIT_INFO;
    submitInfo.commandBufferCount = 1;
    submitInfo.pCommandBuffers = &commandBuffer;

    vkQueueSubmit(graphicsQueue, 1, &submitInfo, VK_NULL_HANDLE);
    vkQueueWaitIdle(graphicsQueue);

    vkFreeCommandBuffers(device, commandPool, 1, &commandBuffer);
}

void HelloTriangleApplication::copyBuffer(VkBuffer srcBuffer, VkBuffer dstBuffer, VkDeviceSize size)
{
    VkCommandBuffer commandBuffer = beginSingleTimeCommands(); 

        VkBufferCopy copyRegion{};
        copyRegion.size = size;
        vkCmdCopyBuffer(commandBuffer, srcBuffer, dstBuffer, 1, &copyRegion);

    endSingleTimeCommands(commandBuffer);
}
```
在我们使用image之前，因为不同图像布局对应不同的硬件内存访问模式，为了获取更好的性能，我们需要转换其layout以完成各种各样的操作。同时为了避免数据冲突，我们还需要使用屏障`Barrier`，避免读写的顺序问题。
首先创建一个辅助函数，帮助我们进行不同数据格式的转换。`void transitionImageLayout()`。
```c++
class HelloTriangleApplication
{   
    private:
        ......
        void transitionImageLayout(VkImage image, VkFormat format, VkImageLayout oldLayout, VkImageLayout newLayout);
        ......
};

void HelloTriangleApplication::transitionImageLayout(VkImage image, VkFormat format, VkImageLayout oldLayout, VkImageLayout newLayout)
{
    VkCommandBuffer commandBuffer = beginSingleTimeCommands();

    VkImageMemoryBarrier barrier{};
    barrier.sType = VK_STRUCTURE_TYPE_IMAGE_MEMORY_BARRIER;
    barrier.oldLayout = oldLayout;
    barrier.newLayout = newLayout;
    barrier.srcQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
    barrier.dstQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
    barrier.image = image;
    barrier.subresourceRange.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
    barrier.subresourceRange.baseMipLevel = 0;
    barrier.subresourceRange.levelCount = 1;
    barrier.subresourceRange.baseArrayLayer = 0;
    barrier.subresourceRange.layerCount = 1;

    VkPipelineStageFlags sourceStage;
    VkPipelineStageFlags destinationStage;

    if (oldLayout == VK_IMAGE_LAYOUT_UNDEFINED && newLayout == VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL)
    {
        barrier.srcAccessMask = 0;
        barrier.dstAccessMask = VK_ACCESS_TRANSFER_WRITE_BIT;

        sourceStage = VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT;
        destinationStage = VK_PIPELINE_STAGE_TRANSFER_BIT;
    }
    else if (oldLayout == VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL && newLayout == VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL)
    {
        barrier.srcAccessMask = VK_ACCESS_TRANSFER_WRITE_BIT;
        barrier.dstAccessMask = VK_ACCESS_SHADER_READ_BIT;

        sourceStage = VK_PIPELINE_STAGE_TRANSFER_BIT;
        destinationStage = VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT;
    }
    else
    {
        throw std::invalid_argument("unsupported layout transition!");
    }

    vkCmdPipelineBarrier(
        commandBuffer,
        sourceStage, destinationStage,
        0,
        0, nullptr,
        0, nullptr,
        1, &barrier
    );

    endSingleTimeCommands(commandBuffer);
}
```
## Copying buffer to image
随后我们专门写一个函数`void copyBufferToImage()`来完成将数组的buffer存储进image里面。
```c++
class HelloTriangleApplication
{  
    private:
		......
        void copyBufferToImage(VkBuffer buffer, VkImage image, uint32_t width, uint32_t height);
		......
};

void HelloTriangleApplication::copyBufferToImage(VkBuffer buffer, VkImage image, uint32_t width, uint32_t height)
{
    VkCommandBuffer commandBuffer = beginSingleTimeCommands();

    VkBufferImageCopy region{};
    region.bufferOffset = 0;
    region.bufferRowLength = 0;
    region.bufferImageHeight = 0;
    region.imageSubresource.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
    region.imageSubresource.mipLevel = 0;
    region.imageSubresource.baseArrayLayer = 0;
    region.imageSubresource.layerCount = 1;
    region.imageOffset = { 0, 0, 0 };
    region.imageExtent = {
        width,
        height,
        1
    };
    
    vkCmdCopyBufferToImage(commandBuffer, buffer, image, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL, 1, &region);

    endSingleTimeCommands(commandBuffer);
}
```
## Preparing the texture image
这时候便可以回到`void createTextureImage()`函数里面完成接下来的内容了。
```c++
void HelloTriangleApplication::createTextureImage()
{
	......
    transitionImageLayout(textureImage, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_LAYOUT_UNDEFINED, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL);
    copyBufferToImage(stagingBuffer, textureImage, static_cast<uint32_t>(texWidth), static_cast<uint32_t>(texHeight));
    transitionImageLayout(textureImage, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL, VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL);

    vkDestroyBuffer(device, stagingBuffer, nullptr);
    vkFreeMemory(device, stagingBufferMemory, nullptr);
}
```
## Cleanup
最后别忘了在`void cleanup()`中销毁image以及其内存。
```c++
void HelloTriangleApplication::cleanup()
{
	......
    vkDestroyImage(device, textureImage, nullptr);
    vkFreeMemory(device, textureImageMemory, nullptr);
	......
}
```
# Image view and sampler
## Texture image view
这时候我们需要为我们创建的image来创建其image view了。声明成员`VkImageView`，函数`void createTextureImageView()`，以及辅助函数`void createImage()`。`void createTextureImageView()`需要在`void initVulkan()`中调用，而辅助函数可以在`void createImageViews()`（给swap chain中的image创建image view的函数中使用）。并在`void cleanup()`中销毁`VkImageView`
```c++
class HelloTriangleApplication
{  
    private:
		......
        VkImageView textureImageView;
		......
        //Image view and sampler
        void createTextureImageView();
        VkImageView createImageView(VkImage image, VkFormat format);
		......
};

void HelloTriangleApplication::createTextureImageView()
{
    textureImageView = createImageView(textureImage, VK_FORMAT_R8G8B8A8_SRGB);
}

VkImageView HelloTriangleApplication::createImageView(VkImage image, VkFormat format)
{
    VkImageViewCreateInfo viewInfo{};
    viewInfo.sType = VK_STRUCTURE_TYPE_IMAGE_VIEW_CREATE_INFO;
    viewInfo.image = image;
    viewInfo.viewType = VK_IMAGE_VIEW_TYPE_2D;
    viewInfo.format = format;
    viewInfo.subresourceRange.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
    viewInfo.subresourceRange.baseMipLevel = 0;
    viewInfo.subresourceRange.levelCount = 1;
    viewInfo.subresourceRange.baseArrayLayer = 0;
    viewInfo.subresourceRange.layerCount = 1;
    
    VkImageView imageView;
    if (vkCreateImageView(device, &viewInfo, nullptr, &imageView) != VK_SUCCESS)
        throw std::runtime_error("failed to create texture image view!");

    return imageView;
}

void HelloTriangleApplication::createImageViews()
{
    swapChainImageViews.resize(swapChainImages.size());

    for (size_t i = 0; i < swapChainImages.size(); i++)
        swapChainImageViews[i] = createImageView(swapChainImages[i], swapChainImageFormat);
}

void HelloTriangleApplication::initVulkan()
{
	......
    createTextureImage();
    createTextureImageView();
	......
}
void HelloTriangleApplication::cleanup()
{
	......
    vkDestroyImageView(device, textureImageView, nullptr);
    vkDestroyImage(device, textureImage, nullptr);
    vkFreeMemory(device, textureImageMemory, nullptr);
	......
}
```
## Samplers
我们还需要为texture创建其Samplers。故声明成员变量`VkSampler`，以及函数`void createTextureSample()`。并在`void initVulkan()`中调用，在`void cleanup()`中销毁。
```c++
class HelloTriangleApplication
{  
    private:
		......
        VkSampler textureSampler;
		......
        //Image view and sampler
        void createTextureSampler();
		......
};

void HelloTriangleApplication::createTextureSampler()
{
    VkPhysicalDeviceProperties properties{};
    vkGetPhysicalDeviceProperties(physicalDevice, &properties);

    VkSamplerCreateInfo samplerInfo{};
    samplerInfo.sType = VK_STRUCTURE_TYPE_SAMPLER_CREATE_INFO;
    samplerInfo.magFilter = VK_FILTER_LINEAR;
    samplerInfo.minFilter = VK_FILTER_LINEAR;
    samplerInfo.addressModeU = VK_SAMPLER_ADDRESS_MODE_REPEAT;
    samplerInfo.addressModeV = VK_SAMPLER_ADDRESS_MODE_REPEAT;
    samplerInfo.addressModeW = VK_SAMPLER_ADDRESS_MODE_REPEAT;
    samplerInfo.anisotropyEnable = VK_TRUE;
    samplerInfo.maxAnisotropy = properties.limits.maxSamplerAnisotropy;
    samplerInfo.borderColor = VK_BORDER_COLOR_INT_OPAQUE_BLACK;
    samplerInfo.unnormalizedCoordinates = VK_FALSE;
    samplerInfo.compareEnable = VK_FALSE;
    samplerInfo.compareOp = VK_COMPARE_OP_ALWAYS;
    samplerInfo.mipmapMode = VK_SAMPLER_MIPMAP_MODE_LINEAR;

    if (vkCreateSampler(device, &samplerInfo, nullptr, &textureSampler) != VK_SUCCESS)
        throw std::runtime_error("failed to create trexture sampler!");
}

void HelloTriangleApplication::initVulkan()
{
	......
    createTextureImage();
    createTextureImageView();
    createTextureSampler();
	......
}

void HelloTriangleApplication::cleanup()
{
	......
    vkDestroySampler(device, textureSampler, nullptr);
    vkDestroyImageView(device, textureImageView, nullptr);
    vkDestroyImage(device, textureImage, nullptr);
    vkFreeMemory(device, textureImageMemory, nullptr);
	......
}
```
## Anisotropy device feature
在最后我们需要在`void createLogicalDevice()`中开启`anisotropic filtering`。同时在`bool isDeviceSuitable()`中要求我们的device需要支持该feature。
```c++
void HelloTriangleApplication::createLogicalDevice()
{
    QueueFamilyIndices indices = findQueuFamilies(physicalDevice);

    std::vector<VkDeviceQueueCreateInfo> queueCreateInfos;
    std::set<uint32_t> uniqueQueueFamilies = { indices.graphicsFamily.value(), indices.presentFamily.value() };

    float queuePriority = 1.0f;
    for (uint32_t queueFamily : uniqueQueueFamilies)
    {
        VkDeviceQueueCreateInfo queueCreateInfo{};
        queueCreateInfo.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
        queueCreateInfo.queueFamilyIndex = queueFamily;
        queueCreateInfo.queueCount = 1;
        queueCreateInfo.pQueuePriorities = &queuePriority;
        queueCreateInfos.push_back(queueCreateInfo);
    }
	// new add
    VkPhysicalDeviceFeatures deviceFeatures{};
    deviceFeatures.samplerAnisotropy = VK_TRUE;
    
    VkDeviceCreateInfo createInfo{};
    createInfo.sType = VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO;

    createInfo.queueCreateInfoCount = static_cast<uint32_t>(queueCreateInfos.size());
    createInfo.pQueueCreateInfos = queueCreateInfos.data();

    createInfo.pEnabledFeatures = &deviceFeatures;
    
    createInfo.enabledExtensionCount = static_cast<uint32_t>(deviceExtensions.size());
    createInfo.ppEnabledExtensionNames = deviceExtensions.data();

    if (enableValidationLayers)
    {
        createInfo.enabledLayerCount = static_cast<uint32_t>(validationLayers.size());
        createInfo.ppEnabledLayerNames = validationLayers.data();
    }
    else 
        createInfo.enabledLayerCount = 0;

    if (vkCreateDevice(physicalDevice, &createInfo, nullptr, &device) != VK_SUCCESS)
        throw std::runtime_error("failed to create logical device!");

    vkGetDeviceQueue(device, indices.graphicsFamily.value(), 0, &graphicsQueue);
    vkGetDeviceQueue(device, indices.presentFamily.value(), 0, &presentQueue);
}

bool HelloTriangleApplication::isDeviceSuitable(VkPhysicalDevice device)
{
    QueueFamilyIndices indices = findQueuFamilies(device);

    bool extensionsSupported = checkDeviceExtensionSupport(device);

    bool swapChainAdequate = false;
    if (extensionsSupported)
    {
        SwapChainSupportDetails swapChainSupport = querySwapChainSupport(device);
        swapChainAdequate = !swapChainSupport.formats.empty() && !swapChainSupport.presentModes.empty();
    }

	//new add
    VkPhysicalDeviceFeatures supportedFeatures;
    vkGetPhysicalDeviceFeatures(device, &supportedFeatures);

	//modify
    return indices.isComplete() && extensionsSupported && swapChainAdequate && supportedFeatures.samplerAnisotropy;
}
```
# Combined image sampler
## Updating the descriptors
首先在函数`void createDescriptorSetLayout()`更改descriptors告诉vulkan需要接受的资源。而后修改函数`void createDescriptorPool()`的poolSize修改其可以分配的数量和资源。最后在函数`void createDescriptorSets()`中配置对应的set，并在其`VkWriteDescriptor`中绑定对应的资源。
```c++
void HelloTriangleApplication::createDescriptorSetLayout()
{
    VkDescriptorSetLayoutBinding uboLayoutBinding{};
    uboLayoutBinding.binding = 0;
    uboLayoutBinding.descriptorCount = 1;
    uboLayoutBinding.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
    uboLayoutBinding.pImmutableSamplers = nullptr;
    uboLayoutBinding.stageFlags = VK_SHADER_STAGE_VERTEX_BIT;

    VkDescriptorSetLayoutBinding samplerLayoutBinding{};
    samplerLayoutBinding.binding = 1;
    samplerLayoutBinding.descriptorCount = 1;
    samplerLayoutBinding.descriptorType = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
    samplerLayoutBinding.pImmutableSamplers = nullptr;
    samplerLayoutBinding.stageFlags = VK_SHADER_STAGE_FRAGMENT_BIT;

    std::array<VkDescriptorSetLayoutBinding, 2> bindings = { uboLayoutBinding, samplerLayoutBinding };

    VkDescriptorSetLayoutCreateInfo layoutInfo{};
    layoutInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_LAYOUT_CREATE_INFO;
    layoutInfo.bindingCount = static_cast<uint32_t>(bindings.size());
    layoutInfo.pBindings = bindings.data();

    if(vkCreateDescriptorSetLayout(device, &layoutInfo, nullptr, &descriptorSetLayout) != VK_SUCCESS)
        throw std::runtime_error("failed to create descriptor set layout!");
}

void HelloTriangleApplication::createDescriptorPool()
{
    std::array<VkDescriptorPoolSize, 2> poolSizes{};
    poolSizes[0].type = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
    poolSizes[0].descriptorCount = static_cast<uint32_t>(MAX_FRAMES_IN_FLIGHT);
    poolSizes[1].type = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
    poolSizes[1].descriptorCount = static_cast<uint32_t>(MAX_FRAMES_IN_FLIGHT);

    VkDescriptorPoolCreateInfo poolInfo{};
    poolInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_POOL_CREATE_INFO;
    poolInfo.poolSizeCount = static_cast<uint32_t>(poolSizes.size());
    poolInfo.pPoolSizes = poolSizes.data();
    poolInfo.maxSets = static_cast<uint32_t>(MAX_FRAMES_IN_FLIGHT);

    if (vkCreateDescriptorPool(device, &poolInfo, nullptr, &descriptorPool) != VK_SUCCESS)
        throw std::runtime_error("failed to create descriptor pool!");
}

void HelloTriangleApplication::createDescriptorSets()
{
    std::vector<VkDescriptorSetLayout> layouts(MAX_FRAMES_IN_FLIGHT, descriptorSetLayout);
    VkDescriptorSetAllocateInfo allocInfo{};
    allocInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_ALLOCATE_INFO;
    allocInfo.descriptorPool = descriptorPool;
    allocInfo.descriptorSetCount = static_cast<uint32_t>(MAX_FRAMES_IN_FLIGHT);
    allocInfo.pSetLayouts = layouts.data();

    descriptorSets.resize(MAX_FRAMES_IN_FLIGHT);
    if (vkAllocateDescriptorSets(device, &allocInfo, descriptorSets.data()) != VK_SUCCESS)
        throw std::runtime_error("failed to allocate descriptor sets!");
    
    for (size_t i = 0; i < MAX_FRAMES_IN_FLIGHT; i++)
    {
        VkDescriptorBufferInfo bufferInfo{};
        bufferInfo.buffer = uniformBuffers[i];
        bufferInfo.offset = 0;
        bufferInfo.range = sizeof(UniformBufferObject);

        VkDescriptorImageInfo imageInfo{};
        imageInfo.imageLayout = VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL;
        imageInfo.imageView = textureImageView;
        imageInfo.sampler = textureSampler;
    
        std::array<VkWriteDescriptorSet, 2> descriptorWrites{};

        descriptorWrites[0].sType = VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET;
        descriptorWrites[0].dstSet = descriptorSets[i];
        descriptorWrites[0].dstBinding = 0;
        descriptorWrites[0].dstArrayElement = 0;
        descriptorWrites[0].descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
        descriptorWrites[0].descriptorCount = 1;
        descriptorWrites[0].pBufferInfo = &bufferInfo;

        descriptorWrites[1].sType = VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET;
        descriptorWrites[1].dstSet = descriptorSets[i];
        descriptorWrites[1].dstBinding = 1;
        descriptorWrites[1].dstArrayElement = 0;
        descriptorWrites[1].descriptorType = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
        descriptorWrites[1].descriptorCount = 1;
        descriptorWrites[1].pImageInfo = &imageInfo;

        vkUpdateDescriptorSets(device, static_cast<uint32_t>(descriptorWrites.size()), descriptorWrites.data(), 0, nullptr);
    }
}
```
## Texture coordinates
更新`Vertex`结构，包含贴图坐标。并修改其成员函数使其可以获取正确的`VkVertexInputBindingDescription`，以及`VkVertexInputAttributeDescription`。
```c++
const std::vector<Vertex> vertices = {
    { { -0.5f, -0.5f }, { 1.0f, 0.0f, 0.0f }, { 1.0f, 0.0f } },
    { { 0.5f, -0.5f }, { 0.0f, 1.0f, 0.0f }, { 0.0f, 0.0f } },
    { { 0.5f, 0.5f }, { 0.0f, 0.0f, 1.0f }, { 0.0f, 1.0f } },
    { { -0.5f, 0.5f }, { 1.0f, 1.0f, 1.0f }, { 1.0f, 1.0f } }
};

VkVertexInputBindingDescription Vertex::getBindingDescription()
{
    VkVertexInputBindingDescription bindingDescription{};
    bindingDescription.binding = 0;
    bindingDescription.stride = sizeof(Vertex);
    bindingDescription.inputRate = VK_VERTEX_INPUT_RATE_VERTEX;

    return bindingDescription;
}

std::array<VkVertexInputAttributeDescription, 3> Vertex::getAttributeDescriptions()
{
    std::array<VkVertexInputAttributeDescription, 3> attributeDescriptions{};

    attributeDescriptions[0].binding = 0;
    attributeDescriptions[0].location = 0;
    attributeDescriptions[0].format = VK_FORMAT_R32G32_SFLOAT;
    attributeDescriptions[0].offset = offsetof(Vertex, pos);

    attributeDescriptions[1].binding = 0;
    attributeDescriptions[1].location = 1;
    attributeDescriptions[1].format = VK_FORMAT_R32G32B32_SFLOAT;
    attributeDescriptions[1].offset = offsetof(Vertex, color);

    attributeDescriptions[2].binding = 0;
    attributeDescriptions[2].location = 2;
    attributeDescriptions[2].format = VK_FORMAT_R32G32_SFLOAT;
    attributeDescriptions[2].offset = offsetof(Vertex, texCoord); 

    return attributeDescriptions;
}
```
## Shaders
修改shaders使其使用对应的贴图。
vertex shader
```glsl
#version 450

layout(binding = 0) uniform UniformBufferObject{
    mat4 model;
    mat4 view;
    mat4 proj;
}ubo;

layout(location = 0) in vec2 inPosition;
layout(location = 1) in vec3 inColor;
layout(location = 2) in vec2 inTexCoord;

layout(location = 0) out vec3 fragColor;
layout(location = 1) out vec2 fragTexCoord;

void main()
{
    gl_Position = ubo.proj * ubo.view * ubo.model * vec4(inPosition, 0.0, 1.0);
    fragColor = inColor;
    fragTexCoord = inTexCoord;
}
```
fragment shader
```glsl
#version 450

layout (binding = 1)  uniform sampler2D texSampler;

layout (location = 0) in vec3 fragColor;
layout (location = 1) in vec2 fragTexCoord;

layout (location = 0) out vec4 outColor;

void main()
{
    // outColor = vec4(fragColor, 1.0);
    outColor = texture(texSampler, fragTexCoord);   
}
```
# Depth buffering
## 3D geometry
首先更新我们的`Vertex`结构体使得其接受三个坐标的顶点。并更新vertex shader
```c++
const std::vector<Vertex> vertices = {
    { { -0.5f, -0.5f, 0.0f }, { 1.0f, 0.0f, 0.0f }, { 1.0f, 0.0f } },
    { { 0.5f, -0.5f, 0.0f }, { 0.0f, 1.0f, 0.0f }, { 0.0f, 0.0f } },
    { { 0.5f, 0.5f, 0.0f }, { 0.0f, 0.0f, 1.0f }, { 0.0f, 1.0f } },
    { { -0.5f, 0.5f, 0.0f }, { 1.0f, 1.0f, 1.0f }, { 1.0f, 1.0f } },

    { { -0.5f, -0.5f, -0.5f }, { 1.0f, 0.0f, 0.0f }, { 1.0f, 0.0f } },
    { { 0.5f, -0.5f, -0.5f }, { 0.0f, 1.0f, 0.0f }, { 0.0f, 0.0f } },
    { { 0.5f, 0.5f, -0.5f }, { 0.0f, 0.0f, 1.0f }, { 0.0f, 1.0f } },
    { { -0.5f, 0.5f, -0.5f }, { 1.0f, 1.0f, 1.0f }, { 1.0f, 1.0f } }
};

const std::vector<uint16_t> indices = {
    0, 1, 2, 2, 3, 0,
    4, 5, 6, 6, 7, 4
};

std::array<VkVertexInputAttributeDescription, 3> Vertex::getAttributeDescriptions()
{
    std::array<VkVertexInputAttributeDescription, 3> attributeDescriptions{};

    attributeDescriptions[0].binding = 0;
    attributeDescriptions[0].location = 0;
    //modify
    attributeDescriptions[0].format = VK_FORMAT_R32G32B32_SFLOAT;
    attributeDescriptions[0].offset = offsetof(Vertex, pos);

    attributeDescriptions[1].binding = 0;
    attributeDescriptions[1].location = 1;
    attributeDescriptions[1].format = VK_FORMAT_R32G32B32_SFLOAT;
    attributeDescriptions[1].offset = offsetof(Vertex, color);

    attributeDescriptions[2].binding = 0;
    attributeDescriptions[2].location = 2;
    attributeDescriptions[2].format = VK_FORMAT_R32G32_SFLOAT;
    attributeDescriptions[2].offset = offsetof(Vertex, texCoord); 

    return attributeDescriptions;
}
```

```glsl
#version 450

layout(binding = 0) uniform UniformBufferObject{
    mat4 model;
    mat4 view;
    mat4 proj;
}ubo;

layout(location = 0) in vec3 inPosition;
layout(location = 1) in vec3 inColor;
layout(location = 2) in vec2 inTexCoord;

layout(location = 0) out vec3 fragColor;
layout(location = 1) out vec2 fragTexCoord;

void main()
{
    gl_Position = ubo.proj * ubo.view * ubo.model * vec4(inPosition, 1.0);
    fragColor = inColor;
    fragTexCoord = inTexCoord;
}
```
## Depth image and view
首先引入一个宏，使得其深度值映射从[-1.0, 1.0]，转变到[0.0, 1.0]。
```c++
#define GLM_FORCE_RADIANS
#define GLM_FORCE_DEPTH_ZERO_TO_ONE
```
我们需要在渲染管线引入深度测试，也就是我们需要创建一个image供其使用。即我们要创建结构`VkImage` `depthDeviceMemory` `VkImageView`。声明函数`void createDepthResources()`然后在`void initVulkan()`中调用，同时创建辅助函数`VkFormat findSupportedFormat()`，`VkFormat findDepthFormat()`，以及`bool hasStencilComponent()`，并修改`VkImage createImageView()`。
```c++
class HelloTriangleApplication
{  
    private:
	    ......
		VkImage depthImage;
        VkDeviceMemory depthImageMemory;
        VkImageView depthImgaeView;
		......
        //Depth buffering
        void createDepthResources();
        VkFormat findSupoortedFormat(const std::vector<VkFormat>& candidates, VkImageTiling tiling, VkFormatFeatureFlags features);
        VkFormat findDepthFormat();
        bool hasStencilComponent(VkFormat format);
		......
};

VkImageView HelloTriangleApplication::createImageView(VkImage image, VkFormat format, VkImageAspectFlags aspectFlags)
{
    VkImageViewCreateInfo viewInfo{};
    viewInfo.sType = VK_STRUCTURE_TYPE_IMAGE_VIEW_CREATE_INFO;
    viewInfo.image = image;
    viewInfo.viewType = VK_IMAGE_VIEW_TYPE_2D;
    viewInfo.format = format;
    viewInfo.subresourceRange.aspectMask = aspectFlags;
    viewInfo.subresourceRange.baseMipLevel = 0;
    viewInfo.subresourceRange.levelCount = 1;
    viewInfo.subresourceRange.baseArrayLayer = 0;
    viewInfo.subresourceRange.layerCount = 1;
    
    VkImageView imageView;
    if (vkCreateImageView(device, &viewInfo, nullptr, &imageView) != VK_SUCCESS)
        throw std::runtime_error("failed to create texture image view!");

    return imageView;
}

void HelloTriangleApplication::createDepthResources()
{
    VkFormat depthFormat = findDepthFormat();

    createImage(swapChainExtent.width, swapChainExtent.height, depthFormat, VK_IMAGE_TILING_OPTIMAL, VK_IMAGE_USAGE_DEPTH_STENCIL_ATTACHMENT_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, depthImage, depthImageMemory);

    depthImgaeView = createImageView(depthImage, depthFormat, VK_IMAGE_ASPECT_DEPTH_BIT);
}

VkFormat HelloTriangleApplication::findSupoortedFormat(const std::vector<VkFormat>& candidates, VkImageTiling tiling, VkFormatFeatureFlags features)
{
    for (VkFormat format : candidates)
    {
        VkFormatProperties props;
        vkGetPhysicalDeviceFormatProperties(physicalDevice, format, &props);

        if (tiling == VK_IMAGE_TILING_LINEAR && (props.linearTilingFeatures & features) == features)
            return format;
        else if (tiling == VK_IMAGE_TILING_OPTIMAL && (props.optimalTilingFeatures & features) == features)
            return format;
    }

    throw std::runtime_error("failed to find supported format!");
}

VkFormat HelloTriangleApplication::findDepthFormat()
{
    return findSupoortedFormat(
        { VK_FORMAT_D32_SFLOAT, VK_FORMAT_D32_SFLOAT_S8_UINT, VK_FORMAT_D24_UNORM_S8_UINT },
        VK_IMAGE_TILING_OPTIMAL,
        VK_FORMAT_FEATURE_DEPTH_STENCIL_ATTACHMENT_BIT
    );
}

void HelloTriangleApplication::initVulkan()
{
    ......
    createDepthResources();
    createFramebuffers();
    ......
}

//modify
swapChainImageViews[i] = createImageView(swapChainImages[i], swapChainImageFormat, VK_IMAGE_ASPECT_COLOR_BIT); 
depthImageView = createImageView(depthImage, depthFormat, VK_IMAGE_ASPECT_DEPTH_BIT);
textureImageView = createImageView(textureImage, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_ASPECT_COLOR_BIT);
```
### Explicitly transitioning the depth image
我们还可以在函数的最后转换其布局，到合适的布局，这要修改函数`void transitionImageLayout()`。
## Render pass
最后我们需要在`Render pass`附加上此附件。修改函数`void createRenderPass()`
```c++
void HelloTriangleApplication::createRenderPass()
{
    VkAttachmentDescription colorAttachment{};
    colorAttachment.format = swapChainImageFormat;
    colorAttachment.samples = VK_SAMPLE_COUNT_1_BIT;
    colorAttachment.loadOp = VK_ATTACHMENT_LOAD_OP_CLEAR;
    colorAttachment.storeOp = VK_ATTACHMENT_STORE_OP_STORE;
    colorAttachment.stencilLoadOp = VK_ATTACHMENT_LOAD_OP_DONT_CARE;
    colorAttachment.stencilStoreOp = VK_ATTACHMENT_STORE_OP_DONT_CARE;
    colorAttachment.initialLayout = VK_IMAGE_LAYOUT_UNDEFINED;
    colorAttachment.finalLayout = VK_IMAGE_LAYOUT_PRESENT_SRC_KHR;

    VkAttachmentDescription depthAttchment{};
    depthAttchment.format = findDepthFormat();
    depthAttchment.samples = VK_SAMPLE_COUNT_1_BIT;
    depthAttchment.loadOp = VK_ATTACHMENT_LOAD_OP_CLEAR;
    depthAttchment.storeOp = VK_ATTACHMENT_STORE_OP_DONT_CARE;
    depthAttchment.stencilLoadOp = VK_ATTACHMENT_LOAD_OP_DONT_CARE;
    depthAttchment.stencilStoreOp = VK_ATTACHMENT_STORE_OP_DONT_CARE;
    depthAttchment.initialLayout = VK_IMAGE_LAYOUT_UNDEFINED;
    depthAttchment.finalLayout = VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL;

    VkAttachmentReference colorAttachmentRef{};
    colorAttachmentRef.attachment = 0;
    colorAttachmentRef.layout = VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL;

    VkAttachmentReference depthAttchmentRef{};
    depthAttchmentRef.attachment = 1;
    depthAttchmentRef.layout = VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL;

    VkSubpassDescription subpass{};
    subpass.pipelineBindPoint = VK_PIPELINE_BIND_POINT_GRAPHICS;
    subpass.colorAttachmentCount = 1;
    subpass.pColorAttachments = &colorAttachmentRef;
    subpass.pDepthStencilAttachment = &depthAttchmentRef;

    VkSubpassDependency denpendency{};
    denpendency.srcSubpass = VK_SUBPASS_EXTERNAL;
    denpendency.dstSubpass = 0;
    denpendency.srcStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT | VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT;
    denpendency.srcAccessMask = 0;
    denpendency.dstStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT | VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT;
    denpendency.dstAccessMask = VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT | VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_WRITE_BIT;

    std::array<VkAttachmentDescription, 2> attachments = { colorAttachment, depthAttchment };

    VkRenderPassCreateInfo renderPassInfo{};
    renderPassInfo.sType = VK_STRUCTURE_TYPE_RENDER_PASS_CREATE_INFO;
    renderPassInfo.attachmentCount = static_cast<uint32_t>(attachments.size());
    renderPassInfo.pAttachments = attachments.data();
    renderPassInfo.subpassCount = 1; 
    renderPassInfo.pSubpasses = &subpass;
    renderPassInfo.dependencyCount = 1;
    renderPassInfo.pDependencies = &denpendency;

    if (vkCreateRenderPass(device, &renderPassInfo, nullptr, &renderPass) != VK_SUCCESS)
        throw std::runtime_error("failed to create render pass!");
}
```
## Framebuffer
这时候应该修改函数`void createFramebuffers()`使得其附带两个附件。
```c++
void HelloTriangleApplication::createFramebuffers()
{
    swapChainFramebuffers.resize(swapChainImageViews.size());

    for (size_t i = 0; i < swapChainImageViews.size(); i++)
    {
        std::array<VkImageView, 2> attachments = {
            swapChainImageViews[i],
            depthImageView
        };

        VkFramebufferCreateInfo framebufferInfo{};
        framebufferInfo.sType = VK_STRUCTURE_TYPE_FRAMEBUFFER_CREATE_INFO;
        framebufferInfo.renderPass = renderPass;
        framebufferInfo.attachmentCount = static_cast<uint32_t>(attachments.size());
        framebufferInfo.pAttachments = attachments.data();
        framebufferInfo.width = swapChainExtent.width;
        framebufferInfo.height = swapChainExtent.height;
        framebufferInfo.layers = 1;

        if (vkCreateFramebuffer(device, &framebufferInfo, nullptr, &swapChainFramebuffers[i]) != VK_SUCCESS)
            throw std::runtime_error("failed to create framebuffer!");
    }
}
```
## Clear values
现在去函数`void recordCommandBuffer()`，添加clear values，让其每次渲染都刷新一遍深度和模板的值。
```c++
void HelloTriangleApplication::recordCommandBuffer(VkCommandBuffer commandBuffer, uint32_t imageIndex)
{
	///modify
    std::array<VkClearValue, 2> clearValues{};
    clearValues[0].color = { { 0.0f, 0.0f, 0.0f, 1.0f } };
    clearValues[1].depthStencil = { 1.0f, 0 };

    renderPassInfo.clearValueCount = static_cast<uint32_t>(clearValues.size());
    renderPassInfo.pClearValues = clearValues.data();
	......
}
```
## Depth and stencil state
最后在`void createGraphicsPipeline()`中启用深度和模板测试。
```c++
void HelloTriangleApplication::createGraphicsPipeline()
{
	......
    VkPipelineDepthStencilStateCreateInfo depthStencil{};
    depthStencil.sType = VK_STRUCTURE_TYPE_PIPELINE_DEPTH_STENCIL_STATE_CREATE_INFO;
    depthStencil.depthTestEnable = VK_TRUE;
    depthStencil.depthWriteEnable = VK_TRUE;
    depthStencil.depthCompareOp = VK_COMPARE_OP_LESS;
    depthStencil.depthBoundsTestEnable = VK_FALSE;
    depthStencil.stencilTestEnable = VK_FALSE;
    ......
    VkGraphicsPipelineCreateInfo pipelineInfo{};
    pipelineInfo.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
    pipelineInfo.stageCount = 2;
    pipelineInfo.pStages = shaderStages;
    pipelineInfo.pVertexInputState = &vertexInputInfo;
    pipelineInfo.pInputAssemblyState = &inputAssembly;
    pipelineInfo.pViewportState = &viewportState;
    pipelineInfo.pRasterizationState = &rasterizer;
    pipelineInfo.pMultisampleState = &multisampling;
    pipelineInfo.pDepthStencilState = &depthStencil;
    pipelineInfo.pColorBlendState = &colorBlending;
    pipelineInfo.pDynamicState = &dynamicState;
    pipelineInfo.layout = pipelineLayout;
    pipelineInfo.renderPass = renderPass;
    pipelineInfo.subpass = 0;
    pipelineInfo.basePipelineHandle = VK_NULL_HANDLE;
    ......
}
```
## Handling window resize
当窗口大小发生改变的时候，depth buffer 附件的大小也应该发生改变，即应该在`void recreateSwapChain()`中改变其大小。
```c++
void HelloTriangleApplication::recreateSwapChain()
{
    int width = 0, height = 0;
    glfwGetWindowSize(window, &width, &height);
    while (width == 0 || height == 0)
    {
        glfwGetFramebufferSize(window, &width, &height);
        glfwWaitEvents();
    }

    vkDeviceWaitIdle(device);

    cleanupSwapChain();

    createSwapChain();
    createImageViews();
    createDepthResources();
    createFramebuffers();
}
```
在最后其清除操作也应该在`void cleanupSwapChain()`中
```c++
void HelloTriangleApplication::cleanupSwapChain()
{
    vkDestroyImageView(device, depthImageView, nullptr);
    vkDestroyImage(device, depthImage, nullptr);
    vkFreeMemory(device, depthImageMemory, nullptr);

    for (size_t i = 0; i < swapChainFramebuffers.size(); i++)
        vkDestroyFramebuffer(device, swapChainFramebuffers[i], nullptr);
    
    for (size_t i = 0; i < swapChainImageViews.size(); i++)
        vkDestroyImageView(device, swapChainImageViews[i], nullptr);
    
    vkDestroySwapchainKHR(device, swapChain, nullptr);
}
```
# Loading models
这章我们就开始加载别人的models了。
## Library
我们需要使用一个库名为`tinyobjloader`，同时我们只需要 `tiny_obj_loader.h`这个头文件即可。
## Sample mesh
保存对应的文件后
我们添加这些函数的路径。
```c++
const std::string ExecutionPath = GetExecutionPath();

const std::string MODEL_PATH = ExecutionPath + "/assets/model/viking_room.obj";
const std::string TEXTURE_PATH = ExecutionPath + "/assets/texture/viking_room.png";
```
同时修改函数`void createTextureImage()`中所加载的贴图。
```c++
void HelloTriangleApplication::createTextureImage()
{
    ......//modify
    stbi_uc* pixels = stbi_load(TEXTURE_PATH.c_str(), &texWidth, &texHeight, &texChannels, STBI_rgb_alpha);
	......
}
```
## Loading vertices and indices
这时候便可以将之前的`vertices`和`indices`修改，准备接受好model的数据了。并且修改函数`void recordCommandBuffer()`中录制的indices数据。
```c++
// const std::vector<Vertex> vertices = {
//     { { -0.5f, -0.5f, 0.0f }, { 1.0f, 0.0f, 0.0f }, { 1.0f, 0.0f } },
//     { { 0.5f, -0.5f, 0.0f }, { 0.0f, 1.0f, 0.0f }, { 0.0f, 0.0f } },
//     { { 0.5f, 0.5f, 0.0f }, { 0.0f, 0.0f, 1.0f }, { 0.0f, 1.0f } },
//     { { -0.5f, 0.5f, 0.0f }, { 1.0f, 1.0f, 1.0f }, { 1.0f, 1.0f } },

//     { { -0.5f, -0.5f, -0.5f }, { 1.0f, 0.0f, 0.0f }, { 1.0f, 0.0f } },
//     { { 0.5f, -0.5f, -0.5f }, { 0.0f, 1.0f, 0.0f }, { 0.0f, 0.0f } },
//     { { 0.5f, 0.5f, -0.5f }, { 0.0f, 0.0f, 1.0f }, { 0.0f, 1.0f } },
//     { { -0.5f, 0.5f, -0.5f }, { 1.0f, 1.0f, 1.0f }, { 1.0f, 1.0f } }
// };

// const std::vector<uint16_t> indices = {
//     0, 1, 2, 2, 3, 0,
//     4, 5, 6, 6, 7, 4
// };
class HelloTriangleApplication
{ 
    private:
	    ......
        std::vector<Vertex> vertices;
        std::vector<uint32_t> indices;
        VkBuffer vertexBuffer;
        VkDeviceMemory vertexBufferMemory;
        VkBuffer indexBuffer;
        VkDeviceMemory indexBufferMemory;
		......
};

	//modify
    vkCmdBindIndexBuffer(commandBuffer, indexBuffer, 0, VK_INDEX_TYPE_UINT32);
```
接下来变可引入`tiny_obj_loader.h`并声明函数`void loadModel()`，在`void initVulkan()`中调用。
```c++
#include <functional>
#define GLM_FORCE_CXX11
#define GLM_FORCE_RADIANS
#define GLM_FORCE_DEPTH_ZERO_TO_ONE
#include "glm/glm.hpp"
#define GLM_ENABLE_EXPERIMENTAL
#include <glm/gtx/hash.hpp>

struct Vertex 
{
    glm::vec3 pos;
    glm::vec3 color;
    glm::vec2 texCoord;

    static VkVertexInputBindingDescription getBindingDescription();   
    static std::array<VkVertexInputAttributeDescription, 3> getAttributeDescriptions(); 

    bool operator == (const Vertex& other) const;
};

namespace std{
    template<> struct hash<Vertex> {
        size_t operator() (Vertex const& vertex) const {
            return ((hash<glm::vec3>()(vertex.pos) ^ (hash<glm::vec3>()(vertex.color) << 1)) >> 1 ^ (hash<glm::vec2>()(vertex.texCoord) << 1));
        }
    };
};

class HelloTriangleApplication
{ 
    private:
	    ......
        void loadModel();
		......
};

bool Vertex::operator == (const Vertex& other) const
{
    return pos == other.pos && color == other.color && texCoord == other.texCoord;
}

void HelloTriangleApplication::loadModel()
{
    tinyobj::attrib_t attrib;
    std::vector<tinyobj::shape_t> shapes;
    std::vector<tinyobj::material_t> materials;
    std::string warn, err;

    if (!tinyobj::LoadObj(&attrib, &shapes, &materials, &warn, &err, MODEL_PATH.c_str()))
        throw std::runtime_error(warn + err);

    std::unordered_map<Vertex, uint32_t> uniqueVertices = {};

    for (const auto& shape : shapes)
    {
        for (const auto& index : shape.mesh.indices)
        {
            Vertex vertex{};

            vertex.pos = {
                attrib.vertices[3 * index.vertex_index + 0],
                attrib.vertices[3 * index.vertex_index + 1],
                attrib.vertices[3 * index.vertex_index + 2]
            };

            vertex.texCoord = {
                attrib.texcoords[2 * index.texcoord_index + 0],
                1.0f - attrib.texcoords[2 * index.texcoord_index + 1]
            };

            vertex.color = { 1.0f, 1.0f, 1.0f };

            if (uniqueVertices.count(vertex) == 0)
            {
                uniqueVertices[vertex] = static_cast<uint32_t>(vertices.size());
                vertices.push_back(vertex);
            }

            indices.push_back(uniqueVertices[vertex]);
        }
    }

}

void HelloTriangleApplication::initVulkan()
{ 
    ......
    loadModel();
    createTextureImageView();
    createTextureSampler();
    createVertexBuffer();
    createIndexBuffer();
    createUniformBuffers();
    createDescriptorPool();
    createDescriptorSets();
    createCommandBuffers();
    createSyncObjects();
}

```
# Generating Mipmaps
## Image creation
添加一个成员变量存储其mipmaplevels。并修改其`void createTextureImage()`,，`void createImage()`，`VkImageView createImageView()`，`void transitionImageLayout()`。
```c++
class HelloTriangleApplication
{  
    private:
		......

        uint32_t mipLevels;
        VkImage textureImage;
		......
};

void HelloTriangleApplication::createImage(uint32_t width, uint32_t height, uint32_t mipLevels, VkFormat format, VkImageTiling tiling, VkImageUsageFlags usage, VkMemoryPropertyFlags properties, VkImage& image, VkDeviceMemory& imageMemory)
{
    ......
    imageInfo.mipLevels = mipLevels;
    ......
}

VkImageView HelloTriangleApplication::createImageView(VkImage image, VkFormat format, VkImageAspectFlags aspectFlags, uint32_t mipLevel)
{
    ......
    viewInfo.subresourceRange.levelCount = mipLevel;
	......
}

void HelloTriangleApplication::transitionImageLayout(VkImage image, VkFormat format, VkImageLayout oldLayout, VkImageLayout newLayout, uint32_t mipLevels)
{
    ......
    barrier.subresourceRange.levelCount = mipLevels;
	......
}

void HelloTriangleApplication::createTextureImage()
{
    int texWidth, texHeight, texChannels;
    stbi_uc* pixels = stbi_load(TEXTURE_PATH.c_str(), &texWidth, &texHeight, &texChannels, STBI_rgb_alpha);
    VkDeviceSize imageSize = texWidth * texHeight * 4;
    mipLevels = static_cast<uint32_t>(std::floor(std::log2(std::max(texWidth, texHeight)))) + 1;

    if (!pixels)
        throw std::runtime_error("failed to load texture image!");

    VkBuffer stagingBuffer;
    VkDeviceMemory stagingBufferMemory;
    createBuffer(imageSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, stagingBuffer, stagingBufferMemory);

    void* data;
    vkMapMemory(device, stagingBufferMemory, 0, imageSize, 0, &data);
    memcpy(data, pixels, static_cast<size_t>(imageSize));
    vkUnmapMemory(device, stagingBufferMemory);

    stbi_image_free(pixels);

    createImage(texWidth, texHeight, mipLevels, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_TILING_OPTIMAL, VK_IMAGE_USAGE_TRANSFER_DST_BIT | VK_IMAGE_USAGE_SAMPLED_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, textureImage, textureImageMemory);

    transitionImageLayout(textureImage, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_LAYOUT_UNDEFINED, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL, mipLevels);
    copyBufferToImage(stagingBuffer, textureImage, static_cast<uint32_t>(texWidth), static_cast<uint32_t>(texHeight));
    transitionImageLayout(textureImage, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL, VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL, mipLevels);

    vkDestroyBuffer(device, stagingBuffer, nullptr);
    vkFreeMemory(device, stagingBufferMemory, nullptr);
}

//modify
createImage(swapChainExtent.width, swapChainExtent.height, 1, depthFormat, VK_IMAGE_TILING_OPTIMAL, VK_IMAGE_USAGE_DEPTH_STENCIL_ATTACHMENT_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, depthImage, depthImageMemory);

createImage(texWidth, texHeight, mipLevels, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_TILING_OPTIMAL, VK_IMAGE_USAGE_TRANSFER_DST_BIT | VK_IMAGE_USAGE_SAMPLED_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, textureImage, textureImageMemory);

swapChainImageViews[i] = createImageView(swapChainImages[i], swapChainImageFormat, VK_IMAGE_ASPECT_COLOR_BIT, 1); 
depthImageView = createImageView(depthImage, depthFormat, VK_IMAGE_ASPECT_DEPTH_BIT, 1);
textureImageView = createImageView(textureImage, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_ASPECT_COLOR_BIT, mipLevels);

transitionImageLayout(depthImage, depthFormat, VK_IMAGE_LAYOUT_UNDEFINED, VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL, 1); 
transitionImageLayout(textureImage, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_LAYOUT_UNDEFINED, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL, mipLevels);
```
## Generating Mipmaps
在`void createTextureImage()`调用中添加`VK_IMAGE_USAGE_TRANSFER_SRC_BIT`，以通知其我们将会使用纹理作为transifer的起点和终点。同时因为后续调用其他函数将其布局转换了，所以，在copy后我们先将其transitionImageLayout()注释掉。
```c++
void HelloTriangleApplication::createTextureImage()
{
	......
    createImage(texWidth, texHeight, mipLevels, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_TILING_OPTIMAL, VK_IMAGE_USAGE_TRANSFER_SRC_BIT | VK_IMAGE_USAGE_TRANSFER_DST_BIT | VK_IMAGE_USAGE_SAMPLED_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, textureImage, textureImageMemory);
	......
	    transitionImageLayout(textureImage, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_LAYOUT_UNDEFINED, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL, mipLevels);
    copyBufferToImage(stagingBuffer, textureImage, static_cast<uint32_t>(texWidth), static_cast<uint32_t>(texHeight));
    // transitionImageLayout(textureImage, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL, VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL, mipLevels);
	......
	generateMipmaps(textureImage, VK_FORMAT_R8G8B8A8_SRGB, texWidth, texHeight, mipLevels);
}
```
创建辅助函数`void generateMipmaps()`， 并在`void createTextureImage()`中调用。
```c++
class HelloTriangleApplication
{  
    private:
		......
        //Generatring Mipmaps
        void generateMipmaps(VkImage image, VkFormat imageFormat, int32_t texWidth, int32_t texHeight, uint32_t mipLevels);
		......
};

void HelloTriangleApplication::generateMipmaps(VkImage image, VkFormat imageFormat, int32_t texWidth, int32_t texHeight, uint32_t mipLevels)
{
    VkCommandBuffer commandBuffer = beginSingleTimeCommands();
    
    VkImageMemoryBarrier barrier{};
    barrier.sType = VK_STRUCTURE_TYPE_IMAGE_MEMORY_BARRIER;
    barrier.image = image;
    barrier.srcQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
    barrier.dstQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
    barrier.subresourceRange.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
    barrier.subresourceRange.baseArrayLayer = 0;
    barrier.subresourceRange.layerCount = 1;
    barrier.subresourceRange.levelCount = 1;

    int32_t mipWidth = texWidth;
    int32_t mipHeight = texHeight;

    for (uint32_t i = 1; i < mipLevels; i++)
    {
        barrier.subresourceRange.baseMipLevel = i - 1;
        barrier.oldLayout = VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL;
        barrier.newLayout = VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL;
        barrier.srcAccessMask = VK_ACCESS_TRANSFER_WRITE_BIT;
        barrier.dstAccessMask = VK_ACCESS_TRANSFER_READ_BIT;

        vkCmdPipelineBarrier(commandBuffer,
            VK_PIPELINE_STAGE_TRANSFER_BIT, VK_PIPELINE_STAGE_TRANSFER_BIT, 0,
            0, nullptr,
            0, nullptr,
            1, &barrier
        );

        VkImageBlit blit{};
        blit.srcOffsets[0] = { 0, 0, 0 };
        blit.srcOffsets[1] = { mipWidth, mipHeight, 1 };
        blit.srcSubresource.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
        blit.srcSubresource.mipLevel = i - 1;
        blit.srcSubresource.baseArrayLayer = 0;
        blit.srcSubresource.layerCount = 1;
        blit.dstOffsets[0] = { 0, 0, 0 };
        blit.dstOffsets[1] = { mipWidth > 1 ? mipWidth / 2 : 1, mipHeight > 1 ? mipHeight / 2 : 1, 1 };
        blit.dstSubresource.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
        blit.dstSubresource.mipLevel = i;
        blit.dstSubresource.baseArrayLayer = 0;
        blit.dstSubresource.layerCount = 1;

        vkCmdBlitImage(commandBuffer,
            image, VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL,
            image, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL,
            1, &blit,
            VK_FILTER_LINEAR
        );

        barrier.oldLayout = VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL;
        barrier.newLayout = VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL;
        barrier.srcAccessMask = VK_ACCESS_TRANSFER_READ_BIT;
        barrier.dstAccessMask = VK_ACCESS_SHADER_READ_BIT;

        vkCmdPipelineBarrier(commandBuffer,
            VK_PIPELINE_STAGE_TRANSFER_BIT, VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT, 0,
            0, nullptr,
            0, nullptr,
            1, &barrier
        );

        if (mipWidth > 1) mipWidth /= 2;
        if (mipHeight > 1) mipHeight /= 2;

    }
    barrier.subresourceRange.baseMipLevel = mipLevels - 1;
    barrier.oldLayout = VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL;
    barrier.newLayout = VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL;
    barrier.srcAccessMask = VK_ACCESS_TRANSFER_WRITE_BIT;
    barrier.dstAccessMask = VK_ACCESS_SHADER_READ_BIT;

    vkCmdPipelineBarrier(commandBuffer,
        VK_PIPELINE_STAGE_TRANSFER_BIT, VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT, 0,
        0, nullptr,
        0, nullptr,
        1, &barrier
    );

    endSingleTimeCommands(commandBuffer);
}
```
## Linear filtering support
在函数`void generateMipmaps()`中添加检查物理设备是否支持线性滤波。
```c++
void HelloTriangleApplication::generateMipmaps(VkImage image, VkFormat imageFormat, int32_t texWidth, int32_t texHeight, uint32_t mipLevels)
{
    VkFormatProperties formatProperties;
    vkGetPhysicalDeviceFormatProperties(physicalDevice, imageFormat, &formatProperties);

    if (!(formatProperties.optimalTilingFeatures & VK_FORMAT_FEATURE_SAMPLED_IMAGE_FILTER_LINEAR_BIT))
        throw std::runtime_error("texture image format dose not support linear blitting!");
	......
}
```
## Sampler
修改shader在不同lod采样贴图。

在`void createTextureSampler()`修改lod层级。
# Multisampling
## Getting available sample count
添加一个类成员保存能够支持的最高的采样次数。声明函数`void getMaxUsableSampleCount()`，并在函数`void pickPhysicalDevice()`中使用。
```c++
class HelloTriangleApplication
{  
    private:
		......
        VkSampleCountFlagBits msaaSamples = VK_SAMPLE_COUNT_1_BIT;
		......
		//Multisampling
		VkSampleCountFlagBits getMaxUsableSampleCount();
		......
};

VkSampleCountFlagBits HelloTriangleApplication::getMaxUsableSampleCount()
{
    VkPhysicalDeviceProperties physicalDeviceProperties;
    vkGetPhysicalDeviceProperties(physicalDevice, &physicalDeviceProperties);

    VkSampleCountFlags counts = physicalDeviceProperties.limits.framebufferColorSampleCounts & physicalDeviceProperties.limits.framebufferDepthSampleCounts;
    if (counts & VK_SAMPLE_COUNT_64_BIT) { return VK_SAMPLE_COUNT_64_BIT; }
    if (counts & VK_SAMPLE_COUNT_32_BIT) { return VK_SAMPLE_COUNT_32_BIT; }
    if (counts & VK_SAMPLE_COUNT_16_BIT) { return VK_SAMPLE_COUNT_16_BIT; }
    if (counts & VK_SAMPLE_COUNT_8_BIT) { return VK_SAMPLE_COUNT_8_BIT; }
    if (counts & VK_SAMPLE_COUNT_4_BIT) { return VK_SAMPLE_COUNT_4_BIT; }
    if (counts & VK_SAMPLE_COUNT_2_BIT) { return VK_SAMPLE_COUNT_2_BIT; }

    return VK_SAMPLE_COUNT_1_BIT;
}

void HelloTriangleApplication::pickPhysicalDevice()
{
    uint32_t deviceCount = 0;
    vkEnumeratePhysicalDevices(instance, &deviceCount, nullptr);

    if (deviceCount == 0)
        throw std::runtime_error("failed to find GPUs with Vulkan support!");
    
    std::vector<VkPhysicalDevice> devices(deviceCount);
    vkEnumeratePhysicalDevices(instance, &deviceCount, devices.data());

    for (const auto& device : devices)
    {
        if(isDeviceSuitable(device))
        {
            physicalDevice = device;
            msaaSamples = getMaxUsableSampleCount();
            break;
        }
    }

    if (physicalDevice == VK_NULL_HANDLE)
        throw std::runtime_error("failed to find a suitable GPU!");
}
```
## Setting up a render target
我们必须为multisampling创建其`image` `imageView` `deviceMemory`来存储。并修改函数`void createImage()`。声明函数`void createColorResources()`，在`void initVulkan()`中调用。同时，这也和我们是`swap chain`相绑定了，所以在`void cleanupSwapChain()`，`void recreateSwapChain()`也需要添加相应的代码。
```C++
class HelloTriangleApplication
{  
    private:
		......
        VkImage colorImage;
        VkDeviceMemory colorImageMemory;
        VkImageView colorImageView;
        ......
        void createColorResources();
        ......
};

void HelloTriangleApplication::createColorResources()
{
    VkFormat colorFormat = swapChainImageFormat;

    createImage(swapChainExtent.width, swapChainExtent.height, 1, msaaSamples, colorFormat, VK_IMAGE_TILING_OPTIMAL, VK_IMAGE_USAGE_TRANSIENT_ATTACHMENT_BIT | VK_IMAGE_USAGE_COLOR_ATTACHMENT_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, colorImage, colorImageMemory);
    colorImageView = createImageView(colorImage, colorFormat, VK_IMAGE_ASPECT_COLOR_BIT, 1);
}

void HelloTriangleApplication::createImage(uint32_t width, uint32_t height, uint32_t mipLevels, VkSampleCountFlagBits numSamples, VkFormat format, VkImageTiling tiling, VkImageUsageFlags usage, VkMemoryPropertyFlags properties, VkImage& image, VkDeviceMemory& imageMemory)
{
    ......
    imageInfo.samples = numSamples;
    ......
}

createImage(swapChainExtent.width, swapChainExtent.height, 1, VK_SAMPLE_COUNT_1_BIT, depthFormat, VK_IMAGE_TILING_OPTIMAL, VK_IMAGE_USAGE_DEPTH_STENCIL_ATTACHMENT_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, depthImage, depthImageMemory); 
createImage(texWidth, texHeight, mipLevels, VK_SAMPLE_COUNT_1_BIT, VK_FORMAT_R8G8B8A8_SRGB, VK_IMAGE_TILING_OPTIMAL, VK_IMAGE_USAGE_TRANSFER_SRC_BIT | VK_IMAGE_USAGE_TRANSFER_DST_BIT | VK_IMAGE_USAGE_SAMPLED_BIT, VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT, textureImage, textureImageMemory);

void HelloTriangleApplication::initVulkan()
{
    ......
    createColorResources();
    createDepthResources();
	......
}

void HelloTriangleApplication::recreateSwapChain()
{
    ......
    createImageViews();
    createColorResources();
    createDepthResources();
	......
}

void HelloTriangleApplication::cleanupSwapChain()
{
    vkDestroyImageView(device, colorImageView, nullptr);
    vkDestroyImage(device, colorImage, nullptr);
    vkFreeMemory(device, colorImageMemory, nullptr);
	......
}
```
## Adding new attachments
最后将其作为附件添加到我们的渲染管线。修改函数`void createRenderPass()`， `void createFramebuffers()`， `void createGraphicsPipeline()`
```c++
void HelloTriangleApplication::createRenderPass()
{
    VkAttachmentDescription colorAttachment{};
    colorAttachment.format = swapChainImageFormat;
    colorAttachment.samples = msaaSamples;
	......
    VkAttachmentDescription depthAttchment{};
    depthAttchment.format = findDepthFormat();
    depthAttchment.samples = msaaSamples;
	......
    VkAttachmentDescription colorAttachmentResolve{};
    colorAttachmentResolve.format = swapChainImageFormat;
    colorAttachmentResolve.samples = VK_SAMPLE_COUNT_1_BIT;
    colorAttachmentResolve.loadOp = VK_ATTACHMENT_LOAD_OP_DONT_CARE;
    colorAttachmentResolve.storeOp = VK_ATTACHMENT_STORE_OP_STORE;
    colorAttachmentResolve.stencilLoadOp = VK_ATTACHMENT_LOAD_OP_DONT_CARE;
    colorAttachmentResolve.stencilStoreOp = VK_ATTACHMENT_STORE_OP_DONT_CARE;
    colorAttachmentResolve.initialLayout = VK_IMAGE_LAYOUT_UNDEFINED;
    colorAttachmentResolve.finalLayout = VK_IMAGE_LAYOUT_PRESENT_SRC_KHR;
    ......
    VkAttachmentReference colorAttachmentResolveRef{};
    colorAttachmentResolveRef.attachment = 2;
    colorAttachmentResolveRef.layout = VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL;
    ......
    subpass.pColorAttachments = &colorAttachmentRef;
    subpass.pDepthStencilAttachment = &depthAttchmentRef;
    subpass.pResolveAttachments = &colorAttachmentResolveRef;
	......
    std::array<VkAttachmentDescription, 3> attachments = { colorAttachment, depthAttchment, colorAttachmentResolve };
	......
}

void HelloTriangleApplication::createFramebuffers()
{
    swapChainFramebuffers.resize(swapChainImageViews.size());

    for (size_t i = 0; i < swapChainImageViews.size(); i++)
    {
        std::array<VkImageView, 3> attachments = {
            colorImageView,
            depthImageView,
            swapChainImageViews[i]
        };
		......
    }
}

void HelloTriangleApplication::createGraphicsPipeline()
{
	......
    VkPipelineMultisampleStateCreateInfo multisampling{};
    multisampling.sType = VK_STRUCTURE_TYPE_PIPELINE_MULTISAMPLE_STATE_CREATE_INFO;
    multisampling.sampleShadingEnable = VK_FALSE;
    multisampling.rasterizationSamples = msaaSamples;
	......
}

```
## Quality improvements
开启Sample Shading
```c++
void createLogicalDevice() { 
	... deviceFeatures.sampleRateShading = VK_TRUE; 
	// enable sample shading feature for the device ... 
} 
void createGraphicsPipeline() { 
	... 
	multisampling.sampleShadingEnable = VK_TRUE; 
	// enable sample shading in the pipeline
	multisampling.minSampleShading = .2f; 
	// min fraction for sample shading; closer to one is smoother 
	... 
}
```
# Compute Shader
## Compute queue families
我们需要在选择物理设备时，考虑到是否能够支持compute shader。修改函数`bool isDeviceSuitable()`
## Loading compute shaders

### ## Preparing the shader storage buffers
我们首先为`storage buffers`声明其所需的成员变量，`VkBuffer`，`VkDeviceMemory`。并声明函数`void createShaderStorageBuffers()`对其进行初始化。
```c++
```
## Descriptors
声明函数`void createComputeDescriptorSetLayout()`构建其`DescriptorSetLayout`。并在`void createDescriptorPool()`为其分配引用。最后声明`void createComputeDescriptorSets()`在此对`Set`进行装填。
```c++

```
## Compute pipelines
声明函数`void ComputePipeline()`对`ComputePipeline`进行初始化。
```c++

```
## Compute shaders
创建文件computeShaders并输入particle相关的代码。
```glsl

```
## Running compute commands
声明函数`void recordComputeCommandBuffer()`录制对应的command。
```c++

```
### Submitting work
最后便是在`void drawFrame()`进行submit。
```c++

```
### Synchronizing graphics and compute
同时我们需要同步我们的compute shader 与 渲染帧。
在函数`void createSyncObjects()`来添加我们所需的同步的数据结构。并在函数`void drawFrame()`中使用。
```c++

```
## Drawing the particle system
声明我们的`Particle`结构体并提供函数成员函数`static VkVertexInputBindingDescription getBindingDescription()`，`static std::array<VkVertexInputAttributeDescription, 2> getAttributeDescriptions()`。
```c++

```