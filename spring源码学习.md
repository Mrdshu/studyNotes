# 抽象资源——Resource
### 1. Resource
spring 配置文件及其他资源(file、url、classpath)都被封装成了 `Resource`对象。
```
public interface Resource extends InputStreamSource {  
       boolean exists();  
       boolean isReadable();  
       boolean isOpen();  
       URL getURL() throws IOException;  
       URI getURI() throws IOException;  
       File getFile() throws IOException;  
       long contentLength() throws IOException;  
       long lastModified() throws IOException;  
       Resource createRelative(String relativePath) throws IOException;  
       String getFilename();  
       String getDescription();  
}  
```

### 2. ResourceLoader
`ResourceLoader`可看作`Resource`的工厂类，接收传入的资源位置`location`，返回`Resource`
```
public interface ResourceLoader {
	Resource getResource(String location);
	ClassLoader getClassLoader();
}
```

### 3. ResourcePatternResolver
`ResourcePatternResolver`可看作`ResourceLoader`的扩展，用于加载多个`Resource`
```
public interface ResourcePatternResolver extends ResourceLoader {
        //加载多个Resource  
	Resource[] getResources(String locationPattern) throws IOException;
}
```

### 4. 示例
```
public static void main(String[] args) throws Exception {
    //获取文件应该存放的路径
    System.out.println(ClassPathResource.class.getResource("/"));

    ResourcePatternResolver resourcePatternResolver = new PathMatchingResourcePatternResolver();

    //通过默认的ResourceLoader工厂类，获取Resource
    Resource[] resources = resourcePatternResolver.getResources("/a.txt");
    for (Resource resource : resources) {
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(resource.getInputStream()));

        System.out.println(bufferedReader.readLine());
    }
}
```

# Bean工厂——BeanFactory
### 1. BeanDefinition
描述Bean的实例，包含一个bean实例的属性值、构造方法参数等参数

### 2. BeanDefinitionHolder
持有一个BeanDefinition、名称、别名数组。在Spring内部，它用来临时保存BeanDefinition来传递BeanDefinition

### 3. BeanDefinitionRegistry
持有BeanDefinition集合，并可新增（register）或删除beanDefinition
```
public interface BeanDefinitionRegistry extends AliasRegistry {
    //注册一个BeanDefinition  
    void registerBeanDefinition(String beanName, BeanDefinition beanDefinition) throws BeanDefinitionStoreException;
    //根据name,从自己持有的多个BeanDefinition 中 移除一个
    void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;
    // 获取某个BeanDefinition
    BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;
    boolean containsBeanDefinition(String beanName);//是否包含
    String[] getBeanDefinitionNames();//获取所有名称
    int getBeanDefinitionCount();//获取持有的BeanDefinition数量
    boolean isBeanNameInUse(String beanName); //判断某个BeanDefinition是否在使用
}
```

### 4. BeanDefinitionReader
`BeanDefinitionRegistry`增加`BeanDefinition`时，只能针对已经封装好的`BeanDefinition`对象：`registerBeanDefinition(String beanName, BeanDefinition beanDefinition)`

`BeanDefinitionReader`可解析配置文件，将配置中的bean转换为beanDefinition并将其注册到BeanDefinitionRegistry中。

可以说BeanDefinitionReader帮助BeanDefinitionRegistry实现了高效、方便的注册BeanDefinition。

### 5. BeanFactory
容器核心
# 事件机制
spring中的事件机制，观察者模式的一种实现。
### 1. ApplicationEvent
事件，改变源本身
### 2. ApplicationEventPublisher
发布事件，调用广播发布
### 3. ApplicationEventMulticaster
发布事件的广播，持有观察者
### 4. ApplicationListener
观察者，接收对应事件后，执行逻辑
