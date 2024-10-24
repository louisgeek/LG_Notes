## JDK 动态代理

通过生成继承了 Proxy 并实现了接口列表的对应代理类的字节码， 然后通过 defineClass 方式定义对应的代理类, 最后通过反射创建代理对象实例





## Android JDK 动态代理

直接通过方法、接口列表、ClassLoader 等参数直接生成了对应代理类， 省去了生成字节码的过程，将代理类的生成都放在了 navtive 层







## 代理类的方法执行原理

代理类在 static 块通过放射将要代理的方法保存在成员变量中， 具体方法执行的时候获取对应的方法通过 Proxy 的 InvocationHandler 的 invoke 方法 代理具体的方法，这里的 InvocationHandler 其实就是我们 Proxy#newInstance 传入的 InvocationHandler