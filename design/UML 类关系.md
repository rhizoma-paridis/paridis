# class

## 泛化关系（Generalization）

泛化（Generalization）也是继承关系。implements 使用虚线，extends 使用实线

![implement](attachments/Pasted%20image%2020221102153109.png)

![extend](attachments/Pasted%20image%2020221102153120.png)

## 组合关系（Composition）

组合关系中的对象间的关系比聚合更紧密。各对象无法独立存在，只能组合成一个整体。

![](attachments/Pasted%20image%2020221102153030.png)

通常在整体类的构造方法中直接实例化成员类。也就是这些对象有相同的生命周期。

## 聚合关系（Aggregation）

聚合关系的对象间的关系比关联关系要紧密一点，属于各对象组成一个整体。但是各对象又能独立存在。

![](attachments/Pasted%20image%2020221102144203.png)

## 关联关系（Association）

关联关系中有双向关联，单向关联，自关联，

![](attachments/Pasted%20image%2020221102144127.png)

如像 Teacher 与 Address 这种属性关系，address 对于 Teacher 来说完全是可有可无。

## 依赖关系（Dependency）

这里的依赖并不是 implement，而是使用关系。

它们只轻度的在方法中出现关联。

1. 方法参数是另外一个对象

   ![](attachments/Pasted%20image%2020221102144053.png)

2. 方法中居部变量是另外一个对象

3. 方法中调用另一个类的静态方法

## 结论

> 上面几种关系中对象的紧密度为：泛化 > 组合 > 聚合 > 关联 > 依赖
