> HTML
> 
> - Hyper Text Markup Language 超文本标记语言
> 
> XML 
> 
> - Extensible Markup Language 可扩展标记语言
> - 为啥不叫 EML？难道 Ex 读着像 X ?

DOM 

- Document Object Model 文档对象模型
- 解析速度慢，内存占用大

一次性读取 XML，把 XML 解析成一个树形结构放入内存中

```
Document 代表整个 XML 文档
Element 代表一个 XML 元素
Attribute 代表一个元素的某个属性
```

SAX

- Simple API  For XML

逐行扫描文档，一边扫描一边解析

以流的形式读取 XML，边读取XML边解析，使用事件回调

```
startDocument：开始读取XML文档；
startElement：读取到了一个元素，例如<book>；
characters：读取到了字符；
endElement：读取到了一个结束的元素，例如</book>；
endDocument：读取XML文档结束。
```

PULL

- 