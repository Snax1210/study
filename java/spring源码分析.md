# spring 源码分析

## Spring AOP

AOP的编程思想就是把这些横切性的问题和主业务逻辑进行分离，从而起到解耦的目的

底层原理：  
Java动态代理、CGLIB代理（取决被代理的类是否为接口类，没有默认项）

this.singletonObjects.get(beanName)
createAopProxy


看源码技巧：  
1. 带着问题去看:有正确的返回类型的
2. 条件断点
3. 二分查找查看断点方法

底层实现：

java.lang.reflect.proxy#newProxyInstance
