# 【Drools规则引擎技术指南】读书札记

## 第一章 基石篇  

* 概念  
**LHS**是用来放置条件  
**RHS**是编写满足条件后处理结果的    
**drools的Rete算法**是drools的核心算法  
**Fact对象**：Fact是指在Drools规则应用中（是指规则因子），JavaBean插入Working、Memory中变成Fact之后，Fact对象不是原来的JavaBean对象进行克隆。而是原来JavaBean对象的引用  



* 对象引用  
规则对于数字型字符串是会自动转化的，但是要注意，只限于使用==号  
<code>
    public class Person{
        private String age;
    }  

    $p:Person(age==15)
</code>  

## 第一章 基础篇  
一套完整的规则文件内容如下：  

关键字  |  描述  
-|-|-  
package  |  包名，只限于逻辑上的管理，若自定义的查询或函数位于同一个包名，不管物理位置如何，都可以直接调用  
import  |  规则引用问题，导入类或方法  
global  |  全局变量  
function  |  自定义函数  
queries  |  查询  
rule end  |  规则体  

**contains比较操作符**  
contains是用来检查一个Fact对象的某个属性值是否包含一个指定的对象值。与之相反也有not contains  
**memberOf比较运算符**  
memberOf用来判断某个Fact对象的某个字段是否在一个或多个集合中【对于数组和集合对象都适用】，与之相反也有not memberOf  
**matches比较运算符**  
matches用来对某个Fact对象的字段与标准的Java正则表达式进行相似匹配  
**SoundsLike比较符**  
soundsLike用来检查单词是否具有与给定值几乎相同的声音  
**str 检查String字段是否是以某一值开头/结尾，还可以判断字符串长度**  
Object(fieldName str[startsWith|endsWith|length] "String"|1)  


**集合的处理**:  
条件中  $s:Person(classNameList[1]=="xxx")  
结论中  单个读取$s.getClassNameList().get(1) 或者循环读取$1.getClassNameList().iterator().next()  




