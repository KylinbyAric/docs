#概述
##分类
1、创建型
常用：单例、工厂、建造者
不常用：原型
2、结构型
常用：代理、桥接、装饰者、适配器
不常用：门面、组合、享元
3、行为型
常用：观察者、模版、策略、责任链、迭代器、状态
不常用：访问者、备忘录、命令、解释器、中介
##原则
###单一原则
怎么判断不满足：
1、代码行数/私有方法过多
2、引用的类过多
3、很难给一个类起名字，什么manager、context
###开闭原则
对扩展开放、对修改关闭
大白话就是能让代码做到在扩展时是以新增而非修改的方式。这样会减少对已有代码(包括实现方和调用方)的改动以及改动后的测试成本
对拓展开放是为了应对变化(需求)，对修改关闭是为了保证已有代码的稳定性；最终结果是为了让系统更有弹性！
具备扩展、抽象、封装潜意识

###里式替换
子类的修改不会影响父类的运行正确性，不会违背父类的设计初衷。
这是关于怎么设计子类的原则，对于子类，更多应该是让他扩展父类的功能，而非修改父类的功能。


###接口隔离
定义的接口尽可能单一，尽量可以继承多个接口

###依赖倒置
依赖注入
这是一种实现技巧，调用类本身不生成和管理依赖类，而是通过参数形式去依赖。
控制反转
这是一种指导框架的思想，面向过程是程序员控制所有的执行流程，控制反转则是框架控制执行流程，程序员只负责编写业务的代码。



DRY：？？
###KISS
尽量保持简单

###YAGNI
不要过度设计，当前不需要的功能，预留一下扩展点就行，不用过多考虑全场景。快速迭代才是王道

###LOD
```java
/*
高内聚是指导类(模块/系统)的设计，低耦合是指导类(模块/系统)间的依赖关系设计
高内聚是指将相同功能放到同一个类
1、这样修改时能同时修改，而不会遗漏
2、有利于提高可读性
低耦合是让依赖关系简单清晰


//如单一原则、开闭原则就是高内聚的体现，依赖倒置、接口隔离、基于接口而非实现、里氏替换、迪米特则是低耦合的体现

*/
```
不该有直接依赖的就不要有依赖，要依赖时尽力依赖接口

###描述代码好坏的维度和词汇
0、可维护性
1、可读性：清晰、简单、分层清晰、整洁
2、复用性:模块化、
3、扩展性：高内聚低耦合
4、高性能
5、安全性
6、健壮性
7、兼容性

精心编写的可维护性可读性复用性代码和快速写的代码+改bug时间都是差不多的。

###滥用get、set
1、这跟没有访问权限有什么区别？
    比如购物车中，商品list和total，如果两者都暴露出去都可以修改，那么非常容易会造成list和total不一致的情况。应该提供统一的操作方法，比如clear、
    addItem方法。即便不提供set,集合属性的get方法也会让集合内的对象发生变化，也是非常不安全的。
2、滥用常量类和全局方法
    个人觉得：
    常量类和一些enum类应该跟着它的主体类走，比如Redis相关的常量类就应该放到RedisConfig中。
    某个类的状态enum应该直接以该类的内部的形式存在。
    某些数字，比如MaxSize、LimitNUm等，应该写成常量，这样以后要修改时，可以只改一处代码就行。到处找不仅繁琐，还很容易遗漏出错。
    
    
###基于接口设计
接口更多是为了抽象解耦，抽象类则更多是为了复用代码。接口设计可以有更多的扩展性、维护性。接口是有设计，再有实现从上到下的设计，抽象类则是先有子类的
复用，是自下而上的。
具体做法：
1、函数命名不能暴露实现细节或者跟业务相关的
2、封装具体细节
3、使用者依赖接口来编写代码
4、当某功能越稳定不怎么需要变动时，则不需要定义接口
```java
//这样的调用方式会导致再发生修改时，需要改所有调用处。怎么解决呢？
ImageStore imageStore = new PrivateImageStore(/*省略构造函数*/);

//1、策略模式+context，context持有该实例，调用处直接调用context.getInstance之类的方法。这样变动时只需要修改一处就好了。
//2、Spring依赖注入方式

```
###多用组合，少用继承
继承的问题
1、子类会新增不必要的方法，增加被误调的概率
2、随着特殊属性的增加，抽象类会倍增，子类还需要查看父类的逻辑代码

