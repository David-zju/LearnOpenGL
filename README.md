# CMake & vcpkg & Git 构建C++工作环境学习笔记

> 声明：本帖主要为自己摸索的经验分享，欢迎批评指正

写代码前最重要的且最痛苦的一步就是配置环境。在本科课程学习时，我们的代码通常只有一个或几个文件，也很少用到第三方库，因此大多数情况下我们会直接拿开箱即用的IDE（如Dev C++，Clion，Visual Studio等）来进行开发工作，而熟悉命令行操作的同学则也会在命令行内直接手动编译链接在文本编辑器中写好的代码（如vscode）。而在之后的学习科研中，我们不仅会面临开发环境变化的问题（例如将Windows下的代码移植到Linux平台上），还有可能会在开发中使用大量的第三方乃至于自己编写的库。这个时候如果使用命令行来编译如此庞大的项目显然过于低效，而QT等IDE的代码在移植到别的开发平台时总是会遇到各种各样的困难，因此lz选择学习使用CMake和vscode作为接下来开发C++的主要工具。

另外，相比于Linux，在windows下似乎没有特别统一的包管理工具。对于C++而言，微软的vcpkg是一个很好的选择。而要在多个平台上同步开发，Git是必不可少的（然鹅lz在之前在命令行内会使用的唯一操作就是git clone [捂脸]）因此在这也整理了一个Git的超超超入门教程。

# 1. Git 操作简介

