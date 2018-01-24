# Schizo

一个超屌的多进程框架, 正处于开发阶段


## 使用



1. 继承 `SchizoService`
2. `@Action` 注解用于 Service, 值指定为可以调起该 Service 的 action
3. `@Api` 注解用于改类内的方法

搞定, 其余的接口, 分进程处理逻辑全部是自动生成的.

自动生成的接口类为 `TestServiceApi` (Service 的类名 + Api)

##### 示例: 服务端 TestService (该模块开发者提供服务)

**TestService.java**

````
@Action("com.meizu.flyme.schizo.test1")
public class TestService extends SchizoService {

    @Api("person")
    Person getPerson(String name) {
        Log.i("SCHIZO", "api person accept request: name is " + name);
        return new Person("Hello", "Schizo");
    }

    @Api("book")
    Book getBook(String title) {
        return new Book(title, "Nobody");
    }


    @Api("book1")
    Book getBook(Person person) {
        Log.i("SCHIZO", "Person is [" + person.name + ",,," + person.surname + "]");
        return new Book(person.name, "Nobody");
    }
}
````

* 每一个 `@Api` 注解的方法, 对应于一个接口
* 同一个接口的不同版本, Api 改个名字就可以兼容

**AndroidManifest.xml**

````
    <service
        android:name=".service.TestService"
        android:exported="false"
        android:process=":net" >
        <intent-filter>
            <action android:name="com.meizu.flyme.schizo.test1" />
        </intent-filter>
    </service>
````

* `android:process=":net"`: 指定进程名
* `<action android:name="com.meizu.flyme.schizo.test1" />`: 指定 Action, `@Action` 通过这个来定位接口类.

#### 示例(自动生成): 客户端 TestServiceApi (框架自动生成如下客户端调用接口)

````
public final class TestServiceApi {
  private static final String ACTION = "com.meizu.flyme.schizo.test1";

  public static void attach(Context context) {
    ComponentManager.attach(context, ACTION);
  }

  public static void detach() {
    ComponentManager.detach(ACTION);
  }

  public static Single<Person> person(String name) {
    return ComponentManager.get(ACTION).process("person", name, Person.class);
  }

  public static Single<Book> book(String title) {
    return ComponentManager.get(ACTION).process("book", title, Book.class);
  }

  public static Single<Book> book1(Person person) {
    return ComponentManager.get(ACTION).process("book1", person, Book.class);
  }
}
````

* `attach(Context)`: 引入该进程模块
* `detach()`: 释放该进程模块
* `person(String)`, `book(String)`, `book1(Person)`: 根据 `@Api` 自动生成的调用接口, 供客户端调用

## 任务拆解

+ 协议与协议: 请求(bean) -> 协议(json) -> 传输(aidl, string) -> 协议(json) -> 响应(bean)
    + 传输层可定制, 未来改成 byte array 或者 byte string
    + 协议层可定制, 当前使用 json
    + 请求与协议可定制, 当前使用 gson
+ 服务: 分进程服务最小单位(SchizoService)
    + 封装 aidl
    + 分发和管理请求
    + 接口注入
    + 多线程处理(待实现)
+ 客户: 使用者可以是任何模块
    + 组件:客户端的 aidl 封装, RxJava 封装
    + 组件管理: 根据 action + api + parameters 定位组件, 沟通服务

+ 编译
    + 根据服务器实现的接口注入(API), 自动生成客户端调用代码(待实现, 目前手动, 见 sample 模块示例)
    + 自动处理依赖关系(待实现)
