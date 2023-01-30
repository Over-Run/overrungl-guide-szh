# 核心模块

**核心** (Core) 模块，是 OverrunGL 各个模块的基础。

我们为核心模块添加了以下基本 API：

- `MemoryStack`
- `MemoryUtil`
- `RuntimeHelper`

除此之外，我们还添加了`Configurations`，供开发者设置内部参数。

## MemoryStack

由于目前 FFM API 并未提供栈操作 API，因此我们提供了与 LWJGL 相似的`MemoryStack`。

`MemoryStack`允许我们进行一次性、快速的小堆内存分配。例如，以下为 OpenGL 中常见的栈操作：

```c
unsigned int texture;
glGenTextures(1, &texture);
// use the texture
```

显然在 Java 中我们没有指针，无法直接传入值类型修改内容。因此，我们需要借助`MemoryStack`完成以上操作：

```java
int texture;
try (MemoryStack stack = MemoryStack.stackPush()) {
    MemorySegment segment = stack.malloc(ValueLayout.JAVA_INT);
    GL.genTextures(1, segment);
    texture = segment.get(ValueLayout.JAVA_INT, 0);
}
// use the texture
```

!!! hint
    注意`ValueLayout.JAVA_INT`是 FFM API针对目前通用泛型尚未加入的情况下临时提出的解决方案，它的类型为`ValueLayout.OfInt`。  
    未来，通用泛型加入时，上面的代码可能会变成下面这样：

```java
int texture;
try (MemoryStack stack = MemoryStack.stackPush()) {
    MemorySegment segment = stack.<int>malloc();
    GL.genTextures(1, segment);
    texture = segment.<int>get(0);
}
// use the texture
```

---

`MemoryStack`还实现了`SegmentAllocator`，这意味着我们能够将`MemoryStack`直接传入参数。

```java
import java.lang.foreign.MemorySegment;
// imports...

class Example {
    private static MemorySegment windowHandle;

    public static void main(String[] args) {
        // init...
        try (MemoryStack stack = MemoryStack.stackPush()) {
            // 请注意只有小堆内存才能使用 MemoryStack 分配，
            // 如果字符串的长度超过了栈的大小会导致 OutOfMemoryError。
            windowHandle = GLFW.createWindow(stack,
                800,
                600,
                "Example",
                MemorySegment.NULL,
                MemorySegment.NULL);
        }
        // ...terminate
    }
}
```

---

`MemoryStack`默认大小可通过设置`Configurations.STACK_SIZE`修改。`STACK_SIZE`单位为KiB。

调用`stackPush`时，每个线程会使用默认大小创建一个堆栈，或者复用之前的堆栈。

使用`create(long)`显式指定创建堆栈的大小。

通过设置`Configurations.DEBUG_STACK`，启用或禁用调试模式。此配置必须在首个`MemoryStack`创建前设置。

## MemoryUtil

!!! note
    尽管我们提供了`MemoryUtil`，但是除非非常需要性能，否则仍然推荐使用`Arena`。

!!! important
    使用`MemoryUtil`分配的内存必须显式释放，否则会造成内存泄露！  
    幸运的是，通过设置`Configurations.DEBUG_MEM_UTIL`，我们能够快速定位内存泄露位置。

`MemoryUtil`是 C 标准内存分配函数的绑定，它的使用方法与 C 一致。

```java
// import static ...
MemorySegment segment = malloc(ValueLayout.JAVA_LONG);
// 正常情况下返回值不会为 NULL，此处仅作示例
if (segment.address() == RuntimeHelper.NULL) {
    throw new OutOfMemoryError();
}
segment.set(ValueLayout.JAVA_LONG, 0, 42L);
System.out.println(segment.get(ValueLayout.JAVA_LONG, 0));
// 重要！
free(segment);
```

## RuntimeHelper

!!! warning
    `RuntimeHelper`为底层内容，如果你不知道自己在做什么，请跳过这一节。

### API Logger

OverrunGL 内部使用名为`apiLogger`的`Consumer`来打印内部消息。

要将`apiLogger`设置为你自己的 logger（例如 slf4j），使用以下代码：

```java
RuntimeHelper.setApiLogger(logger::warn);
```

### Downcall Handle

FFM API将本机函数转换为`MethodHandle`以便我们在 Java 中调用。

关于这部分的详细说明请见前言中链接的 JEP 页面。

### toArray

`RuntimeHelper`提供了`toArray`方法及其重载。`toArray`将`MemorySegment`中的内容读取到数组中。