> 资料来源：[图解Git](https://marklodato.github.io/visual-git-guide/index-zh-cn.html)

- Git 相关概念

  <img src="https://user-images.githubusercontent.com/55902119/202471985-46690ff8-db37-43cf-8140-17213fd6e848.png" alt="drawing" height="300"/> </br>
  暂存（add），提交（commit），reset，checkout（签出）

  1. 提交（commit）可以理解成给Git提交了一个存档点，在之后的代码过程中都可以回溯到这里。在Git中，每次提交时必须填写一条记录（这个信息是必填的，否则寻找当初修改记录时无疑是大海捞针）
     提交的新存档节点如果是列表的末尾，那么就把此时的节点设为父节点。然后把当前分支指向新的提交节点。而即便当前分支是某次提交的祖父节点，git会同样操作。下图中，在main分支的祖父节点stable分支进行一次提交，生成了1800b。 这样，stable分支就不再是main分支的祖父节点。此时，[合并](https://marklodato.github.io/visual-git-guide/index-zh-cn.html#merge) (或者 [衍合](https://marklodato.github.io/visual-git-guide/index-zh-cn.html#rebase)) 是必须的。

  <img width="748" alt="%E6%88%AA%E5%B1%8F2022-11-17_14 26 32" src="https://user-images.githubusercontent.com/55902119/202472713-aab8fe0a-1b41-418e-8f1d-7f2f43362ef0.png">
  <img width="618" alt="%E6%88%AA%E5%B1%8F2022-11-17_14 25 32" src="https://user-images.githubusercontent.com/55902119/202472745-7524fd33-9ca1-4238-aa1a-0b4bda3ef5f8.png"><br/>

  1. 暂存（add）是代码提交前的一个必需步骤，它将我们修改的部分转为stage状态。当提交（commit）时会把很多次暂存（add）的结果合并为一次修改。
  2. 签出（checkout）用于从历史提交（或者暂存区域）中拷贝文件到工作目录，也可用于切换分支

- Git 的安装以及Github相关的设置
  >由于git大多数同学应该已经安装，如果没有安装的话其实网上的教程也非常详尽，可以参考以下两个
  
  [Git 安装配置](https://www.runoob.com/git/git-install-setup.html)

  [Github 简明教程 | 菜鸟教程](https://www.runoob.com/w3cnote/git-guide.html)

- 在vscode中使用Git

  由于Git是一款命令行工具，因此很多同学比较推崇使用命令行而非图形化工具来操作，个人感觉是仁者见仁智者见智的。不过对于初学者来说，个人感觉图形化界面在使用上会更直观一点。lz比较推荐的几款图形化工具有Github官方客户端（对于使用Github很好用），GitKranken（非常强大，功能齐全），vscode插件（轻量，而且vscode内置）

  vscode内部已经把Git常用的命令图形化化了，因此不需要在命令行内手动敲代码。具体的使用可以直接过一遍微软官方的教程：

  [在 Visual Studio Code 中使用 Git 版本控制工具 - Learn](https://docs.microsoft.com/zh-cn/learn/modules/use-git-from-vs-code/)
  
  （当然如果熟悉上面的几个git的基本概念之后，图形界面基本上可以零基础直接上手了，实在有问题再回来看官方的教程）

# 2. 环境安装与配置

> 参考官方文档‣
> 注意，MacOS在执行以下流程前，需要先安装Apple Developer Tools（我是直接装了整个Xcode 23333）
> 在Windows操作系统下，得事先确保有一个可用的C++编译器（lz使用MSVC）

## 2.1 安装vcpkg

[https://github.com/microsoft/vcpkg](https://github.com/microsoft/vcpkg)

使用 git 将其下载到本地后执行自动安装脚本

```bash
> git clone https://github.com/microsoft/vcpkg
> .\vcpkg\bootstrap-vcpkg.bat
```

- 可选：将vcpkg的安装目录添加到系统环境变量path
  (这样就可以在其他文件夹下直接调用vcpkg的命令了)

    **Mac/Linux** 
    给Linux/Unix 系统增加环境变量，是使用`export` 命令。为了永久生效，需要考虑加入到系统的配置文件中。在User目录下存在一个系统终端（如bash/zsh等）的配置文件，如图所示。 

    <img width="749" alt="%E6%88%AA%E5%B1%8F2022-11-16_20 54 21" src="https://user-images.githubusercontent.com/55902119/202475341-fd857066-45f9-4981-bffa-9701e74f4bec.png">

    Mac下显示隐藏文件的快捷键为 `ctrl`+ `shift`+ `.`。需要加上一句

    ```jsx
  export PATH=/usr/Dev/vcpkg:$PATH
    ```

    在环境变量中，各个值是以冒号分隔开的。上面的语句表示给 `PATH` 重新赋值，同时在后面加上原来的 `PATH` 。

    **Windows**
    直接搜索环境变量，添加vcpkg包的根目录即可

## 2.2 安装cmake

[Download | CMake](https://www.notion.so/aea175b45a984c5896382ee31ff7c9e6)

在官网选择对应的CMake版本下载安装即可

## 2.3 vcpkg 安装所需库

以C++中常用的矩阵库Eigen为例，当我们想要安装时，键入

<!-- ![Untitled 1](https://user-images.githubusercontent.com/55902119/202475629-86b02e30-d8b8-4f00-8b2a-dd6f0b34a4e0.png) -->

  <img src="https://user-images.githubusercontent.com/55902119/202475629-86b02e30-d8b8-4f00-8b2a-dd6f0b34a4e0.png" alt="drawing" height="100"/> </br>

说明这个库的名字不对，使用search查看正确的名字

<img src="https://user-images.githubusercontent.com/55902119/202476354-fa365133-5ee4-480b-b8fa-3dd05229c368.png" alt="drawing" height="200"/> </br>

这样我们就知道了正确的名字为eigen3，执行

```bash
vcpkg install eigen3 
```

默认为32位，如果要编译并安装64位版本，执行

```bash
vcpkg install eigen3:x64-windows
```

当然在arm版Mac上也能自动检测系统架构来进行编译

<img width="565" alt="%E6%88%AA%E5%B1%8F2022-11-16_21 39 24" src="https://user-images.githubusercontent.com/55902119/202476586-e0ddc41e-f00b-42c7-b5cb-0bf37167fd59.png">

这里最后有提示我们在cmake内应该如何配置，当然有些包可能并不会有提示

如果你很懒（例如像当初那个搞不明白VSCode的我一样），可以用Visual Studio来写C++，在此之前只需要在命令行执行

```bash
vcpkg integrate install
```

之后所有由vcpkg安装并管理的包都会被自动识别出来（哦耶）。到此为止，你就可以愉快地开始coding了。如果你对VSCode抱着一种奇怪的执念，或者是有跨平台编译一定需要cmake，请往下看。

## 2.4 VSCode上的配置

首先需要在电脑上安装cmake并配置好环境变量

在VSCode上安装CMake以及C++相关拓展

1. **C/C++ Extension**

   ![Untitled 3](https://user-images.githubusercontent.com/55902119/202476743-8f40a0f3-91ac-4756-85b2-e89b9f787ccf.png)

2. **CMake Extenstion**

   ![Untitled 4](https://user-images.githubusercontent.com/55902119/202476788-068c6678-98f7-404e-858c-cedd7ebdbd2b.png)

3. **CMake Tools Extension**

   ![Untitled 5](https://user-images.githubusercontent.com/55902119/202476913-124edbd5-8088-4c60-a620-0c7e71f262bb.png)

安装完毕后为了能够让VSCode知道vcpkg管了一堆包，我们需要在vscode中添加设置。方法为在settings.json 中写入

```jsx
{
    "cmake.configureSettings": {
        "CMAKE_TOOLCHAIN_FILE":
            "{vcpk_repository}/scripts/buildsystems/vcpkg.cmake"
    }
}
```

由于VSCode的设置管理形式为json文件，因此我们可以直接在全局的settings.json里面改，也可以在项目文件夹下新建一个.vscode 文件夹，然后在里面新建一个如上内容的json文件。（鉴于vscode会在不同设备之间同步设置，因此我建议还是local setting比较靠谱，不然可能会因为奇怪的报错而抓耳挠腮好久）

# 3. 操作案例

> 一般大家在图形学课程中都需要使用C++写大作业，而图形学需要调用很多OpenGL库，项目结构较为复杂，比较适合作为案例分享

## 3.1 新建文件夹与CMakeLists.txt

项目结构和CMakeLists.txt内容如下：

<img width="250" alt="%E6%88%AA%E5%B1%8F2022-11-16_22 50 20" src="https://user-images.githubusercontent.com/55902119/202477669-cfe2fb3f-13e6-41c2-bd52-beb27676d074.png">
<img width="488" alt="%E6%88%AA%E5%B1%8F2022-11-17_11 11 30" src="https://user-images.githubusercontent.com/55902119/202477713-6857bdf4-2481-490a-82f7-a04b7cdf85a8.png">

在cmake中，使用find_package找到我们所需要的包，然后再使用target_link_libraries将我们编译的主程序与包链接在一起，当在cmake完成配置后（就是下一步要做的），我们在主程序内就可以直接include我们想要的第三方库了。

<img width="594" alt="%E6%88%AA%E5%B1%8F2022-11-17_11 10 26" src="https://user-images.githubusercontent.com/55902119/202477895-b805dbef-0be4-48c0-bbd0-6c7069645cd6.png">

## 3.2 配置项目

用Ctrl+Shift+P就可以搜索到需要的相关命令，首先选择CMake的工具包和变量

![Untitled 6](https://user-images.githubusercontent.com/55902119/202478061-844c3284-6183-488e-8908-e506e5ccf287.png)
![Untitled 7](https://user-images.githubusercontent.com/55902119/202478074-26213c0d-269b-4ae2-849b-9e8ca4f35a79.png)


这里如果选择这个工具包，则需要注意cmake会自动寻找存在的64位/32位包，如果引用的其中一个包只安装了32位，而另一个包只安装了64位，cmake的报错显示为找不到xxx包。（然而实际上只是编译版本不太对）如果只使用64位或32位工具包则也需要本地安装的包与编译器匹配。

然后选择variant，我们需要调试就选debug

![Untitled 8](https://user-images.githubusercontent.com/55902119/202478215-84737551-3964-451e-bd1c-f9fd2c63ee69.png)

然后配置CMake（相当于在build 目录执行 `cmake..`)

> 如果配置失败，一种可能的原因是没有删除之前的配置缓存，需要选择删除缓存并配置（也能通过Ctrl+Shift+P找到）

![Untitled 9](https://user-images.githubusercontent.com/55902119/202478356-499c644d-2604-4ea5-8d50-555db5107fee.png)

## 3.3 完成代码撰写

在主程序中，我们的main.cpp的内容如下

```cpp
#include <glad/glad.h>
#include <GLFW/glfw3.h>

#include <iostream>

void framebuffer_size_callback(GLFWwindow* window, int width, int height);
void processInput(GLFWwindow *window);

// settings
const unsigned int SCR_WIDTH = 800;
const unsigned int SCR_HEIGHT = 600;

int main()
{   
    // glfw: initialize and configure
    // ------------------------------
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

#ifdef __APPLE__
    glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
#endif

    // glfw window creation
    // --------------------
    GLFWwindow* window = glfwCreateWindow(SCR_WIDTH, SCR_HEIGHT, "LearnOpenGL", NULL, NULL);
    if (window == NULL)
    {
        std::cout << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }
    glfwMakeContextCurrent(window);
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);

    // glad: load all OpenGL function pointers
    // ---------------------------------------
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
    {
        std::cout << "Failed to initialize GLAD" << std::endl;
        return -1;
    }    

    // render loop
    // -----------
    while (!glfwWindowShouldClose(window))
    {
        // input
        // -----
        processInput(window);

        // render
        // ------
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // glfw: swap buffers and poll IO events (keys pressed/released, mouse moved etc.)
        // -------------------------------------------------------------------------------
        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    // glfw: terminate, clearing all previously allocated GLFW resources.
    // ------------------------------------------------------------------
    glfwTerminate();
    return 0;
}

// process all input: query GLFW whether relevant keys are pressed/released this frame and react accordingly
// ---------------------------------------------------------------------------------------------------------
void processInput(GLFWwindow *window)
{
    if(glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
        glfwSetWindowShouldClose(window, true);
}

// glfw: whenever the window size changed (by OS or user resize) this callback function executes
// ---------------------------------------------------------------------------------------------
void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
    // make sure the viewport matches the new window dimensions; note that width and 
    // height will be significantly larger than specified on retina displays.
    glViewport(0, 0, width, height);
}
```

上面的OpenGL部分的内容不是本次的重点，想要详细了解的可以参考下面的链接。

[你好，窗口](https://learnopengl-cn.github.io/01%20Getting%20started/03%20Hello%20Window/)

## 3.4 代码编译运行与调试

还是Ctrl+Shift+P，找到cmake：build，当然直接调试其实也是会事先编译一次的（即使之前有编译结果的情况下）

<img width="596" alt="%E6%88%AA%E5%B1%8F2022-11-17_11 27 18" src="https://user-images.githubusercontent.com/55902119/202478812-91e64055-0e06-42a1-b8fb-cad0a0f6396a.png">

然后打上断点就可以直接调试了（选择上面的debug）可以看到左侧可以预览当前的变量等信息

<img width="1058" alt="%E6%88%AA%E5%B1%8F2022-11-17_11 36 07" src="https://user-images.githubusercontent.com/55902119/202478900-8e6978da-4f32-4e83-a054-032cf9c8c594.png">

输出会在调试控制台内显示

<img width="727" alt="%E6%88%AA%E5%B1%8F2022-11-17_11 43 46" src="https://user-images.githubusercontent.com/55902119/202478951-1242bd2b-7e44-4ec2-aadc-3e8357b184e6.png">

最后程序的效果是生成一个如图所示的窗口

<img width="912" alt="%E6%88%AA%E5%B1%8F2022-11-17_11 48 29" src="https://user-images.githubusercontent.com/55902119/202478997-a1f217b2-743a-4aed-bde9-b503c126f98f.png">

## 3.5 发布与同步

<img width="306" alt="%E6%88%AA%E5%B1%8F2022-11-17_11 50 40" src="https://user-images.githubusercontent.com/55902119/202479135-60aab09d-889b-4373-b2e6-438a570c7fa3.png">
<img width="606" alt="%E6%88%AA%E5%B1%8F2022-11-17_11 53 25" src="https://user-images.githubusercontent.com/55902119/202479179-b9905972-828b-4963-9c46-089a738356fc.png">
<img width="606" alt="%E6%88%AA%E5%B1%8F2022-11-17_11 54 37" src="https://user-images.githubusercontent.com/55902119/202479237-45d21c96-985c-40c9-8824-5219e877bfe8.png">


在左侧选择Publish to Github，选择是否为公开仓库，再选择需要上传的部分（这相当于自动帮你创建了.gitignore文件，进阶用法可以学习gitignore文件的格式规范）

创建完毕之后就可以使用左侧的工具栏进行git操作了。

<img width="320" alt="%E6%88%AA%E5%B1%8F2022-11-17_14 07 31" src="https://user-images.githubusercontent.com/55902119/202479572-4403fb4b-aee8-4e3f-b723-4016eee1a717.png">

这里在根目录下新建了一个readme.md文件

