# Java

## jdk1.8中对hash算法和寻址算法是如何优化的

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

hash值的高16为与低16位进行异或运算

意义：

高低16位都参与运算



寻址算法优化

(n - 1) & hash -> 数组里的一个位置