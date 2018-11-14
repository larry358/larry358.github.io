---
title: 用Spring构建REST服务
categories:
- General
excerpt: |
  学习如何简单地用Spring构建RESTful服务
feature_text: |
  REST已经迅速成为构建web服务的事实标准，因为这些服务易于构建和消化。
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

原文链接：[Building REST services with Spring](http://spring.io/guides/tutorials/rest/)

# 用Spring构建REST服务

&nbsp;&nbsp;&nbsp;&nbsp;REST已经迅速成为构建web服务的事实标准，因为这些服务易于构建和消化。

&nbsp;&nbsp;&nbsp;&nbsp;对于REST如何适用于这个微服务的世界有更加广泛的讨论，但是————在这个教程里————我们仅仅关注于构建RESTful的服务。

&nbsp;&nbsp;&nbsp;&nbsp;为什么要用REST？REST拥抱了Web的诸多规程，包括架构，益处和其余各个方面。由于它的作者罗伊·菲尔丁(Roy Fielding)参与到了许多管理Web运作的规范当中，这点并不令人感到惊讶。

&nbsp;&nbsp;&nbsp;&nbsp;有什么好处？Web及其核心协议HTTP，提供了一系列特性：

- 合适的行为(`GET`,`POST`,`PUT`,`DELETE`,...)
- 缓存
- 重定向和转发
- 安全（加密和认证）

&nbsp;&nbsp;&nbsp;&nbsp;这些都是构建可快速恢复的服务中的关键因素。然而这并非全部。Web是由许多小型规范构建的，因此它容易变革，而不至于被“标准战争”所阻碍。

&nbsp;&nbsp;&nbsp;&nbsp;开发者可以调用实现了各种各样的规范的第三方工具包，从而简单且方便地同时获得客户端和服务器的技术。

&nbsp;&nbsp;&nbsp;&nbsp;因此，在HTTP之上构建的RESTful API提供了构建灵活的API的方法，它们能够：

- 提供后端兼容性
- 可演进的API
- 可伸缩的服务
- 能保证安全的服务
- 广泛的服务，从声明式到非声明式

&nbsp;&nbsp;&nbsp;&nbsp;重要的是，应当意识到，尽管无处不在，但REST并非一项标准，*本质上*，它更是一种方法，一种风格，一系列帮助你构建Web级系统的，作用在体系上的*约束*。在这篇教程中，我们将采用Spring产品组合构建一个RESTful服务，来一窥REST不计其数的特性。

## 入门

&nbsp;&nbsp;&nbsp;&nbsp;在本教程的整个过程中，我们将采用[Spring Boot](https://spring.io/projects/spring-boot)。前往[Spring Initializr](https://start.spring.io/)并选择以下内容：

- Web
- JPA
- H2
- Lombok

&nbsp;&nbsp;&nbsp;&nbsp;之后选择"Generate Project"（生成项目）。将会下载一个`.zip`文件。解压缩之。你会在里面找到一个包含`pom.xml`构建文件的，基于Maven的简单项目。（注：你*可以*使用Gradle。本教程中的示例将会基于Maven。）

&nbsp;&nbsp;&nbsp;&nbsp;Spring Boot可以运用于任何IDE。你可以用Eclipse，IntelliJ IDEA，Netbeans等。[The Spring Tool Suite](https://spring.io/tools/)是一个开源的，基于Eclipse的IDE发行版本，它提供了Eclipse的Java EE发行版本的一个超集。它包含使Spring应用工作更容易的特性。它并不是必须的，但如果你想让你的按键获得额外的**特质**的话，还是考虑使用它吧。这里有一段演示如何上手STS和Spring Boot的视频，它是使你熟悉这些工具的一个总的介绍。

&nbsp;&nbsp;&nbsp;&nbsp;如果你采用IntelliJ IDEA作为你在本教程中使用的IDE的话，你需要安装lombok插件。为了解我们如何向IntelliJ IDEA安装插件，请参考[managing-plugins](https://www.jetbrains.com/help/idea/managing-plugins.html)。之后你需要确保Preferences → Compiler → Annotation Processors中的"Enable annotation processing"复选框被选中，正如这个页面描述的那样：https://stackoverflow.com/questions/14866765/building-with-lomboks-slf4j-and-intellij-cannot-find-symbol-log

## 故事到目前为止...

&nbsp;&nbsp;&nbsp;&nbsp;就让我们从能构建的最简单的东西开始吧。实际上，为了让它尽可能地简单，我们甚至可以暂时忘掉REST的概念。（稍后，我们将添加REST，以理解它所带来的不同。）

&nbsp;&nbsp;&nbsp;&nbsp;我们的例子构造了一个用来管理一家公司的雇员的简单薪资服务模型。简单存入过后，你需要在一个叫H2的内存中数据库储存雇员对象，并且通过JPA访问它们。这会被Spring MVC层封装，以便从远程访问。

`nonrest/src/man/java/payroll/Employee.java`

```java
package payroll;

import lombok.Data;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Data
@Entity
class Employee {

    private @Id @GeneratedValue Long id;
    private String name;
    private String role;

    Employee(String name, String role) {
        this.name = name;
        this.role = role;
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;尽管这个Java类规模小，它却包含许多东西：

- `@Data`是一个Lombok注解，用来根据对象的字段创建所有getter和setter，以及`equals`，`hash`和`toString`方法。
- `@Entity`是一个JPA注解，用来使该类的对象能随时储存在基于JPA的数据存储里。
- `id`，`name`和`role`是我们的对象范围内的属性，其中第一个属性被更多的JPA注解标记，以表明它是主键，且会被JPA提供者自动填入。
- 一个自定义的构造器，它为满足我们创建新对象而定义，但它目前还没有id。

&nbsp;&nbsp;&nbsp;&nbsp;有了这个我们领域内的对象定义，我们现在就能借助[Spring Data JPA](https://spring.io/guides/gs/accessing-data-jpa/)来处理枯燥的数据库交互。Spring Data资源库是一组拥有支持对后端数据存储进行读取、更新、删除、建立数据记录的方法的接口。一些资源库还支持数据分页以及用合适的方法排序。Spring Data基于接口中方法命名所遵循的约定来合成实现。

> 有多个JPA以外的资源库实现。你可以使用Spring Data MongoDB, Spring Data GemFire, Spring Data Cassandra等。在本教程中，我们将继续使用JPA。
> 
`nonrest/src/main/java/payroll/EmployeeRepository.java`

```java
package payroll;

import org.springframework.data.jpa.repository.JpaRepository;

interface EmployeeRepository extends JpaRepository<Employee, Long> {

}
```

&nbsp;&nbsp;&nbsp;&nbsp;这个接口继承了Spring Data JPA的`JpaRepository`，指明了对象领域类`Employee`和id类`Long`。尽管该接口看上去空空如也，但它拥有支持以下功能的强大能力：

- 创建新实例
- 更新已有实例
- 删除
- 查找（根据简单或复杂的属性查找一个和全部）

&nbsp;&nbsp;&nbsp;&nbsp;Spring Data的[资源库解决方案](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories)使得它可以绕开数据存储的具体细节。取而代之的是，它采用用于特定领域的用语解决大多数问题。

&nbsp;&nbsp;&nbsp;&nbsp;信不信由你，这已经足够让一个应用运行了！一个最小的Spring Boot应用是一个`public static void main`入口和`@SpringBootApplication`注解。它告诉Spring Boot在可以帮助的地方给予帮助。

`nonrest/src/main/java/payroll/PayrollApplication.java`

```java
package payroll;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class PayrollApplication {

    public static void main(String... args) {
        SpringApplication.run(PayrollApplication.class, args);
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;`@SpringBootApplication`是一个meta注解，它集成了**组件扫描**，**自动配置**和**属性支持**。在本教程中，我们不会深入Spring Boot的细节，但大体上，它会启动一个Servlet容器，运作我们的服务。

尽管如此，一个没有数据的应用并不是很有趣的，所以让我们预加载它。下面这个类将会被Spring自动加载：

`nonrest/src/main/java/payroll/LoadDatabase.java`

```java
package payroll;

import org.springframework.boot.CommandLineRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.Configuration;

@Configuration
@Slf4j
class LoadDatabase {

    @Bean
    CommandLineRunner initDatabase(EmployeeRepository repository) {
        return args -> {
            log.info("Preloading " + repository.save(new Employee("Bilbo Baggins", "burglar")));
            log.info("Preloading " + repository.save(new Employee("Frodo Baggins", "thief")));
        };
    }
}
```

当它被加载的时候，发生了什么？

- 一旦应用上下文被加载，Spring Boot就会运行所有的`CommandLineRunner`Bean。
- 这个runner将会请求一份你刚刚创建的`EmployeeRepository`的复制。
- 它将会用这个复制来创建两个实体，并保存它们。
- `@Slf4j`是一个用来自动创建一个命名为`log`的基于Slf4j的`LoggerFactory`的Lombok注解，它让我们用日志记录这些新创建的“雇员”。

鼠标右键点击并**运行**`PayRollApplication`，你会得到如下输出：

```
...
2018-08-09 11:36:26.169  INFO 74611 --- [main] payroll.LoadDatabase : Preloading Employee(id=1, name=Bilbo Baggins, role=burglar)
2018-08-09 11:36:26.174  INFO 74611 --- [main] payroll.LoadDatabase : Preloading Employee(id=2, name=Frodo Baggins, role=thief)
...
```

这不是全部的日志，而是有关预加载数据的关键部分。（说真的，看一下控制台的全部内容吧，很令人感到光荣的。）

## HTTP是平台

为了用Web层封装你的资源库，你必须求助于Spring MVC。多亏了Spring Boot，我们几乎不需要为基础设施编写代码。相反，我们可以关注行为：

`nonrest/src/main/java/payroll/EmployeeController.java`

```java
package payroll;

import java.util.List;

import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
class EmployeeController {

    private final EmployeeRepository repository;

    EmployeeController(EmployeeRepository repository) {
        this.repository = repository;
    }

    // 总的根部

    @GetMapping("/employees")
    List<Employee> all() {
        return repository.findAll();
    }

    @PostMapping("employees")
    Employee newEmployee(@RequestBody Employee newEmployee) {
        return repository.save(newEmployee);
    }

    // 单个条目

    @GetMapping("/employees/{id}")
    Employee one(@PathVariable Long id) {
        
        return repository.findById(id)
            .orElseThrow(() -> new EmployeeNotFoundException(id));
    }

    @PutMapping("/employees/{id}")
    Employee replaceEmployee(@RequestBody Employee newEmployee, @PathVariable Long id) {

        return repository.findById(id)
            .map(employee -> {
                employee.setName(newEmployee.getName());
                employee.setRole(newEmployee.getRole());
                return repository.save(employee);
            })
            .orElseGet(() -> {
                newEmployee.setId(id);
                return repository.save(employee);
            });
    }

    @DeleteMapping("/employees/{id}")
    void deleteEmployee(@PathVariable Long id) {
        repository.deleteById(id);
    }
}
```

- `@RestController`指明每个方法返回的数据都会被直接写入响应体，而不是渲染模板。
- 一个`EmployeeRepository`被构造器注入到控制器里。
- 对每一种操作，我们都有对应的路由（`@GetMapping`，`@PostMapping`，`@PutMapping`和`@DeleteMapping`，与HTTP的`GET`，`POST`，`PUT`和`DELETE`调用一一对应）。（注：阅读每个方法并理解它们做了什么，这对你是有帮助的。）
- `EmployeeNotFoundException`是一个用来表明一名雇员被查找却没有查找到的时候的异常。

`nonrest/src/main/java/payroll/EmployeeNotFoundException.java`

```java
package payroll;

class EmployeeNotFoundException extends RuntimeException {

    EmployeeNotFoundException(Long id) {
        super("Could not find employee " + id);
    }
}
```

当一个`EmployeeNotFoundException`被抛出时，Spring MVC配置里的这点额外的小东西会被用来渲染一个**HTTP 404**：

`nonrest/src/main/java/payroll/EmployeeNotFoundAdvice.java`

```java
package payroll;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;

@ControllerAdvice
class EmployeeNotFoundAdvice {

    @ResponseBody
    @ExceptionHandler(EmployeeNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    String employeeNotFoundHandler(EmployeeNotFoundException ex) {
        return ex.getMessage();
    }
}
```

- `@ResponseBody`标志着这个Advice类会被直接渲染进响应体。
- `@ExceptionHandler`将该Advice类配置为仅当一个`EmployeeNotFoundException`被抛出时才响应。
- `@ResponseStatus`说明一个`HttpStatus.NOT_FOUND`问题，例如**HTTP 404**。
- Advice类的方法体生成内容。对于该类的情况而言，它给出这个异常的消息。

要启动这个应用，可以用鼠标右键点击`PayRollApplication`里的`public static void main`并且从你的IDE中选择**Run**，或者：

Spring Initializr使用maven封装器，所以输入以下内容：

```shell
$ ./mvnw clean spring-boot:run
```

用你安装的maven版本的话，输入以下内容：

```shell
$ mvn clean spring-boot:run
```

在这个应用启动后，我们可以立即对其发起查询请求。

```shell
$ curl -v localhost:8080/employees
```

这将会产生下列输出：

```
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> GET /employees HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 200
< Content-Type: application/json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Thu, 09 Aug 2018 17:58:00 GMT
<
* Connection #0 to host localhost left intact
[{"id":1,"name":"Bilbo Baggins","role":"burglar"},{"id":2,"name":"Frodo Baggins","role":"thief"}]
```

这里你可以看到以紧密格式呈现的预加载的数据。

如果你试着查询一个不存在的用户的话······

```shell
$ curl -v localhost:8080/employees/99
```

你会得到······

```
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> GET /employees/99 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 404
< Content-Type: text/plain;charset=UTF-8
< Content-Length: 26
< Date: Thu, 09 Aug 2018 18:00:56 GMT
<
* Connection #0 to host localhost left intact
Could not find employee 99
```

这条消息用自定义的**Could not find employee 99**漂亮地展示了一个**HTTP 404**错误。

展示现时编码的交互并不困难······

```shell
$ curl -X POST localhost:8080/employees -H 'Content-Type:application/json' -d '{"name": "Samwise Gamgee", "role": "gardener"}'
```

创建一个新的`Employee`记录，然后把内容返回给我们：

```json
{"id":3,"name":"Samwise Gamgee","role":"gradener"}
```

你可以改动这个用户：

```shell
$ curl -X PUT localhost:8080/employees/3 -H 'Content-type:application/json' -d '{"name": "Samwise Gamgee", "role": "ring bearer"}'
```

更新这个用户：

```json
{"id":3,"name":"Samwise Gamgee","role":"ring bearer"}
```

> 根据你构建服务的方法，可能会出现明显的变化。在这个场景里，**替换**是比**更新**更好的描述。例如，如果没有提供名字的话，取而代之的是，它将会被留空。

并且你可以删除······

```shell
$ curl -X DELETE localhost:8080/employees/3
$ curl localhost:8080/employees/3
Could not find employee 3
```

这很好，没什么问题。但是我们已经有RESTful服务了吗？（如果你没有掌握提示的话，答案是否定的。）

到底缺了些什么呢？

## 是什么让一个东西变得RESTful？

到目前为止，你已经有了一个基于Web的，用来处理与雇员数据有关的核心操作的服务。但是这并不足以让东西变得“RESTful”。

- 美观的URL，如/employees/3，并不是REST。
- 仅仅使用`GET`，`POST`等并不是REST。
- 把所有增删改查操作摆开来并不是REST。

实际上，到现在我们已经构建的东西，用**RPC**（**远程过程调用**）描述会更好。这是因为没有办法知道如何与这个服务进行交互。如果你今天发布了它，你还需要为所有详情写一份文档，或者在某处主持一个开发者门户。

罗伊·菲尔丁的这段声明可能更进一步地揭开**REST**和**RPC**的不同之谜：

```
将任何基于HTTP的接口称作REST API的人数正令我感到沮丧。当前的例子是SocialSite REST API。那是RPC。它拼命叫喊自己是RPC。展示出来的东西中有如此多的耦合，以至于它应当被评为X级。

为了在“超文本是约束”这一观念下构建清晰的REST架构风格所需要做的事情是什么呢？换句话说，如果应用状态的引擎（因此就是API）不是由超文本驱动的，那么它就不是RESTful的，不能成为REST API。就是这样。有没有哪个地方有一些有缺陷的指南需要被修复？
```

不将超媒体包括在我们展示的内容中的副作用就是客户端必须硬编码URI以导向API。这导致了早在电子商务于Web崛起之前就有的同样脆弱的天性。这是我们的JSON输出需要伊甸小小帮助的信号。

这里引入[Spring HATEOAS](https://spring.io/projects/spring-hateoas)，一个以帮助你编写超媒体驱动的输出为目标的Spring项目。为将你的服务升级成RESTful服务，在你的构建中加入以下内容：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

这个轻量级库给予我们用来定义一个RESTful服务，然后将它渲染为可被客户端接受和处理的格式的结构。

任何RESTful服务的关键材料是向相关的操作中添加[链接](https://tools.ietf.org/html/rfc5988)。为了让你的控制器更加地RESTful，像这样添加链接：

```java
@GetMapping("/employees/{id}")
Resource<Employee> one(@PathVariable Long id) {

    Employee employee = repository.findById(id)
            .orElseThrow(() -> new EmployeeNotFoundException(id));

    return new Resource<>(employee,
            linkTo(methodOn(EmployeeController.class).one(id)).withSelfRel(),
            linkTo(methodOn(EmployeeController.class).all()).withRel("employees"));
}
```

这和我们之前有过的这一方法很类似，但有几处发生了变化：

- 方法的返回类型从`Employee`变成了`Resource<Employee>`。`Resource<T>`是来自Spring HATEOAS的一个通用容器，它不仅包括数据，还包括一个链接的集合。
- `linkTo(methonOn(EmployeeController.class).one(id)).withSelfRel()`要求Spring HATEOAS建立一个指向`EmployeeController`的`one()`方法的链接，并且将它标记为[自身](https://www.iana.org/assignments/link-relations/link-relations.xhtml)的链接。
- `linkTo(methodOn(EmployeeController.class).all()).withRel("employees")`要求Spring HATEOAS建立一个指向总的根部的方法`all()`的链接，并且将它称为“employees”。


“建立一个链接”意味着什么？Spring HATEOAS的其中一个核心类是`Link`，它包括一个**URI**和一个**rel**（关系）。链接使Web更具力量。在万维网诞生之前，其余的文档系统会渲染信息或链接，但正是具有数据的文档之间的链接把Web缝合了起来。

罗伊·菲尔丁鼓励采用使Web取得成功的这些相同技术来构建API，二链接就是其中之一。

如果你重新启动这个应用并且查询**Bilbo**的雇员记录，那么你将会得到与早先略有不同的响应信息：

<span style="color:blue;">一个单一雇员的RESTful展示</span>
```json
{
    "id": 1,
    "name": "Bilbo Baggins",
    "role": "burglar",
    "_links": {
        "self": {
            "href": "http://localhost:8080/employees/1"
        },
        "employees": {
            "href": "http://localhost:8080/employees"
        }
    }
}
```

这一松散化的输出不仅展示了你之前见到过的数据元素（`id`，`name`和`role`），而且展示了一个包含两个URI的`_links`条目。整个文档是使用[HAL](http://stateless.co/hal_specification.html)格式化的。

HAL是一种轻量级的[媒体类型](https://tools.ietf.org/html/draft-kelly-json-hal-08)，它不仅能编码数据，还能编码超媒体控制，以向消费者提示他们能导航到的API的其余部分。就这里的情况而言，有一个“自身”链接（有点像代码中的`this`声明），伴随它的是一个回到总的根部的链接。

为了使总的根部也变得更加RESTful，你会想到在以下代码里包括顶级的链接和任意RESTful组件：

```java
@GetMapping("/employees") {
Resources<Resource<Employee>> all() {

    List<Resource<Employee>> employees = repository.findAll().stream()
        .map(employee -> new Resource<>(employee,
            linkTo(methodOn(EmployeeController.class).one(employee.getId())).withSelfRel(),
            linkTo(methodOn(EmployeeController.class).all()).withRel("employees")))
        collect.(Collectors.toList());

    return new Resources<>(employees,
        linkTo(methodOn(EmployeeController.class).all()).withSelfRel());
}
```

哇哦！那个刚才还只是`repository.findAll()`的方法，已经变得这么大了！让我们拆开它。

`Resources<>`是以封装集合为目标的另一个Spring HATEOAS容器。它也让你在其中包括链接。不要略过那第一句话。“封装集合”是什么时候？雇员的集合吗？

不完全正确。

既然我们在谈论REST，它就应当封装**雇员资源**的集合。

那就是你获取所有雇员的对象，却在之后将它们转化为一个`Resource<Employee>`对象的列表的原因。（多亏了Java 8 Stream API！）

如果你重新启动应用，并且获取根节点的信息，那么你就会看到如下面所示的信息。

<span style="color:blue;">一个单一雇员的RESTful展示</span>
```json
{
  "_embedded": {
    "employeeList": [
      {
        "id": 1,
        "name": "Bilbo Baggins",
        "role": "burglar",
        "_links": {
          "self": {
            "href": "http://localhost:8080/employees/1"
          },
          "employees": {
            "href": "http://localhost:8080/employees"
          }
        }
      },
      {
        "id": 2,
        "name": "Frodo Baggins",
        "role": "thief",
        "_links": {
          "self": {
            "href": "http://localhost:8080/employees/2"
          },
          "employees": {
            "href": "http://localhost:8080/employees"
          }
        }
      }
    ]
  },
  "_links": {
    "self": {
      "href": "http://localhost:8080/employees"
    }
  }
}
```

对提供了雇员资源的根节点来说，有一个顶层的“self”链接。“collection”是在“_embedded”部分下列出来的。这就是HAL展示集合的方式。

并且，集合中的每个成员个体都拥有各自的信息以及相关的链接。

添加这些链接的意义何在？它使不断地改进REST服务成为可能。既能维护现有的链接，又能在将来添加新的连接。较新的客户端能享受新链接的好处，老旧的客户端则能在旧链接上维持它们。这在重新定位和迁移服务时尤其有用。只要维持原有的链接结构，客户端就仍能找到想要的东西，并且与之交互。