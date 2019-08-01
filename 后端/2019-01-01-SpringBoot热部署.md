# SpringBoot热部署

有2种实现方式：

1. 使用 Spring Loaded
2. 使用 spring-boot-devtools
3. 使用 Maven插件

## 一 Spring Loaded

需要下载 springloaded.jar，启动时在 VM arguments 中加入参数：

```
-javaagent:D:\springloaded-1.2.6.RELEASE.jar -noverify
```

## 二 spring-boot-devtools

pom.xml 中添加依赖

```xml
<dependencies>
	<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

## 三 maven的方式

```xml
<build>
	<plugins>
    	<plugin>
        	<groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        	<dependencies>
                <!-- 以 maven 方式启动 -->
            	<dependency>
                	<groupId>org.springframework</groupId>
                    <artifactId>springloaded</artifactId>
                    <version>1.2.6.RELEASE</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

# SpringBoot 发布方式

## 一 jar 包

在 pom.xml中`<packaging>jar</packaging>`，使用 maven install 打包成 jar 即可，再使用 java 命令启动项目：`java -jar xxx.jar`

## 二 war 包

### 1 修改打包方式

在 pom.xml中`<packaging>war</packaging>`

### 2 加入tomcat 依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat </artifactId>
</dependency>
```

### 3 修改启动类

```java
/**
 * 1 启动类继承 SpringBootServletInitializer
 * 2 重写 configure 方法
 */
@SpringBootApplication
public class DemoApplication extends SpringBootServletInitializer{
    
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder){
        return builder.sources(DemoApplication.class);
    }
    
    public static void main(String[] args){
        SpringApplication.run(DemoApplication.class, args);
    }
}
```


