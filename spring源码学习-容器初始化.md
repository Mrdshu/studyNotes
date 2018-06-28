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

### 3. ResourceLoader
`ResourcePatternResolver`接口用于加载多个`Resource`
```
public interface ResourceLoader {
	Resource getResource(String location);
	ClassLoader getClassLoader();
}

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

# 参数读取——PropertySource
