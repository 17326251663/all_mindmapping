# day03-SOQL-SOSL

##  在SOQL和SOSL查询中使用apex变量

在apex中的SOQL,SOSL查询中可以使用apex变量和表达式 ,使用的方式为加前缀 ":"  ,称为绑定.

可以用于:

> 查找子句中的搜索字符串
>
> 在WHERE子句中过滤文字
>
> IN或NOT IN操作符在WHERE子句中的值.允许对一组动态值进行过滤。注意，这对于id或字符串列表特别有用.尽管它可以用于任何类型的列表分部名称
>
> 与分部子句一起使用
>
> LIMIT子句中的数值
>
> 偏移子句中的数值
>
> (距离(DISTANCE)函数中的单位参数不支持Apex绑定变量.)

````apex
Account A = new Account(Name='xxx');
insert A;
Account B;

 
// A simple bind
B = [SELECT Id FROM Account WHERE Id = :A.Id];


// A bind with arithmetic
B = [SELECT Id FROM Account
     WHERE Name = :('x' + 'xx')];
 
String s = 'XXX';
 
// A bind with expressions
B = [SELECT Id FROM Account
     WHERE Name = :'XXXX'.substring(0,3)];
 
// A bind with an expression that is itself a query result
B = [SELECT Id FROM Account
     WHERE Name = :[SELECT Name FROM Account
                    WHERE Id = :A.Id].Name];
 
Contact C = new Contact(LastName='xxx', AccountId=A.Id);
insert new Contact[]{C, new Contact(LastName='yyy',
                                    accountId=A.id)};
 
// Binds in both the parent and aggregate queries
B = [SELECT Id, (SELECT Id FROM Contacts
                 WHERE Id = :C.Id)
     FROM Account
     WHERE Id = :A.Id];
 
// One contact returned
Contact D = B.Contacts;
 
// A limit bind
Integer i = 1;
B = [SELECT Id FROM Account LIMIT :i];
 
// An OFFSET bind
Integer offsetVal = 10;
List<Account> offsetList = [SELECT Id FROM Account OFFSET :offsetVal];
 
// An IN-bind with an Id list. Note that a list of sObjects
// can also be used--the Ids of the objects are used for
// the bind
Contact[] cc = [SELECT Id FROM Contact LIMIT 2];
Task[] tt = [SELECT Id FROM Task WHERE WhoId IN :cc];
 
// An IN-bind with a String list
String[] ss = new String[]{'a', 'b'};
Account[] aa = [SELECT Id FROM Account
                WHERE AccountNumber IN :ss];

// A SOSL query with binds in all possible clauses

String myString1 = 'aaa';
String myString2 = 'bbb';
Integer myInt3 = 11;
String myString4 = 'ccc';
Integer myInt5 = 22;
 
List<List<SObject>> searchList = [FIND :myString1 IN ALL FIELDS
                                  RETURNING
                                     Account (Id, Name WHERE Name LIKE :myString2
                                              LIMIT :myInt3),
                                     Contact,
                                     Opportunity,
                                     Lead
                                  WITH DIVISION =:myString4
                                  LIMIT :myInt5];

````



## All ROWS

[select count() from Account where Accound.id = a.ID ALL ROWS];

SOQL语句可以使用 ALL ROWS关键字以查询组织中的所有记录，包括已删除的记录和已归档的活动。

只能用于查询语句.



## SOQL 循环

既=即遍历由SOQL查询返回的所有SObject记录

````
for (variable : [soql_query]) {
    code_block
}
````



## SObject集合

可以在List,Set,Map中管理SObject.



	### List\<sObject>

使用:

````
//创建List
List<Account> myList = new List<Account>();

//从SOQL查询自动填充List<sObject>
List<Account> accts = [SELECT Id, Name FROM Account LIMIT 1000];

//添加和检索sObject
List<Account> myList = new List<Account>(); // Define a new list
Account a = new Account(Name='Acme'); // Create the account first
myList.add(a);                    // Add the account sObject
Account a2 = myList.get(0);      // Retrieve the element at index 0

//批量添加
// Define the list
List<Account> acctList = new List<Account>();
// Create account sObjects
Account a1 = new Account(Name='Account1');
Account a2 = new Account(Name='Account2');
// Add accounts to the list
acctList.add(a1);
acctList.add(a2);
// Bulk insert the list
insert acctList;


