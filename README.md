# Spring Boot Docker 部署
## 一、创建Spring boot项目
### 1.依赖spring-boot-starter-web
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
### 2.创建Controller
```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String helloWorld(){
        return "Hello Spring boot";
    }
}
```
## 二、使用Docker镜像构建插件
### 1.依赖dockerfile-maven-plugin插件
```xml
<properties>
    <docker.image.prefix>ruihang</docker.image.prefix>
</properties>
```
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>

        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
            <version>1.3.6</version>
            <configuration>
                <repository>${docker.image.prefix}/${project.artifactId}</repository>
                <buildArgs>
                    <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                </buildArgs>
            </configuration>
        </plugin>
    </plugins>
</build>
```
其中`spring-boot-maven-plugin`是Spring boot生成项目时自带的

### 2.编写Dockerfile文件
在项目根目录下创建`Dockerfile`文件
```dockerfile
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG JAR_FILE
ADD ${JAR_FILE} app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
其中`JAR_FILE`参数由插件的配置参数设置。

## 三、安装Docker
### 1.Linux的安装
Linux的不多说，很多文档
### 2.Windows的安装
Docker for Windows的官方页面：
[https://store.docker.com/editions/community/docker-ce-desktop-windows](https://store.docker.com/editions/community/docker-ce-desktop-windows)
安装包下载地址：
[https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe](https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe)

安装说明：
如果Windows10系统，最低是专业版（对！家庭版不能使用），因为Docker需要在Windows下使用Hype-f。
其他安装步骤按照即可，中途会注销和重启计算机。

安装完成后：
1. 右击托盘的Docker图标
2. 选择"Settings..."
3. 勾选"Expose daemon on tcp://localhost:2357 without TLS"

## 四、开始构建Docker镜像
### 1.执行dockerfile:build
```
mvn install dockerfile:build
```
### 2.查看镜像
```
docker images
```
可以看到新增了一个镜像
```
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
ruihang/springboot-docker   latest              679603d30222        16 minutes ago      116MB
```

### 3.运行测试
```
docker run --rm -p 8080:8080 ruihang/springboot-docker
```
访问`http://localhost:8080/hello`，如有响应结果就成功了。