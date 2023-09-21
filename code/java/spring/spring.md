
## Instantiate bean 构建 bean 实例。

使用无参数构造器创建

## BeanDefinitionReader

XmlBeanDefinitionReader

## initializeBean

### populateBean

	普通属性
	对象属性
### invokeAwareMethods

这里主要是 BeanFactoryAware 的执行， 而 ApplicationContextAware 则是被 ApplicationContextAwareProcessor 封装，在 BeanPostProcessor 过程中执行。

### BeanPostProcessor

### invokeInitMethods

invokeInitMethods

执行 Init 方法

## registerDisposableBeanIfNecessary

注册 destory 方法

prototype 与 singleton 实例区别在于不加入 singletonObjects 缓存与不注册 destory 方法。

BeanFactoryPostProcessor


FactoryBean