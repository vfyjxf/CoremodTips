# Mixin构造器合并的行为

Mixin具有一个特性，允许你往目标类插入新的field member,那么它背后是如何工作的？

(本文基于net.fabricmc:sponge-mixin:0.15.2+mixin.0.8.7)


## 具体流程
对于java来说，java会将`初始化块`放置在`构造函数体`前，`super`调用后面。

Mixin会对Mixin类的`随机(具体的说，是第一个发现有LineNumber的构造函数,顺序取决于ClassNode#methods)`一个构造函数进行行号分析，提取出来夹在`super调用`和`构造函数体`当中的`初始化块`，

然后检查是否包含下列非法字节码指令

并且检查获取的初始化块的最后一个指令是否是`PUTFIELD`,若不是，则非法。

### 行号分析流程

- 获取构造函数所有字节码操作
- 获取到super调用前的第一个行号标注
- 获取到最后一个`PUTFIELD`指令后的,`RETURN`指令前的行号标注
- 若不存在`RETURN`指令(如果在构造函数使用throw语句,则没有`RETURN`指令)，返回非法区间



### 非法的字节码
```
Opcodes.RETURN, Opcodes.ILOAD, Opcodes.LLOAD, Opcodes.FLOAD, Opcodes.DLOAD,

Opcodes.ISTORE, Opcodes.LSTORE, Opcodes.FSTORE, Opcodes.DSTORE, Opcodes.ASTORE
```


如果你是上游mixin，以下字节码也是非法的:
```
Opcodes.IALOAD, Opcodes.LALOAD, Opcodes.FALOAD, Opcodes.DALOAD, Opcodes.AALOAD,

Opcodes.BALOAD, Opcodes.CALOAD, Opcodes.SALOAD, Opcodes.IASTORE, Opcodes.LASTORE,

Opcodes.FASTORE, Opcodes.DASTORE, Opcodes.AASTORE, Opcodes.BASTORE, Opcodes.CASTORE, Opcodes.SASTORE
```

## 用来参考的Java代码

```java

public class X {

  //这里的赋值是初始化块
  public int a = 1;


    //这里也是初始化块,但是由于mixin的限制，这里不能出现上述替代的非法字节码
  {
    foo();
    ...
  }

  public X() {
  //super调用    
  //初始化块代码被插入的地址
  //构造函数体
  }

}

```


## 一些注意事项和小Tips

- 最好(**永远**)不要在Mixin类的构造函数里面直接throw 异常，mixin并不会合并构造函数体到目标类当中，因此你不必做任何保护措施
- 由于正常方法调用并不在`初始化块`的黑名单字节码当中，因此类似下面的Mixin代码也会被正常合并到目标类当中
```java

@Mixin(BeMixinClass.class)
public class MixinBeClass {
    
    @Unique
    private int my$abc;

    {
        System.out.println("FromMixinInit");
        my$abc = 10;//注意这里，由于mixin会检查初始化块的最后一个指令是否是PUTFIELD，因此这里的赋值是必要的
    }

    public MixinBeClass() {
        System.out.println("FromMixin");//没有任何作用，Mixin不会合并到目标类的构造函数里
    }

}

public class BeMixinClass  {

    public BeMixinClass(int a,int b,int c,Object d){
        //插入到这里
        System.out.println();
        System.out.println();
    }

}

```
