---
title: androidstudio-develop-skills
tags:

---



1. 无法使用导入的framework.jar

具体体现： 龙哥电脑上修改了framework源码并且编译了jar包导入AS之后，ctrl+1有提示，但是还是报红，而且build报错

原因：Android11全编之后在目录out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/下只能生成classes-header.jar，如果想得到classes.jar，还需要执行 make javac-check-framework，同时生成的classes.jar中不包括WifiSsid等class文件，需要去out/target/common/obj/JAVA_LIBRARIES/framework-wifi.com.android.wifi_intermediates/下找到classes.jar

解决：得到两个classes.jar之后，放在一个临时目录下分别解压，删掉两个jar包再生成一个framework.jar，再导入到AS中。

 jar -xvf xx.jar

rm *.jar

jar -cvfM framework.jar ./



---

# 随记

- 每个AS项目都包括一个或者多个内有源代码文件和资源文件的模块，模块类型有Android应用模块、库模块、Google APP Engine模块。
- 每个应用模块都包括manifests、java、res三个文件夹。
- ctrl+shift+f12 隐藏所有工具窗口
- AS可以在build.gradle中声明模块依赖项，gradle会自动查找并提供。
- 应用模块化：对应用进行模块化处理就是将应用项目的逻辑组建拆分成独立模块的过程。
- URI：通用资源标识符，通过URI可以定位网络上额资源。
- URL：统一资源定位符，是URI的子集，不同的是URL还定义了访问资源的方式，如何种协议等。
- 实时模板：https://medium.com/google-developers/writing-more-code-by-writing-less-code-with-android-studio-live-templates-244f648d17c7#.h1jn0hq31
- 通过File > New > Import Sample可以查找示例代码，支持查看预览、导入。
- 

1. 快捷键
2. 调试

- 使用logcat查看日志，可以通过自定义tag对结果进行过滤。

- Debug断点调试：

  Show Excution Point：定位到当前正在被执行的断点
  Step Over：单步执行，即前进到下一行代码
  Step Into：进入调试方法的第一行，不能调到类库方法内
  Force Step Into：与Step Into方法类似，但是可以进入类库方法
  Step Out：前进到当前方法外的下一行，与Step Into方法配合使用
  Drop Frame：返回到方法执行的初始点
  Run Tp Cursor：跳转到光标所在处，需要当前断点已经执行到最后一个，且光标所在的代码行要符合由上到下的执行顺序，不能颠倒

- 条件断点：当断点打在循环体内时，不需要不停地点击执行，可以右击断点，在弹出的Condition窗口里加入任何布尔表达式。

- 变量赋值：可以通过更改断点监控到的变量的值，来改变app运行的结果。

- 日志断点：能够在断点处输出日志，从而避免了在代码中写 Log，然后再重新运行程序。

- 异常断点

- Suspend选项：右击断点后会出现Suspend选项，可以挂起全部或者部分线程。

- 禁用断点

- 断点组：调试某些功能时打上的断点暂时不需要又不想删除时，可以通过断点组管理切换断电状态

> https://developer.android.google.cn/studio/debug 官方调试参考文档

---

\1. 导入eclipse项目到AS中，直接在AS中import eclipse ADT ，会遇到一些问题，如无法build之类的，所以正确的步骤应该是：

删除项目目录下的.classpath文件和.project文件，再执行import eclipse ADT，导入之后执行gradle sync操作时会报错，需要在build.gradle文件中加入 google() 字段，然后可能在导入jar时报错，可能的解决方式为改compile files为implementation files，然后就正常了

\2. 导入framework.jar

AS 4.2以下的版本导入framework.jar步骤：

复制framework.jar到moudle的libs目录下

add as library 

​                compileOnly files('libs/framework.jar')              

​                project的build.gradle tasks.withType(JavaCompile) {        options.compilerArgs.add('-Xbootclasspath/p:libs/framework.jar')    }              

AS 4.2 以上：

​                moudle的build.gradle gradle.projectsEvaluated {    tasks.withType(JavaCompile) {        Set<File> fileSet = options.bootstrapClasspath.getFiles()        List<File> newFileList =  new ArrayList<>();        //JAVA语法，可连续调用，输入参数建议为相对路径        newFileList.add(new File("libs/framework.jar"))        //最后将原始参数添加        newFileList.addAll(fileSet)        options.bootstrapClasspath = files(                newFileList.toArray()        )    } }              

参考链接：https://blog.csdn.net/u014175785/article/details/116235760
