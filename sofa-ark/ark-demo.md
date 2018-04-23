> [工程地址](https://github.com/alipay/sofa-ark/tree/master/sofa-ark-samples/sample-springboot-ark)

## 简介
该样例工程演示了如何借助 `Maven` 插件将一个 Spring Boot Web 工程打包成标准格式规范的可执行 Ark 包；

## 准备
因该样例工程依赖 [sample-ark-plugin](https://github.com/alipay/sofa-ark/tree/master/sofa-ark-samples/sample-ark-plugin) , 因此需要提前在本地安装该 `Ark Plugin` 

## 工具
官方提供了 `Maven` 插件 - `sofa-ark-maven-plugin` ，只需要简单的配置项，即可将 Spring Boot Web 工程打包成标准格式规范的可执行 Ark 包，插件坐标为：

```xml
<plugin>
    <groupId>com.alipay.sofa</groupId>
    <artifactId>sofa-ark-maven-plugin</artifactId>
    <version>0.2.0</version>
</plugin>
```

> [详细请参考插件使用文档](sofastack.github.io/docs/ark-jar.html)


## 入门
基于该样例工程，我们一步步描述如何将一个 Spring Boot Web 工程打包成可运行 Ark 包

### 创建 SpringBoot Web 工程
在官网 [https://start.spring.io/](https://start.spring.io/) 下载一个标准的 Spring Boot Web 工程

### 引入 sample-ark-plugin
在工程主 `pom.xml` 中如下配置，添加另一个样例工程打包生成的 `Ark Plugin` 依赖，[参考文档](sofastack.github.io/docs/ark-plugin-demo.html) 

```xml
<dependency>
     <groupId>com.alipay.sofa</groupId>
     <artifactId>sample-ark-plugin</artifactId>
     <classifier>ark-plugin</classifier>
     <version>0.2.0</version>
 </dependency>
```

### 配置打包插件
在工程主 `pom.xml` 中如下配置 `Maven` 插件 `sofa-ark-maven-plugin` :

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.alipay.sofa</groupId>
            <artifactId>sofa-ark-maven-plugin</artifactId>
            <executions>
                <execution>
                    <id>default-cli</id>
                    
                    <!--goal executed to generate executable-ark-jar -->
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                    
                    <configuration>
                        <!--specify destination where executable-ark-jar will be saved, default saved to ${project.build.directory}-->
                        <outputDirectory>./target</outputDirectory>
                        
                        <!--default none-->
                        <arkClassifier>executable-ark</arkClassifier>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

在该样例工程中，我们只配置了一部分配置项，这已经足够生成一个可用的可执行 `Ark` 包，各配置项含义如下：
* outputDirectory: `mvn package` 打包后，输出的 `Ark` 包文件存放目录；

* arkClassifier: 指定发布的 `Ark` 包其 `Maven` 坐标包含的 `classifier` 值，默认为空；

**关于 arkClassifier 配置项需要特别注意下，默认值为空；如果不指定 classifier ，上传到仓库的 Jar 包其实是一个可运行的 Ark 包；如果需要和普通的打包加以区分，需要配置该项值。**

### 打包、安装、发布
和普通的工程操作类似，使用 `mvn package` , `mvn install` , `mvn deploy` 即可完成插件包的安装和发布；

### 运行
我们提供了两种方式在 Ark 容器上启动工程应用，通过命令行启动或者在 IDE 启动；在 IDE 启动时，需要额外添加依赖；使用命令行启动非常简便，直接使用 `java -jar` 即可启动应用；下面我们说下如何在 IDE 启动 Ark 应用；

* Spring Boot 工程：Spring Boot 工程需要添加如下依赖即可：

```xml
<dependency>
    <groupId>com.alipay.sofa</groupId>
    <artifactId>sofa-ark-springboot-starter</artifactId>
    <version>0.2.0</version>
</dependency>
```

* 普通 Java 工程： 相较于 SpringBoot 工程，普通的 Java  工程需要添加另一个依赖：

```xml
<dependency>
    <groupId>com.alipay.sofa</groupId>
    <artifactId>sofa-ark-support-starter</artifactId>
    <version>0.2.0</version>
</dependency>
```

除此之外，还需要在工程 `main` 方法最开始处，执行容器启动，如下：

```java
public class Application{
    
    public static void main(String[] args) { 
        SofaArkBootstrap.launch(args);
        ...
    }
    
}
```

### 运行测试用例
SOFAArk 提供了 `org.junit.runner.Runner` 的两个实现类，`ArkJUnit4Runner` 和 `ArkBootRunner`，分别用于集成 JUnit4 测试框架和 Spring Test；

#### ArkJUnit4Runner
`ArkJUnit4Runner` 类似 `JUnit4`，使用注解 `ArkJUnit4Runner`，即可在 SOFAArk 容器之上运行普通的 JUnit4 测试用例；示范代码如下：

```java
@RunWith(ArkJUnit4Runner.class)
public class UnitTest {

    @Test
    public void test() {
        Assert.assertTrue(true);
    }

}
```

`ArkJUnit4Runner` 和 `JUnit4` 使用基本完全一致，`JUnit4` 测试框架的其他特性都能够完全兼容，

#### ArkBootRunner
`ArkBootRunner` 类似 `SpringRunner`，参考[文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-testing.html)学习 `SpringRunner` 的用法；为了能够在 SOFAArk 容器之上运行 Spring Boot 测试用例，只需要简单使用 `@RunWith(ArkBootRunner.class)` 替代 `@RunWith(SpringRunner.class)` 即可；示范代码如下：

```java
@RunWith(ArkBootRunner.class)
@SpringBootTest(classes = SpringbootDemoApplication.class)
public class IntegrationTest {

    @Autowired
    private SampleService sampleService;

    @Test
    public void test() {
        Assert.assertTrue("A Sample Service".equals(sampleService.service()));
    }

}
```

`ArkBootRunner` 和 `SpringRunner` 使用基本完全一致；