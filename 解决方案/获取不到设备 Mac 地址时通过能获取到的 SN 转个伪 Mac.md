当设备获取不到 mac 的时候，但是业务又需要用 mac 做识别，有一种简单的伪 mac 的思路，因为业务方不校验 mac 是否合理（不考虑是不是合法的 mac），只校验格式：
假设 mac 地址字符串一般长这样 00-01-6C-06-A6-29 或 00:01:6C:06:A6:29，此处我们采用 : 隔开 16 进制数， 比如能够获取到的 SN 是 06723881 ，做个简单的映射
代码里把 06723881 三位三位截取得到 067 672 723 238 388 881 
然后分别用 intToHex 做转换，然后超过 2 位截取后 2 位，不够 2 位前面补 0
最终组合得到 43:a0:d3:0e:04:71