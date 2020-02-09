# 【Drools规则引擎技术指南】读书札记

## 第一章 基石篇  

* 概念  
**LHS**是用来放置条件  
**RHS**是编写满足条件后处理结果的    


* 对象引用  
规则对于数字型字符串是会自动转化的，但是要注意，只限于使用==号  
<code>
    public class Person{
        private String age;
    }  

    $p:Person(age==15)
</code>  
