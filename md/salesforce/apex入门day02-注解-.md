

# apex入门第二天

## 常用注解

### @isTest

被标记的类为测试类,被标记的方法为测试方法

OnInstall=true 在打包期间会执行此测试方法

SeeAllData=true 可以访问组织中所有预先定义好的数据

isParallel=true 使测试方法可以并行执行

SeAllData和isParallel不能在同一方法上同时使用.



### @AuraEnabled

此注解允许客户端和服务端访问Apex控制器方法.被此注解修饰的方法对Lightning组件包括Lightning web组件和Aura组件可用.

在api 44.0+版本中,可以使用@AuraEnabled(cache=true)在客户机上缓存方法结果.



### @deprecated

被此注解标注的方法为被废弃.



### @future

被此方法修饰的方法为异步方法,@future方法之间不能相互调用. 构造,setMethodName,getMethodName方法不能使用@future注解.注意方法为**static**,**无返回值**,**参数必须是原始数据类型**

要为外部服务或API创建Web服务标注，可以使用标记为（callout = true）的未来方法创建Apex类



### @InvocableMethod

使用InvocableMethod注解来标识可以作为可调用操作运行的方法.

属性: label 默认为方法名,description 为方法的描述,默认为 null.

只能应用在static 方法上,不能与其他注解同时使用.类中只能有一个方法标注.类不能为内部类.

方法参数最多只能有一个



### @InvocableVariable 

InvocableVariable注释使用Invocablevariable注释来标识自定义类中可调用方法使用的变量。Invocablevariable注释标识了一个类变量，该变量用作Invocab1eMethod方法的可调用操作的输入或输出参数。如果您创建自己的自定义类来用作可调用方法的输入或输出，则可以注释各个类成员变量，使它们对方法可用。下面的代码示例显示了一个带有可调用变量的可调用方法。



### @namespaceAccessible 

 @namespaceAccessible使包中的公共Apex可用于使用相同名称空间的其他包。如果没有此注释，则2GP软件包中定义的Apex类，方法，接口和属性将无法与共享名称空间的其他软件包访问。声明为全局的Apex在所有命名空间中始终可用，并且不需要注释。



### @ReadOnly

只读的注释@Readonly注释允许您对Lightning平台数据库执行不受限制的查询。所有其他限制仍然适用。需要注意的是，该注释在删除请求返回行数的限制时，会阻止您在请求中执行以下操作:DML操作、对系统的调用。调度，调用带有@future注释的方法，以及发送电子邮件。



### @RemoteAction

RemoteAction注解支持通过JavaScript调用Visualforce中使用的Apex方法.此过程通常称为JavaScript远程处理.

````apex
@RemoteAction
global static String getItemId(String objectName) { ... }

````



### @SuppressWarnings

SuppressWarnings注释此注释在Apex中不做任何事，但可用于向第三方工具提供信息。@suppressWarnings注释在Apex中什么也不做，但是可以用来向第三方工具提供信息。



### @testSetup

TestSetup注释使用@testSetup注释定义的方法用于创建对类中的所有测试方法可用的公共测试记录。语法测试设置方法在测试类中定义，不接受参数，也不返回值。



### @TestVisible

TestVisible注释使用TestVisible注释允许测试方法访问测试类之外的另一个类的私有成员或受保护成员。

这些成员包括方法、成员变量和内部类。此注释为仅运行测试启用了更宽松的访问级别。

如果被非测试类访问，该注释不会改变成员的可见性。

使用这个注释，您不必将方法和成员变量的访问修饰符更改为public，您可以在测试方法中访问它们。

例如，如果一个私有成员变量不应该暴露给外部类，但是它应该可以被一个测试方法访问，那么您可以向这个变量

添加TestVisible注释。定义。这个例子展示了如何使用TestVisible来注释私有类成员变量和私有方法。

````apex
public class TestVisibleExample {
    // Private member variable
    @TestVisible private static Integer recordNumber = 1;
 
    // Private method
    @TestVisible private static void updateRecord(String name) {
        // Do something
    }
}

````

