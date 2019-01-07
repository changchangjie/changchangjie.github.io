---
title: 徒手撸框架-实现IOC
date: 2018-11-02 14:00:00
categories: 
- Spring
---

### IOC是什么
IoC 不是一种技术，只是一种思想。传统应用程序都是由我们在类内部主动创建依赖对象，从而导致类与类之间高耦合，难于测试；有了IoC容器后，把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，所以对象与对象之间是 松散耦合，这样也方便测试，利于功能复用。
IoC很好的体现了面向对象设计法则之一—— 好莱坞法则：“别找我们，我们找你”；即由IoC容器帮对象找相应的依赖对象并注入，而不是由对象主动去找
<!--more-->

### IOC有什么用
在 Java 中 Everything is Object，我们的程序就是由若干对象组成的。当我们的项目越来越大，合作的开发者越来越多的时候，我们的类就会越来越多，类与类之间的引用就会成指数级的增长。如下图所示：
![](https://ws4.sinaimg.cn/large/006tNc79ly1fnd1qwnp8qj30aa06ot8t.jpg)
这样的工程简直就是灾难，如果我们引入 Ioc 框架。由框架来维护类的生命周期和类之间的引用。我们的系统就会变成这样：
![](https://ws4.sinaimg.cn/large/006tKfTcly1fnd1yq1vzpj30aa06v0sr.jpg)

用一个类比来理解这个问题。IOC框架就是我们生活中的房屋中介，首先中介会收集市场上的房源，分别和各个房源的房东建立联系。当我们需要租房的时候，并不需要我们四处寻找各类租房信息。我们直接找房屋中介，中介就会根据你的需求提供相应的房屋信息。大大提升了租房的效率，减少了你与各类房东之间的沟通次数

### Spring 的 IoC 是怎么实现的
了解Spring框架最直接的方法就阅读Spring的源码。但是Spring的代码抽象的层次很高，且处理的细节很高。Spirng IoC 主要是以下几个步骤。
1. 初始化 IoC 容器。
2. 读取配置文件。
3. 将配置文件转换为容器识别对的数据结构（这个数据结构在Spring中叫做 BeanDefinition)
4. 利用数据结构依次实例化相应的对象
5. 注入对象之间的依赖关系

### 自己实现一个IoC框架
1. 定义bean的数据结构

```java
public class BeanDefinition {

    private String name;

    private String className;

	//get set
}
```

2. 工具类

```java
//负责处理 Java 类的加载
public class ClassUtils {

    public static ClassLoader getDefultClassLoader(){
        return Thread.currentThread().getContextClassLoader();
    }

    public static Class loadClass(String className) throws ClassNotFoundException {
        return getDefultClassLoader().loadClass(className);
    }
}
//负责处理对象的实例化
public class BeanUtils {

    public static <T> T instanceByCglib(Class<T> clz, Constructor ctr, Object[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(clz);
        enhancer.setCallback(NoOp.INSTANCE);
        if(ctr == null){
            return (T) enhancer.create();
        }else {
            return (T) enhancer.create(ctr.getParameterTypes(),args);
        }
    }
}
//使用反射来完成对象的依赖注入
public class ReflectionUtils {

    public static void injectField(Field field, Object obj, Object value) throws IllegalAccessException {
        if(field != null) {
            field.setAccessible(true);//访问私有属性
            field.set(obj, value);//设置 obj 的 field 为 value
        }
    }

}
```

3. bean工厂

```java
public interface BeanFactory {
    Object getBean(String name) throws Exception;
}
```
```java
public class BeanFactoryImpl implements BeanFactory {

    private static final ConcurrentHashMap<String,Object> beanMap = new ConcurrentHashMap<>();

    private static final ConcurrentHashMap<String, BeanDefinition> beanDefineMap= new ConcurrentHashMap<>();//存储的是对象的名称和对象对应的数据结构的映射

    private static final Set<String> beanNameSet = Collections.synchronizedSet(new HashSet<>());


    @Override
    public Object getBean(String name) throws Exception {

        //查找对象是否已经实例化过
        Object bean = beanMap.get(name);
        if(bean != null){
            return bean;
        }

        //如果没有实例化，那就需要调用createBean来创建对象
        bean = createBean(beanDefineMap.get(name));

        if(bean != null) {

            //对象创建成功以后，注入对象需要的参数
            populatebean(bean);

            //再把对象存入Map中方便下次使用。
            beanMap.put(name,bean);
        }

        //结束返回
        return bean;
    }

    private Object createBean(BeanDefinition beanDefinition) throws Exception {
        String beanName = beanDefinition.getClassName();
        Class clz = ClassUtils.loadClass(beanName);
        if(clz == null) {
            throw new Exception("can not find bean by beanName");
        }
        return BeanUtils.instanceByCglib(clz,null,null);
    }

    private void populatebean(Object bean) throws Exception {
        Field[] fields = bean.getClass().getSuperclass().getDeclaredFields();
        if (fields != null && fields.length > 0) {
            for (Field field : fields) {
                String beanName = field.getName();
                beanName = StringUtils.uncapitalize(beanName);
                if (beanNameSet.contains(field.getName())) {
                    Object fieldBean = getBean(beanName);
                    if (fieldBean != null) {
                        ReflectionUtils.injectField(field,bean,fieldBean);
                    }
                }
            }
        }
    }

    protected void registerBean(String name, BeanDefinition bd){
        beanDefineMap.put(name,bd);
        beanNameSet.add(name);
    }
}
```
首先我们看到在 BeanFactory 的实现中有两 HashMap，beanMap 和 beanDefineMap。 beanDefineMap 存储的是对象的名称和对象对应的数据结构的映射。beanMap 用于保存 beanName和实例化之后的对象。

容器初始化的时候，会调用 BeanFactoryImpl.registerBean 方法。把 对象的 BeanDefination 数据结构，存储起来。

当我们调用 getBean() 的方法的时候。会先到 beanMap 里面查找，有没有实例化好的对象。如果没有，就会去beanDefineMap查找这个对象对应的 BeanDefination。再利用DeanDefination去实例化一个对象。

对象实例化成功以后，我们还需要注入相应的参数，调用 populatebean()这个方法。在 populateBean 这个方法中，会扫描对象里面的Field，如果对象中的 Field 是我们IoC容器管理的对象，那就会调用ReflectionUtils.injectField来注入对象。

一切准备妥当之后，我们对象就完成了整个 IoC 流程。最后这个对象放入 beanMap 中,方便下一次使用。

所以我们可以知道 BeanFactory 是管理和生成对象的地方

4. 容器

我们所谓的容器，就是对BeanFactory的扩展，负责管理 BeanFactory。我们的这个IoC 框架使用 Json 作为配置文件，所以我们容器就命名为 JsonApplicationContext
```java
public class JsonApplicationContext extends BeanFactoryImpl {

    private String fileName;
    public JsonApplicationContext(String fileName) {
        this.fileName = fileName;
    }
    public void init() throws IOException {
        loadFile();
    }
    private void loadFile() throws IOException {

        InputStream inputStream = Thread.currentThread().getContextClassLoader().getResourceAsStream(fileName);
        String text = IOUtils.toString(inputStream,"utf8");
        List<BeanDefinition> beanDefinitions = JSONObject.parseArray(text,BeanDefinition.class);

        if(CollectionUtils.isNotEmpty(beanDefinitions)) {
            for (BeanDefinition beanDefinition : beanDefinitions) {
                registerBean(beanDefinition.getName(), beanDefinition);
            }
        }
    }
}
```
这个容器的作用就是 读取配置文件。将配置文件转换为容器能够理解的 BeanDefination。然后使用 registerBean 方法。注册这个对象。

至此，一个简单版的 IoC 框架就完成

5. 框架的使用
三个对象
```java
public class Hand {
    public void waveHand(){
        System.out.println("挥一挥手");
    }
}

public class Mouth {
    public void speak(){
        System.out.println("say hello world");
    }
}

public class Robot {
    //需要注入 hand 和 mouth
    private Hand hand;
    private Mouth mouth;

    public void show(){
        hand.waveHand();
        mouth.speak();
    }
}
```
配置文件
```json
[
  {
    "name":"robot",
    "className":"me.changjie.entity.Robot"
  },
  {
    "name":"hand",
    "className":"me.changjie.entity.Hand"
  },
  {
    "name":"mouth",
    "className":"me.changjie.entity.Mouth"
  }
]
```
测试类
```java
public class App {

    public static void main(String[] args) throws Exception {
        JsonApplicationContext applicationContext = new JsonApplicationContext("application.json");
        applicationContext.init();
        Robot aiRobot = (Robot) applicationContext.getBean("robot");
        aiRobot.show();
    }
}
```
输出
```java
挥一挥手
say hello world
```
可以看到成功为aiRobot 注入了 hand 和 mouth。

至此我们 Ioc 框架开发完成


### 原文链接
> [徒手撸框架--实现IoC](https://www.xilidou.com/2018/01/08/spring-ioc/)
> [源码](https://github.com/changchangjie/spring-study)



