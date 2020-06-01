# apex

如何创建一个apex文件.


````text
创建apex文件类型:
https://ap17.lightning.force.com/lightning/setup/ApexPages
右上角设置Developer Console进入console界面
左上角点击file 选择create 进行创建
````

基础内容:

以一个apex类为例

````apex
//官方文档的话,为了apex的安全性,对类要进行inherited sharing 进行修饰
//一个类如果想以基类的身份出现,就要加上virtual关键字
// with sharing 关键字允许您为类指定当前用户的共享规则(继承类会自动共享);
//在系统上下文对象中Apex代码可以访问所有对象和字段
//without sharing 确保不会强制执行当前用户的共享规则.
//实现迭代器进行增强for循环(java)
//如果进行测试,类上标注@isTest 方法上标注@isTest ,声明static 无参 无返回值
public inherited sharing virtual class AmController extend GGInterface  implements Iterator<AmController>{
   
   
   //加载顺序,this,supper的使用和java规则一直
   public AmController(){
       super('111')
   }
   
   
   //为变量提供自动配置,如果只配置get,为可读.只配置set为可写.
   //都配置为可读写状态. 
   public String name{
       get;set;
   }
   
   //如果希望字段不被序列化,加transient关键字进行修饰
   public transient Integer Id;
   
   //final 被final修饰的变量只能被赋值一次,同java
   public final String gender;
   
   //从子类中得到的方法,如果对其进行从写,要在方法体上加上override关键字进行修饰.
   public override void getGG(){
       
   }
   
   public void work(){
       
       //判断类型是否为次类型或次子类
       if('' instanceof Object)
       System.debug('yes')
       
   }
   
   //内部类的with sharing 由自身决定
   //内部规则与java内部类一致
   class inClass{
       
   }
    
    
}
````

基本数据类型:

String ,Double,Float,integer,Date,Datetime,decimal,boolean,Long,blob,ID,Object,Time



与SOQL规则一致,忽略大小写.



=== 比较大小写 ==比较忽略大小写         (!==  !=同)

\>>  \>>>  ^  ^=  位异或 异或与java一致.

流程控制结构,分支结构,与java一致(for,while,while...do,if..else).

````apex
switch on 变量 {

​	when value{

​	}

​	when  else{

​	}

} 
变量 value支持 enum instanceof boolean string
````

创建数据结构略有差异,不支持菱形特性

List<String> list = new List<String>{'1','2','3'}; //数组

Set<String> set = new Set<String>{'1','2','3'}; //链表

Map<String,String> map = new Map<String,String>{'a'=>'A','b'=>'B'} //哈希散列



变量赋值时可以使用 | 进行表达式判断.

String str = boolean||"123";



上下类型转换,与java一致.

表达式和运算符与Java基本一致.