````

记录ID生成

Apex会自动为使用DML插入或插入的sObject列表中的每个对象生成ID。因此，即使包含一个sObject实例的多个列表都不能被插入或升序。null ID。这种情况意味着需要将两个ID写入内存中的同一结构，这是非法的。

````
Account a = new Account();
List<Account> accs = new List<Account>{a, a};

insert accs;
````



此外还可以用一维数组的格式表示List

#### List\<sObject>排序

使用List.sort()对list进行排序.

#### 实现自定义排序

实现`implements` `Comparable`接口,自定义外部排序.

```
global class OpportunityWrapper implements Comparable {

    public Opportunity oppy;

    // Constructor
    public OpportunityWrapper(Opportunity op) {
        oppy = op;
    }

    // Compare opportunities based on the opportunity amount.
    global Integer compareTo(Object compareTo) {
        // Cast argument to OpportunityWrapper
        OpportunityWrapper compareToOppy = (OpportunityWrapper)compareTo;

        // The return value of 0 indicates that both elements are equal.
        Integer returnValue = 0;
        if (oppy.Amount > compareToOppy.oppy.Amount) {
            // Set return value to a positive value.
            returnValue = 1;
        } else if (oppy.Amount < compareToOppy.oppy.Amount) {
            // Set return value to a negative value.
            returnValue = -1;
        }
         
        return returnValue;      
    }
}

```

### Set\<sObject>

集包含唯一元素。sObjects的唯一性是通过比较对象的字段来确定的。例如，如果您尝试将两个具有相同名称的帐户添加到集合中，而未设置其他字段，则仅将一个sObject添加到该集合中。

如果将描述添加到其中一个帐户，则该描述将被视为唯一，并且两个帐户都将添加到该帐户中。



### Map<sObject,sObject>

键唯一,遵java





## Trigger

可以使用invoked来调用apex,Apex触发器使您在修改salesforce记录(例如插入,更新,或删除)之前或之后执行自定义操作.

触发器是在以下类型之前或之后执行的apex代码:

insert

update

delete

merge

upsert

undelete

例如，您可以在将对象的记录插入数据库之前，删除记录之后，甚至从回收站中还原记录之后运行触发器。

您可以为支持触发器的顶级标准对象（例如联系人或客户），一些标准子对象（例如CaseComment）和自定义对象定义触发器。要定义触发器，请从要访问其触发器的对象的对象管理设置中转到触发器。

有两种类型的触发器：

- 在触发器被用于更新或验证记录值之前，将其保存到数据库之前。
- After触发器用于访问系统设置的字段值（例如记录的ID 要么 LastModifiedDate字段），并影响其他记录的更改，例如登录审核表或使用队列触发异步事件。*触发after触发器*的记录是只读的。

在创建触发器之前，请考虑以下事项：