可以通过组合、委托、接口的方式解决
1、组合+委托解决了代码复用的问题
2、接口解决了多继承问题
```java

public interface Flyable {//接口
  void fly();
}
public class FlyAbility implements Flyable {
  @Override
  public void fly() { //... }
}
//省略Tweetable/TweetAbility/EggLayable/EggLayAbility

public class Ostrich implements Tweetable, EggLayable {//鸵鸟
  private TweetAbility tweetAbility = new TweetAbility(); //组合
  private EggLayAbility eggLayAbility = new EggLayAbility(); //组合
  //... 省略其他属性和方法...
  @Override
  public void tweet() {
    tweetAbility.tweet(); // 委托
  }
  @Override
  public void layEgg() {
    eggLayAbility.layEgg(); // 委托
  }
}
```

```java
/*
六边形项目结构（根据实际情况自行组织与定义）：

* InboundHandler 代替 controller
    * *WebController：处理 web 接口
    * *WeChatController：处理微信公众号接口
    * *AppController：处理 app 接口
    * *MqListener：处理 消息
    * *RpcController：处理子系统间的调用
* service 服务于 use case，负责的是业务流程与对应规则
    * CQPS + SRP：读写分离和单一原则将 use case 分散到不同的 service 中，避免一个巨大的 service 类（碰到过 8000 行的 service）
* Domain 服务于核心业务逻辑和核心业务数据
    * 最高层组件，不会依赖底层组件
    * 易测试
* outBoundhandle 代替 rep
    * MqProducer：发布消息
    * Cache：从缓存获取数据
    * sql：从数据库获取数据
    * Rpc：从子系统获取数据

----
*/
```

DDD和普通MVC
1、DDD的controller和dao层基本没太多差异
2、区别在于service，service层的核心业务对象domain会封装数据和操作。把原先MVC中的service中的核心业务放到domain。
好处在于
    1、核心业务和普通service剥离，提高可读性、扩展性
    2、domain的核心业务与框架、存储层彻底隔离，移植性高
3、这要求domain一定要与业务高度相关
```java
 /**
   potato00fa:我对DDD的看法就是，它可以把原来最重的service逻辑拆分并且转移一部分逻辑，可以使得代码可读性略微提高，另一个比较重要的点是使得模型充血以后，
基于模型的业务抽象在不断的迭代之后会越来越明确，业务的细节会越来越精准，通过阅读模型的充血行为代码，能够极快的了解系统的业务，对于开发来说能说明显的提升开发效率。
   在维护性上来说，如果项目新进了开发人员，如果是贫血模型的service代码，无论代码如何清晰，注释如何完备，代码结构设计得如何优雅，都没有办法第一
时间理解系统的核心业务逻辑，但是如果是充血模型，直接阅读充血模型的行为方法，起码能够很快理解70%左右的业务逻辑，因为充血模型可以说是业务的精准抽象，
我想，这就是领域模型驱动能够达到"驱动"效果的由来吧
   **/
```

###重构
####单元测试
1、单元测试一定是要针对功能来编写测试用例，而不是针对内部实现逻辑。这样以后功能重构，测试用例也不会因为内部逻辑的修改而运行错误
2、单元测试多做一些，尽量不要黑盒测试，有意识的测试解决bug的时间要比无意识的要短。
3、耐心在单元测试中真的很重要。


###一些准则
####注释尽量只写"做什么"
因为怎么做可能经常变化，但做什么就很少。除非一些复杂逻辑，但复杂逻辑也应该拆分成多个单一简单逻辑。
####参数不要过多
超过3个则需要考虑是否拆分函数或者用类包裹参数
####函数体不要过长、一行不要太长
####不要用布尔/null等参数来控制函数逻辑
这违背了单一职责，可以考虑把该函数拆成多个函数

###一些checklist
####是否有基本的错误，比如变量引用错误、未判空。
####是否违背了一些设计原则或者思想。
####如果要修改/新增的话，容易么？不容易的原因是在哪？
####是否过度设计了？
####是否可读啊？

###一些设计模式
####单例
线程池、数据库连接池建议不要设计成单例，因为这通常需要扩展，而单例模式对扩展并不友好。单例一定程度上相当于放弃了多态和继承关系。













