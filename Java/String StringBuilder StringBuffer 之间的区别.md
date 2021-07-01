### String 字符串常量

是不可变对象，每次对String类型进行改变的时候都等同于生成了一个新的String对象，然后将指针指向新的String对象，所以性能最差，对于要经常改变内容的字符串不用String。



### StringBuilder 字符串变量（非线程安全）

是字符串变量，对它操作时，并不会生成新的对象，而是直接对该对象进行更改，所以性能较好，不保证线程安全

适合在单线程的情况下使用

### StringBuffer 字符串变量（线程安全）

StringBuffer和StringBuilder一样，是字符串变量，但是他带有synchronized关键字，保证线程安全，所以性能比 StringBuilder 。适合多线程的情况下使用





性能上通常StringBuilder > StringBuffer > String。

- String：适用于少量的字符串操作的情况。
- StringBuilder：适用于单线程下在字符缓冲区进行大量操作的情况。
- StringBuffer：适用多线程下在字符缓冲区进行大量操作的情况。