大家好，我是可爱又机灵的开源小妹。

如今微服务架构和领域驱动设计 DDD 愈来愈盛行，于是我们有了大量的 DO 对象与 DTO 对象的映射转化场景。

以前的我都是傻乎乎的使用 getter / setter 方式转换，又慢又容易出错。

周末小妹在家好好的研究了一下，给大家带来开源项目 `Orika`，是一个使用字节码技术栈实现的**高性能 Java 对象映射框架**，就是众多映射工具中**简单易用又高效**的代表之作！

## 优势
### 性能
对比其他很多工具使用反射方式实现的映射，`Orika` 它是直接动态加载 Javasist 类库生成对象映射的字节码进行字段映射，这种方式**比传统的反射赋值，速度上会快很多**。

### 易用
无需手动敲重复的 getter / setter 方法，不用写繁琐的 **Convert** 转化类，**无需配置**就可直接使用！

### 灵活
支持两个对象的字段名不同的映射关系，也支持同一个字段名不同数据类型的转换，甚至于支持嵌套对象的字段映射，完全能够**满足你不同的转换需求**！

## 快速入门
### 1. 引包

```xml
<dependency>
   <groupId>ma.glasnost.orika</groupId>
   <artifactId>orika-core</artifactId>
   <version>1.5.4</version> <!-- or latest version -->
</dependency>
```

### 2. 获取映射工厂 MapperFactor

```Java
MapperFactor mapperFactory = new DefaultMapperFactor.Builder().build();
```
你可以使用 MapperFactor 注册指定的类之间的字段对应关系：

```Java
// 若两个对象字段一一对应，此步可以省略
mapperFactory.classMap(PersonSource.class, PersonDestination.class)
	.field("firstName", "givenName")
	.field("lastName", "sirName")
	.byDefault()
	.register();
```
当然也可以根据你的需求，注册自定义的 Converter 等。

### 3. 获取 MapperFacade，进行对象映射
上一步已经获取了映射工厂 MapperFactor，而这一步通过工厂获取实例，并进行映射。

```Java
// 获取实例
MapperFacade mapper = mapperFactor.getMapperFacade();

// 原对象
PersonSource source = new PersonSource();
// set some field values

// 映射，获取映射后的对象
PersonDestination destination = mapper.map(source, PersonDestination.class);
```


是不是很简单！`Orika` 的一大特点就是使用起来非常简单，当然如果你有更特殊的使用场景，可以自行查阅它的官方文档进行定制化，相信你会渐渐放弃让人奔溃的 getter / setter 方式。

我也不用天天加班了！(ó﹏ò。) 

## 对比
我还尝试了几款常见的对象映射框架，比如 Spring 的 BeanUtils, Dozer 和 MapStruct 等。

### BeanUtils
使用比较简单，但是使用反射 Method 的 invoke(Object obj, Object... args)去赋值，**效率低下**，并且不能支持不同名称的字段属性映射等复杂的场景。

### Dozer
有良好的定制化属性映射功能，支持简单属性、复杂类型的映射和递归映射等功能。但是同样使用了反射技术进行赋值，**效率非常不能让人满意**。

### MapStruct
是一个能够在编译期自动生成 Mapper 类的工具，自动生成的代码采用的 getter / setter 方式进行赋值，所以它的执行效率很高。但是 MapStruct 框架有一个致命的弱点，是使用起来比较繁琐，每一对映射对象都需要新增一个 Mapper 接口，再由编译时自动生成具体的实现，使用起来**不便利**。

```Java
@Mapper
public interface CarMapper {
 
    CarMapper INSTANCE = Mappers.getMapper( CarMapper.class );
 
		// 编译后会自动生成这个方法的具体实现
    @Mapping(source = "numberOfSeats", target = "seatCount")
    CarDto carToCarDto(Car car);
}
```

## 小妹总结
随着如 DTO 与 DO 类相互映射的场景越来越多，找到一个称手的对象映射框架非常重要。

今天小妹介绍的 `Orika` 框架，就是其中非常优秀的作品。它在使用的**便捷程度、灵活程度和性能**方便做出了很好的平衡，非常推荐大家上手试一试这款优秀的开源作品！

听到开源小妹的安利，你是不是也有点心动了，赶紧去公众号后台回复「**映射**」，获取开源项目的地址吧~~~

问君能有几多愁，开源项目解千愁，小妹和大家说再见啦，下期再见哦！
