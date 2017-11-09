### 缓存范围：

Java 为整型值包装类 Byte、Character、Short、Integer、Long 设置了缓存，用于存储一定范围内的值，详细如下：

| 类型        | 缓存值范围           |
| --------- | --------------- |
| Byte      | -128 ~ 127      |
| Character | 0 ~ 127         |
| Short     | -128 ~ 127      |
| Integer   | -128 ~ 127（可配置） |
| Long      | -128 ~ 127      |

### 例子：

在一些情况下，比如自动装箱时，如果值在缓存值范围内，将不创建新对象，直接从缓存里取出对象返回，比如：

```
Integer v1 = 10;
Integer v2 = 10;
Integer v3 = new Integer(10);
Integer v4 = 128;
Integer v5 = 128;
Integer v6 = Integer.valueOf(10);
System.out.println(v1 == v2); // true
System.out.println(v1 == v3); // false
System.out.println(v4 == v5); // false
System.out.println(v1 == v6); // true
```

### 缓存实现机制：

这里使用了设计模式享元模式。

以 Short 类实现源码为例：

```
// ...public final class Short extends Number implements Comparable<Short> {
    // ...
    private static class ShortCache {
        private ShortCache(){}

        static final Short cache[] = new Short[-(-128) + 127 + 1];

        static {
            for(int i = 0; i < cache.length; i++)
                cache[i] = new Short((short)(i - 128));
        }
    }

    // ...
    public static Short valueOf(short s) {
        final int offset = 128;
        int sAsInt = s;
        if (sAsInt >= -128 && sAsInt <= 127) { // must cache
            return ShortCache.cache[sAsInt + offset];
        }
        return new Short(s);
    }
    // ...}
```

在第一次调用到 `Short.valueOf(short)` 方法时，将创建 -128 ~ 127 对应的 256 个对象缓存到堆内存里。

这种设计，在频繁用到这个范围内的值的时候效率较高，可以避免重复创建和回收对象，否则有可能闲置较多对象在内存中。