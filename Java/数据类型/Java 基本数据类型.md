Primitive Data Type  基本(原生)数据类型

- 四类八种：整数型(4种)、浮点(小数)型(2种)、布尔型、字符型 
- 定义时变量引用指向具体的数值，值存储在栈内存中
- 用 == 比较

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210219191924767.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xvdWlzZ2Vlaw==,size_16,color_FFFFFF,t_70#pic_center)


|数据类型 | byte/字节 | bit/比特/位                        | 区间范围 最小值 — 最大值 | 默认值 |
| ------- | -------- | --------- | ---------------------------------- | -------------------- |
| byte 字节型 Byte | 1    | 8       | -128 — 127(-2^7 — 2^7-1) | 0                    |
| short 短整型 Short | 2    | 16      | -32768 — 32767(-2^15 — 2^15-1) | 0                    |
| int 整型 Integer | 4    | 32      | -2^31 — 2^31 -1                    | 0                    |
| long 长整型 Long | 8    | 64      | -2^63 — 2^63 -1                    | 0L                   |
|         |          |           |                                    |                      |
| float 单精度浮点型 Float | 4    | 32      | 1.401298E-45 — 3.402823E+38       | 0.0F                 |
| double 双精度浮点型 Double | 8    | 64      | 4.900000E-324 — 1.797693E+308     | 0.0D                 |
|         |          |           |                                    |                      |
| boolean 布尔型 Boolean | 1       | 8        | true / false                       | false                |
|         |          |           |                                    |                      |
| char 字符型 Character | 2   | 16      | \u0000 — \uFFFF (0 — 65535) | \u0000 |

- 未显示声明类型的数值类型
  - 整数类型默认 int
  - 浮点类型默认 double
- 浮点类型可能只是一个近似值，并非精确的值，有限 离散 舍入误差 大约 接近但不等于
- 数据范围不一定与字节数相关
- byte 字节是计算机中最小的存储单元、最小的数据类型
- bit 位是最小的信息单位
- char  ASCII

### 字面值

> literal 字面常值(字面值、字面量、直接量、源文本)

- 整数字面值默认是 int
如果要表示 long ，则需要加 l 或者 L 
- 浮点字面值默认是 double
如果要表示 float，则需要加 f 或者 F 



单精度
双精度