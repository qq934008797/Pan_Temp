# Spring源码深度解析 (第2版)

## 整体架构
Core Container：提供了控制反转和依赖注入
	Beans: 配置文件、创建、进行Ioc/DI 
	Core:核心工具类
	Context: 在beans和core的基础之上，提供了JNDI（java name Directory Interface）注册的框架式访问，提供了大量扩展
			添加了国际化、资源绑定、事件传播、和对Context的透明创建支持。 ApplicationContext接口是Context模块的关键
	Expresstion Language:在运行时查询和操作对象，获取对象的属性值、方法调用、访问上下文、容器、逻辑运算、命名、检索名称对象、list投影等
Data Asscess/Integration:数据访问/集成
	jdbc:java Database Connectivity 消除冗余的数据库连接
	orm： 入jpa、hibernate、ibatis封装 交互层
	oxm：对xmlbean、Xstream抽象层
	jms：消息制造和消费
	transaction：声明性事务管理；
Web：
	web模块：面向Web的集成特性、上传、使用servlet listeners声明性事务管理ioc及面向web应用的上下文
	web-servlet：spring mvc
	web-porlet：prolet
AOP：
	Aspects：提供了对AspectJ的集成
	Instrumentation：提供了class instrumentation支持和classloader实现，使得可以在特定的应用服务器上使用	
	
## AOP-AspectJ

### AspectJ 语言简洁
	AspectJ实现了AOP
	脱离了Spring当初看AspectJ的使用方法，跟javac的编译平级， ajc.exe
	无需修改HelloWord.java文件，却控制了java的执行，在其中干自己的事情，完成切面
	
	执行命令： ajc HelloWord.java TestAspect.java
	关键词：around当程序执行HelloWord对象的sayHello的方法适合执行下面这个代码块     proceed标识调用原有的方法
	demo: 要求不改变helloWord类的情况下加入事务语言；
```java
public class HellWord{
   public void sayHello(){
	sout("HellWord");
   }
   public static void main(String[] args){
		HellWord g  = new HellWord();
		g.sayHello();
   }
}
```
```aspect
public aspect TestAspect{
   void around():call(void sayHello()){
	sout("111111");
	proceed();
	sout("22222");
   }
}
```
Alias别名：获取对象的实例名字转换
## 2.容器-核心类：DefalutListableBeanFactory
		DefaultListableBeanFactory(综合所有功能，对bean 注册后的处理)
			extends AbstractAutowireCapableBeanFactory
			implements ConfigurableListableBeanFactory
						, BeanDefinitionRegistry
### BeanDefinitionRegistry(接口：定义对BeanDifinition的各种方法)
		BeanDefinitionRegistry extends AliasRegistry(接口：定义对alias的简单的增删改查的) 
### ConfigurableListableBeanFactory(接口：并继承：Beanfactory配置清单、指定忽略类型及接口)
		ConfigurableListableBeanFactory extends ListableBeanFactory(根据各种条件获取bean的配置清单)
		   , AutowireCapableBeanFactory, ConfigurableBeanFactory
### AbstractAutowireCapableBeanFactory(综合对：AbstractBeanFactory并对 AutowireCapableBeanFactory 进行实现)
    AbstractAutowireCapableBeanFactory extends 
	AbstractBeanFactory (综合了：FactoryBeanRegistrySupport和ConfigurablBeanFactory的功能)
	implements
		AutowireCapableBeanFactory(接口：提供了创建bean、自动注入、初始化、应用的bean后处理器)
			extends	BeanFactory:(接口：定义获取bean及bean的各种属性)
	
	FactoryBeanRegistrySupport(抽象类：在defaultSingletonBeanRegistry基础上增加了对FactoryBean的特殊功能处理)
		extends DefaultSingletonBeanRegistry(实现：单例接口)
			 extends SimpleAliasRegistry(实现：map作为alias的缓存，且实现  AliasRegistry )
  			 implements SingletonBeanRegistry(接口：定义对单例的注册及获取)
	ConfigurablBeanFactory：(接口：提供配置Factory的各种方法)
	
	
## 2.容器- 核心类：XmlBeanDefinitionReader
ResourceLoader:定义资源加载器，
BeanDefinitionReader：实现定义资源接口，并转换为BeanDefinition 的功能
EvironmentCapable：定义从资源文件转换为Document的功能
DocumentLoader：定义从资源文件到Document的功能
AbstractBeanDefinitionReader：对EnviromentCapable、BeanDefinitionReader类定义的实现
BeanDefinitionDocumentReader：定义读取Document 并注册BeanDefinition功能
BeanDefinitionParserDelegate：定义解析element的各种方法
流程：
	通过继承自 AbstractBeanDefinitionReader的方法，来使用ResourceLoader讲资源文件路径转换为对应的Resource文件；
	通过对DocumentLoader将Resource文件进行转换，将Resource文件转换为Document文件
	通过实现接口BeanDefinitionDocumentReader的DefaultBeanDefinitionDocumentReader类对Document进行解析，并使用BeanDefinitionParserDelegate对Element进行解析

## 容器的基础 XmlBeanFactory

### Resource
	java中，将不同的资源抽象成url，通过注册不同的handler来处理不同来源的资源的读取逻辑，但是没有定义相对Classpath或者ServletContext等资源的
	handler，虽然可以注册自己的urlStreamhandler来解析特定的URL前缀协议，比如“classpath：且URL也没有提供基本的方法，可是资源是否存在、是否可用、可读。
	而Spring内部就对使用到的资源实现了抽象接口，提供了存在exists、可读、是否打开、不同资源转换、createRelative创建相对资源的方法。还提供了了getDescription来处理错误打印信息

### 加载Bean
	XmlBeanFactory构造函数调用了XmlBeanDefinitionReader类型的reader属性提供的方法（this.reader.loadBeanDefinition(resource)）
	对应时序图：
	 XmlBeanFactory》XmlBeanDefinitionReader》new EncodeResource》loadBeanDefinition》doLoadBeanDefinitoin(inputSource,enCodeResource.getResourece).return(loadBeanDefinition)
	 获取对xml文件袋验证模式；加载xml文件、并得到对应的document；根据document注册bean信息
### 获取Xml 的验证模式，校验 XmlValidationModeDetector.getValidationModeForResource
	为了保证xml文件的正确性，比较常用的两种DTD和XSD；DTD 文档定义类型，xml约束模式语言，是xml文件的验证机制，属于xml文件的一部分。xsd
	XML Schema 是基于 XML 的 DTD 替代者。XML Schema 可描述 XML 文档的结构。
### DefaultDocumentLoader
	与平常的sax解析xml套路一致，spring没有什么特殊的地方；首先创建DocumenBuilderFactory，然后在创建DocumentBuilder，进而解析inputSource来返回Doucment对象。
	对于entityResolver解释，提供了如何寻找dtd声明的方法，DelegatingEntityResolver类为EntityResolver的实现类
```java
package interpreterPattern;
import com.singularsys.jep.*;
public class JepDemo
{
    public static void main(String[] args) throws JepException
    {
        Jep jep=new Jep();
        //定义要计算的数据表达式
        String 存款利息="本金*利率*时间";
        //给相关变量赋值
        jep.addVariable("本金",10000);
        jep.addVariable("利率",0.038);
        jep.addVariable("时间",2);
        jep.parse(存款利息);    //解析表达式
        Object accrual=jep.evaluate();    //计算
        System.out.println("存款利息："+accrual);
    }
}
```


