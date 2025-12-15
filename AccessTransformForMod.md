# Apply AT to Mod

ForgeGradle7的重构为Forge的AT提供了一个独立的gradle plugin，并且它的实现基于ArtifactTransform，因此几乎所有的Artifact都能被它所处理。
如果你需要修改mod里的权限修饰符来避免在mixin里使用反射灯操作，你可以按照以下步骤操作。

1. 将Gradle Plugin Portal加入plugin的repo里

`settings.gradle`:
```
pluginManagement {
    repositories {
        gradlePluginPortal()
    }
}
```
 
2. apply [`net.minecraftforge.accesstransformers`](https://plugins.gradle.org/plugin/net.minecraftforge.accesstransformers) 这个插件
```
id("net.minecraftforge.accesstransformers") version "5.0.3"
```
  
3. 使用下列语法来告知插件你需要AT哪个依赖，使用的AT文件是哪些
```groovy
implementation('net.minecraftforge:coremods') {
    accessTransformers.configure(it) {
        config = project.file('accesstransformer.cfg')
    }
}
```
其中，accessTransformers.confiture(it)表示标记了`implementation('net.minecraftforge:coremods')`这个依赖，
config = project.file('accesstransformer.cfg')表示使用的AT配置是当前[Project](https://docs.gradle.org/current/userguide/part1_gradle_init.html#step_1_initializing_the_project)下的`accesstransformer.cfg`文件

## 注意

这个插件只会修改字节码，并不会修改源代码，因此不要被idea里看到的源码误解，只要编译通过了即可，同时它只会处理开发环境使用的jar，因此你需要自己在生产环境处理，由于forge/neoforge的AT在运行时都没有额外的检查，因此你可以直接在AT file里写mod类
