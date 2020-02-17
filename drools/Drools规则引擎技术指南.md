# 【Drools规则引擎技术指南】读书札记

## 第一章 基石篇  

* 概念  
**LHS**是用来放置条件  
**RHS**是编写满足条件后处理结果的    
**drools的Rete算法**是drools的核心算法  
**Fact对象**：Fact是指在Drools规则应用中（是指规则因子），JavaBean插入Working、Memory中变成Fact之后，Fact对象不是原来的JavaBean对象进行克隆。而是原来JavaBean对象的引用  
**drl文件内容的单行注释不要用"#"号**  


* 对象引用  
规则对于数字型字符串是会自动转化的，但是要注意，只限于使用==号  
<code>
    public class Person{
        private String age;
    }  

    $p:Person(age==15)
</code>  

## 第二章 基础篇  
一套完整的规则文件内容如下：  

关键字 | 描述  
-|-|-
package | 包名，只限于逻辑上的管理，若自定义的查询或函数位于同一个包名，不管物理位置如何，都可以直接调用
import | 规则引用问题，导入类或方法
global | 全局变量
function | 自定义函数
queries | 查询
rule end | 规则体


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


**集合的处理**：  
条件中  $s:Person(classNameList[1]=="xxx")  
结论中  单个读取$s.getClassNameList().get(1) 或者循环读取$1.getClassNameList().iterator().next()  

**12个规则属性**：  
activation-group、agenda-group、auto-focus、date-effective、date-expires、dialet、duration、enalbled、lock-on-active、no-loop、ruleflow-group、salience  
**no-loop**  
默认值：false  
属性说明：防止死循环，当规则通过update之类的函数修改了Fact对象时，可能使规则再次被激活，从而导致死循环。将no-loop设置为true的目的是避免当前那规则then部分被修改后的事实再次被激活，从而防止死循环发生。  
*tips*  
在一个规则文件中，一个Fact对象通过Drools函数被修改，规则体将再次激活。也就是说，在RHS部分使用了update相类似的语法(insert同理)，变更了Fact对象在规则中的内容，就会导致规则重新被激活和匹配。  

**ruleflow-group**  
**lock-on-active**  
默认值：false  
属性说明：一个更强大的解决死循环的属性，即无论如何更新规则事实对象，当前规则也只能被触发一次。  

**salience**  
默认值：0  
属性说明：规则体被执行的顺序。可以是负数，其值越大，执行顺序越高，排名越靠前。Drools还支持动态配置优秀级  
<code>
rule "isSalience4"salience (Math.random() * 10 + 1)  
when  
then  
    System.out.println("isSalience4");  
end  
</code>  

**enabled**  
默认值：true  
属性说明：指规则是否可以执行，若规则体设置为enabled false，则规则体将视为永久不被激活  

**date-effective**  
属性说明：只有当期那系统时间大于等于设置的时间或日期，规则才会被激活。可接受的日期格式为"dd-MMM-yyyy"  
**date-expires**  
与上面的属性相反，Drools规则引擎日期属性格式为dd-MMM-yyyy是并不常用的格式，在开发的过程中，一般都会写成yyyy-MM-dd形式。如果在
规则中使用通俗的Java日期格式，需要在Java调用规则中添加代码  
<code>
System.setProperty("drools.dateformat","yyyy-MM-dd");  
</code>  
使用日期格式化注意，需要在创建会话之前执行该代码，也就是在实例kieSession之前  


**duration**  
**activation-group**  
属性说明：激活分组，通过字符串定义分组名称，具有相同名称的规则体有且只有一个规则被激活，其他规则体的LHS部分仍然为true也不会再被执行。  
activation-group属性类似于规则流程中的XOR网关，当有规则体被执行完毕后，其他规则将不会被激活，即使其他规则中的LHS部分为true。  

**agenda-group**  
属性说明：agenda-group是议程分组，属于另一种可控的规则执行方式，是指用户可以通过配置agenda-group的参数来控制规则的执行，而且只有获取焦点的规则才会被激活。  
**auto-focus**  
默认值：false  
***timer*  
属性说明：timer属性是一个定时器，用来控制规则的执行时间  


## 第三章 中级篇  
**global**  
global全局变量与Fact(事实)对象不同，不会因为值变化而影响到规则的再次激活  
**function**  
感觉可以把function看作java中的静态方法理解  
**declare**  
declare声明在规则引擎中的功能主要有两个：一是声明新类型，二是声明元数据类型  
declare可以声明实体之间的继承关系  
**when**  
条件元素eval尽可能少点用，导致引擎的性能问题  
条件元素not与exists一个代表不存在，另一个代表存在，两者也可以搭配使用  
<code>not (exist Person())</code>  
**from**  
条件元素from支持对象源，返回一个
**collect**  
collect收集，可以和from一起用  
**halt**  
立即终止规则执行，只要有一条规则执行了drools.halt，则其他规则将不再进行判断，直接返回结束  

**规则模板**  
规则模板是指规则条件比较值是可变的，且可生成多个规则进行规则调用。  


## 第三章 扩展篇  
RETE网络算法，节点是共享的，规则在执行过程中不关心规则的个数。  
  













