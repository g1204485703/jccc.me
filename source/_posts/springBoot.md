# Spring Boot 

## 新建Spring Boot 项目

<!--more-->

1. ### File - New - Project

   ![newProject](/imag/new project.png)

2. ### 选择 Spring Initializer，使用 Spring 提供的快速构建 Service 

   https://start.srping.io 

   ![projectinit01](/imag/project init 1.png)

3. ### 填写相应项目信息 

   Group：组织名称
   Artifact：项目名称
   Version：版本
   Name：项目名![project init 2](/imag/project init 2.png)

4. ### 勾选所需的依赖

   ![1548657001676](/imag/1548661606578.png)

5. ### 选择本地存储位置

   ![1548657030708](/imag/1548657030708.png)

6. ### 完成，maven会自动加载依赖，完成项目构建

   ![1548657751751](/imag/1548657751751-1551082094718.png)



## 第一个Spring Boot demo

### 简单 Controller

1. #### 新建 HelloController

   ```java
   @RestController
   public class HelloController {
       
       @RequestMapping(value = "/hello",method = RequestMethod.GET)
       public String Hello(){
           return "hello";
       }
   }
   ```

   

2. #### 选择 DemoApplication，启动项目，等待项目在8080端口完成启动

   ![1548657810384](/imag/1548657810384.png)

3. #### 使用浏览器访问 localhost:8080/hello，成功返回 “Hello Spring Boot”

   ![1548657864829](/imag/1548657864829.png)

4. ##### 

### 连接数据库

1. #### 将默认的 resources - application.properties 配置文件重命名为 application.yaml

   两种后缀均为配置文件，yaml 文件相比 properties 更为简洁直观，故采用 yaml 类型的配置文件

2. #### 添加 Mysql 数据库配置

   ```yaml
   spring:
     datasource:
       driver-class-name: com.mysql.cj.jdbc.Driver
       url: jdbc:mysql://gjicong.asuscomm.com:3306/school_mutual?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT
       username: root
       password: 123456
   ```

   配置数据库连接相关配置，

3. ![icons8-globe-64](/imag/icons8-globe-64.png)

4. 

![icons8-globe-64](imag/icons8-globe-64.png)
