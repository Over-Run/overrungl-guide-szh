# OverrunGL 使用手册

为 [OverrunGL](https://github.com/Over-Run/overrungl) 编写的使用手册。

如果你认为本手册有问题，请创建一个 [Issue](https://github.com/Over-Run/overrungl-guide-szh/issues)。  
如果你在使用 OverrunGL 的过程中遇到问题，请发起新[讨论](https://github.com/Over-Run/overrungl/discussions)。

## 简介

OverrunGL 是使用 Java 20 编写的本机绑定库，提供了与 C 库交互的能力。

OverrunGL 使用了最新的 [FFM API](https://openjdk.org/jeps/434)，它使我们无需编写任何 C 代码即可调用本机库。

OverrunGL 旨在令方法调用更类似于纯 Java。以下是一段 FFM API 与 GLFW 模块结合使用的例子：

```java
import java.lang.foreign.MemorySegment;
// imports...

class Example {
    private static MemorySegment windowHandle;

    public static void main(String[] args) {
        // init...
        try (Arena arena = Arena.openShared()) {
            windowHandle = GLFW.createWindow(arena,
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

我们将在接下来的章节中详细讲解各个模块与本机交互的方式。
