title: 求下一个2的幂
date: 2019/04/18
categories:
- Data Analysis
tags:
- Algorithm
---


## 最直观 ##

```scala
def getNextPowerOf2(x: Int): Int = {
    if (x > (Int.MaxValue >> 1) + 1) {
        return 0
    }

    if ((x & (x - 1)) == 0) {
        return x
    }

    val k = (math.log10(x)/math.log10(2.0)).ceil
    math.pow(2.0, k).toInt
}

```

## 移位操作 ##

```java
n--;           // 1101 1101 --> 1101 1100
n |= n >> 1;   // 1101 1100 | 0110 1110 = 1111 1110
n |= n >> 2;   // 1111 1110 | 0011 1111 = 1111 1111
n |= n >> 4;   // ...
n |= n >> 8;
n |= n >> 16;  // 1111 1111 | 1111 1111 = 1111 1111
n++;           // 1111 1111 --> 1 0000 0000
```

- 右移1位，做异或，保证最高位、最高位下一位为“1”；这样有2位为“1”了，进而右移2位，做异或，保证最高4位为“1”；进而在右移4位（不停的乘2），做异或……最后得到所有位都为“1”，再`n++`，可得一个2的幂。
- `n--`是为了处理`n`本身是2的幂的场景。如不做`n--`，会得到`n`的2倍作为结果。
- 处理32位整数，可扩展，如处理64位整数，添加`n >> 32`。

```scala
def getNextPowerOf2(x: Int): Int = {
    if (x > (Int.MaxValue >> 1) + 1) {
        return 0
    }

    var n = x - 1
    n |= n >> 1
    n |= n >> 2
    n |= n >> 4
    n |= n >> 8
    n |= n >> 16
    n += 1
    n
}

```