````apex
@isTest
private class TestVisibleExampleTest {
    @isTest static void test1() {
        // Access private variable annotated with TestVisible
        Integer i = TestVisibleExample.recordNumber;
        System.assertEquals(1, i);
 
        // Access private method annotated with TestVisible
        TestVisibleExample.updateRecord('RecordName');
        // Perform some verification
    }
}

````

## Apex REST 注解

添加了6个新的注释，使您能够将Apex类公开为RESTful Web服务。



###  @RestResource

@RestResource用于类级别注解,语序您将Apex类公开为REST资源.



### @HttpDelete

遵循rest风格,主要用在用于进行删除操作的方法公开为上,将方法公开为REST资源.

方法必须为全局静态方法



### @HttpGet

主要用于查询操作,接收get请求,如果http请求进行head请求同样会触发.返回查询结果.



### @HttpPatch

此方法在发送http patch请求时调用,并更新指定资源.



### @HttpPut

主要用于修改或创建操作,新增或修改一个资源,接收put请求



### @HttpPost

主要用于添加操作,新增一个资源.接收post请求



## apex类与java类的区别:

> 内部类和接口只能在外部类的内部一层声明
>
> 静态变量和方法只能在顶级类中声明,而不能在内部类中声明
>
> 内部类不能使用this关键字来访问外部实例
>
> 默认的访问修饰符为private
>
> public访问修饰符意味着该方法或变量可由此应用或名称空间的任何定点使用
>
> global访问修饰符意味着该方法或变量可由任何具有类访问权的apex代码使用
>
> 方法和类在默认情况下是final的
>
> 被重写的方法需要override修饰
>
> 异常类必须扩展(继承)Exception
>
> 一个类如果希望可以被扩展,需要添加virtual关键字



## 类的自定义创建

类定义创建使用类编辑器在Salesforce中创建一个类。

1. 从安装程序中，在快速查找框中输入Apex类，然后选择Apex类。

2. 单击New。

3. 单击“版本设置”以指定Apex的版本和与此类一起使用的API。如果您的组织已经安装了来自AppExchange的托管包，您还可以指定每个托管包的哪个版本与这个类一起使用。对所有版本使用默认值。这将类与最新版本的Apex和API以及每个托管包相关联。如果希望访问与最新包版本不同的组件或功能，可以指定托管包的旧版本。您可以指定Apex的旧版本和API来维护特定的行为。

4. 在类编辑器中，输入类的Apex代码。单个类的长度最多可达100万个字符，不包括注释、测试方法或使用@isTest定义的类。

5.  单击Save保存您的更改并返回到类详细信息屏幕，或者单击Quick Save保存您的更改并继续编辑您的类。在保存您的类之前，您的Apex类必须正确编译。还可以通过单击“从WSDL生成”从WSDL自动生成类。参见SOAP服务:从WSDL文档定义类。保存后，可以通过类方法或其他Apex代码(如触发器)调用类。

   

## 命名约定

我们建议遵循Java标准进行命名，也就是说，类以大写字母开头，方法以小写动词开头，变量名应该有意义在同一个类中定义具有相同名称的类和接口是不合法的。内部类与其外部类具有相同的名称也是不合法的。但是，方法和变量在类中有自己的名称空间，所以这三种类型的名称不会相互冲突。特别是，变量、方法和类中的类具有相同的名称是合法的。



## 名称就近原则

相同变量名 ,会优先使用局部变量.可以使用this关键字进行特殊指明.



## 名称空间 namespace

就是......例如:

Person person = new Person(); <=> System.Person person = new System.Person();

默认的系统名称空间前缀可以省略不写.



## 模式名称空间  Schema namespace

........告辞



## equals and hashCode

同java一致,.......请到idea快捷键生成



## 在Apex中处理数据

通过在持久层添加数据并与之交互. SObject数据类型是保存数据对象的主要数据类型.

使用DML语句进行数据交互.



### sObject的类型

SObject变量表示一行数据

### Custom labels

自定义标签,自定义标签不是标准对象,不能创建自定义标签的新实例.只能使用system访问自定义标签的值,像system.label.label_name.

````java
String errorMsg = System.Label.generic_error;

````



## Data Manipulation Language(DML)

数据库操作语言.



