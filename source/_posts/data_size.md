title: Data Size
date: 2018/10/21
categories:
- Program Development
tags:
- Underlying
---


- 1 bit是1位，0 ~ 1，可表示$2^{1}$个数。
- 1 Byte是1字节，8 bits，0000 0000 ~ 1111 1111，即0 ~ 255、0x00 ~ 0xFF，可表示$2^{8} = 256$个数。
- Java中：

|类型|长度|
|---|---|
|byte|1 Byte （FF）|
|short|2 Bytes （FFFF）|
|int|4 Bytes （FFFF FFFF）|
|long|8 Bytes （FFFF FFFF FFFF FFFF）|
|float|4 Bytes|
|double|8 Bytes|
|boolean|1 bit|
|char|2 Bytes|