- 上升 触发前后 插 或之前和之后 更新 适当触发。
- 合并 触发前后 删除 对于丢失的记录，以及之前和之后 更新触发获胜记录。请参阅[触发器和合并语句](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_triggers_merge_statements.htm)。
- 取消删除记录后执行的触发器仅适用于特定对象。请参阅[触发器和恢复的记录](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_triggers_recovered_records.htm)。
- 直到触发器结束，才记录字段历史记录。如果在触发器中查询字段历史记录，则看不到当前事务的任何历史记录。
- 字段历史记录跟踪当前用户的权限。如果当前用户无权直接编辑对象或字段，但是用户在启用历史记录跟踪的情况下激活了更改对象或字段的触发器，则不会记录更改的历史记录。
- 必须从触发器异步进行标注，以便在等待外部服务的响应时不会阻止触发过程。异步调用是在后台进程中进行的，并且在外部服务返回响应时会收到响应。要进行异步标注，请使用异步Apex（例如未来方法）。有关更多信息，请参见[使用Apex调用标注](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_callouts.htm#apex_callouts)。
- 在API版本20.0和更早版本中，如果Bulk API请求导致触发触发器，则将要处理的触发器的200条记录中的每个块都分成100条记录中的块。在Salesforce API 21.0版和更高版本中，不会进一步拆分API块。如果Bulk API请求导致触发器多次触发200条记录的块，则在同一HTTP请求的这些触发器调用之间重置控制器限制。

### Bulk Triggers(批量触发)

默认情况下，所有触发器都是批量触发器，并且可以一次处理多个记录。您应该始终计划一次处理多个记录。



​	触发语法:

````
trigger TriggerName on ObjectName (trigger_events) {
                     code_block
                     }
````



例如，以下代码为 插入之前 和 更新之前 Account对象上的事件：

````
trigger myAccountTrigger on Account (before insert, before update) {
    // Your code here
}
````

触发器的代码块不能包含 静态的关键词。触发器只能包含适用于内部类的关键字。此外，您不必手动提交由触发器进行的任何数据库更改。如果您的Apex触发器成功完成，则所有数据库更改都将自动提交。如果您的Apex触发器未成功完成，则将回滚对数据库所做的任何更改。



触发上下文变量:

<https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_triggers_context_variables.htm>



## async apex

Apex提供了多种异步运行您的Apex代码的方法。选择最适合您需求的异步Apex功能。

### queue Apex

通过important Queueable接口

具体参照:

<https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_queueing_jobs.htm>



## 将Apex方法公开为SOAP Web服务

Apex类方法可以作为自定义SOAP Web服务调用公开。这允许外部应用程序调用Apex Web服务以在Salesforce中执行操作。使用webservice关键字定义这些方法。例如：

````
	global class MyWebService {
    webservice static Id makeContact(String contactLastName, Account a) {
        Contact c = new Contact(lastName = contactLastName, AccountId = a.Id);
        insert c;
        return c.id;
    }
}

https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_web_services_methods_considerations.htm
````

## 将Apex类公开为REST Web服务

<https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_rest.htm>

### Apex REST简介

您可以公开Apex类和方法，以便外部应用程序可以通过REST体系结构访问您的代码和应用程序。这是通过使用以下命令定义Apex类来完成的：@RestResource注释以将其公开为REST资源。同样，将注释添加到您的方法以通过REST公开它们。例如，您可以添加@HttpGet 方法的注释，以将其公开为可以由HTTP调用的REST资源 得到请求。有关更多信息，请参见[Apex REST注释](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_classes_annotations_rest.htm)

这些类包含可与Apex REST一起使用的方法和属性。



对Apex REST类的调用计入组织的API调控器限制。所有标准Apex调控器限制都适用于Apex REST类。例如，对于同步Apex，最大请求或响应大小为6 MB；对于异步Apex，最大请求或响应大小为12 MB。有关更多信息，请参见[执行执行器和限制](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_gov_limits.htm)。



Apex REST支持以下身份验证机制：

- OAuth 2.0
- 会话ID

### Apex REST批注

添加了六个新的注释，使您可以将Apex类公开为RESTful Web服务。

- [@RestResource](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_classes_annotation_rest_resource.htm)（urlMapping = '/ yourUrl '）
- [@HttpDelete](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_classes_annotation_http_delete.htm)
- [@HttpGet](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_classes_annotation_http_get.htm)
- [@HttpPatch](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_classes_annotation_http_patch.htm)
- [@HttpPost](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_classes_annotation_http_post.htm)
- [@HttpPut](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_classes_annotation_http_put.htm)

### Apex REST方法

Apex REST支持两种表示资源的格式：JSON和XML。默认情况下，JSON表示形式在请求或响应的正文中传递，格式由 Content-Type HTTP标头中的属性。如果Apex方法没有参数，则可以从HttpRequest对象中检索Blob主体。如果在Apex方法中定义了参数，则会尝试将请求正文反序列化为那些参数。如果Apex方法具有非无效返回类型，则资源表示形式将序列化到响应主体中。

Apex REST不支持Apex对象中的Chatter的XML序列化和反序列化。Apex REST确实支持Apex对象中的Chatter的JSON序列化和反序列化。



用注释的方法 @HttpGet 要么 @HttpDelete应该没有参数。这是因为GET和DELETE请求没有请求正文，因此没有反序列化的内容。

单个带有注释的Apex类 @RestResource 不能有多个使用相同HTTP请求方法注释的方法。例如，同一个类不能有两个用@HttpGet注释的方法。



**Apex REST当前不支持Content-Type的请求 :Multipart/form data**

### 用户定义的类型

您可以在Apex REST方法中为参数使用用户定义的类型。Apex REST将请求数据反序列化为上市， 私人的， 要么 全球 用户定义类型的类成员变量，除非将该变量声明为 静态的 要么 短暂的。例如，包含用户定义的类型参数的Apex REST方法可能如下所示：

````
@RestResource(urlMapping='/user_defined_type_example/*')
global with sharing class MyOwnTypeRestResource {

    @HttpPost
    global static MyUserDefinedClass echoMyType(MyUserDefinedClass ic) {
        return ic;
    }

    global class MyUserDefinedClass {

        global String string1;
        global String string2 { get; set; }
        private String privateString;
        global transient String transientString;
        global static String staticString;
    }
}
````



## Apex REST基本代码示例

<https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_rest_code_sample_basic.htm>



基本代码示例:

<https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_rest_code_sample_restrequest.htm>



## 命名空间

### Schema命名空间

Schema命名空间和Schema类不是同一个概念。Schema类属于System命名空间下，Schema命名空间包含很多类处理Schema元数据信息。

​    Schema类的方法包含schema 信息描述。

​    Schema类主要方法如下：

- public static Map<String, Schema.SObjectType> getGlobalDescribe()：

​    返回一个map，这个map表示所有的sObject名称(key)到sObject tokens(value)的map，其中tokens可以为在你的项目中标准的和自定义的Object对象。



这个map具有以下的特征：

​    1.动态的，根据权限在运行时生成sObject;

​    2.sObject名称不区分大小写；

​    3.key可以映射出Object是否是一个自定义对象；

​    4.key如果是标准的sObject则不需要前缀，否则需要加命名空间的前缀。

- public static List<Schema.DescribeDataCategoryGroupResult> describeDataCategoryGroups(String sObjectNames)

​    返回一个与指定的对象关联的类别组列表.

- public static List<Schema.DescribeSObjectResult> describeSObjects(List<String> sObjectTypes)

​    返回指定的sObject的描述信息。通常可以先调用getGlobalDescribe()方法获取组织中所有的对象列表，然后通过迭代遍历使用此方法获取指定的单个的sObject的元数据信息。



**Schema命名空间**

​    Schema命名空间下的类和方法用来处理schema 元信息(metadata)，当实例化或者使用Schema类或者方法的时候，可以省略Schema命名空间。



以下的代码中封装了PickList的values的值的获取方法，形参分别为需要获取的sObjectName以及字段的名称，如果不存在指定的sObjectName或者字段名称没有设置返回值

````
/**
 * Created by 喜静 on 2020/4/29.
 */

public with sharing class PickListValuesUtil {


    public Map<String,Object> getPicklistValues(String sObjectName,String sFieldName){

        Map<String,Object> result = new Map<String, Object>();

        Map<String,SObjectType> sMap = Schema.getGlobalDescribe();

        //判断
        if (sMap.containsKey(sObjectName)) {
            //存在此对象的名称
                Map<String,Schema.SObjectField> sObjectFieldMap = sMap.get(sObjectName).getDescribe().fields.getMap();
                    //判断是否存在此字段
            if (sObjectFieldMap.containsKey(sFieldName)) {
              Schema.DescribeFieldResult sDescribeFieldResult =  sObjectFieldMap.get(sFieldName).getDescribe();
               List<Schema.PicklistEntry> picklistEntries = sDescribeFieldResult.getPicklistValues();
                    for (Schema.PicklistEntry  picklistEntry: picklistEntries){
                            result.put(picklistEntry.value,new Map<String,Object>{
                                    'value'=>picklistEntry.value,
                                    'isActive'=>picklistEntry.isActive(),
                                    'isDefaultValue' => picklistEntry.isDefaultValue(),
                                    'label' => picklistEntry.label
                            });
                    }
            }
        }

        return  result;

    }

}
````



## ApexPages命名空间

此命名空间下的类主要用于VF的控制.

主要的类包括但不限于以下：

- ApexPages.StandardController:当为一个标准Controller定义扩展的时候使用此类。StandardController对象为Salesforce提供的预构建VF的控制器对象引用;
- ApexPages.Action:使用Action类和方法用于VF自定义控制器和扩展中，实现前后台交互;
- ApexPages.Message:可以使用此类将信息传递到前台显示，常用于显示异常信息（系统异常or自定义异常）;



