#  apex入门day04

## JSON

json作为一种轻量级的数据传输格式,salesforce也对其有良好的的支持.在salesforce中前后台交互时,使用JSON

可以将apex中的object对象进行序列化和反序列化,

主要有三个类来处理JSON:  

1.System.JSON   

2.System.JSONGenerator

3.System.JSONParser



### System.JSON

使用方法:

````
List<Student__c> studentList = [SELECT Id,Name,gender__c,education__c from Student__c LIMIT 1];

        //通过serialize序列化(一行打印)
        String str1 = System.JSON.serialize(studentList);
        //通过serializePretty序列化(多行打印)
        String str2 = System.JSON.serializePretty(studentList);

        //此方法用于将JSON内容反序列化成Apex的Object对象
        List<Student__c> deStudentList1 = JSON.deserialize(str1);
        //此方法用于将指定的JSON内容反序列化成基本数据类型的集合，如果不是基本数据类型，则在反序列化时报异常
        JSON.deserializeUntyped();
````



其他请参考:

https://www.cnblogs.com/zero-zyq/p/5372117.html



## Trigger

详情请查看: https://www.cnblogs.com/zero-zyq/p/5413731.html