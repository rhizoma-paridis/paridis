内部类分类。

1. 定义在外部类的方法体内/代码块
    - 局部内部类（有类名）
    - 匿名内部类（没有类名）
2. 定义在外部类的成员位置上
    - 静态内部类（有static修饰符）
    - 成员内部类（没有static修饰符）

## 成员内部类

这个是最常用的。

```Java
public class Out {

    private String name;
    
    private void haha() {
        System.out.println("haha");
    }

    private class Inner{
        private String name;
        
        public void hello() {
            Out.this.haha();
            Out.this.name = "out hello";
            name = "inner hello";
        }
    }
}
```

`private class Inner`使用`private`时，Inner 只能在Outer内部使用。无法将Inner暴露出去。但是无论修饰符是什么，都不影响 Inner 访问 Outer的属性与方法。包括私有属性与方法。

但是访问属性时可以省略`Out.this`。在省略时，如果内部类有与外部类同名的属性，会根据就近原则，即默认情况下访问的是成员内部类的成员。

在外部使用内部类语法：`Out.Inner inner = new Out().new Inner();`因为内部类是依附于外部类的，所以要先创建内部类。

## 局部/匿名内部类

```Java
public class Outer {//外部类

    public void OuterFun1() {
        System.out.println("外部类成员方法");
        class Inner {//局部内部类
            
        }
    }
}
```

1. 跟局部变量一样，不能使用修饰符。可以使用 final，但是有什么用呢？
2. 作用域在代码块里。

其他行为与成员内部类大致一样。

```Java
public class Outer {//外部类
    Object obj = new Inner(){
        @Override
        public void innerFun1() {
            
        }
    };
}

interface Inner {
    public void innerFun1();
}
```

他最大的作用我觉得就是提供一个方法，所以jdk8后，碰见这种情况最好使用lambda。当然如果说这里非要有两个方法。我觉得那就是违背了函数式编程的柯里化。

## 静态内部类

这是唯一一个能在class前面添加static的地方。

```Java
public class Outer {//外部类

    public static class Inner {//内部类

    }
}
```

因为Inner不再持有`Outer.this`，所以他更像是一个正常的类。只是因为在Outer内部，作用于的原因，可以访问 Outer的私有静态变量。

1. 无法直接访问Outer 私有成员变量。

## 总结

内部类以为作用域的特殊性，所以可以访问外部类的私有属性。一切特点都是围绕这一点的。有没有 static 决定了内部类有没有 `Outer.this`。这东西决定了访问外部成员变量的方式，其他的就没什么了。

修饰符决定类内部类能不能暴露出去。但是一般设计内部类就是不愿意暴露出去。所以不用 public 就可以了。