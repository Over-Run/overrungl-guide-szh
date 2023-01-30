# JOML 模块

JOML 模块负责将 [JOML](https://github.com/JOML-CI/JOML) 中的值类型转换为`MemorySegment`。

## 分配内存

`allocate`方法为值类型分配内存。内存的生命周期与分配器一致。

```java
Matrix4f projection = new Matrix4f().setOrtho2D(0, width, 0, height);
MemorySegment segment = Matrixn.allocate(SegmentAllocator.nativeAllocator(SegmentScope.auto()),
    projection);
GL.uniformMatrix4fv(uProjection, 1, false, segment);
```

`malloc`方法重载则使用 [MemoryUtil](core.md#memoryutil) 或 [MemoryStack](core.md#memorystack)。

## put
