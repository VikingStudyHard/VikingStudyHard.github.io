---

title: 【Mac】在IntelliJ IDEA中使用Antlr4
date: 2019.9.8
categories: 
 - [环境配置]

tags: 
 - Mac
 - IntelliJ IDEA

description: 在 IntelliJ IDEA 中使用 Antlr4，根据输入自动生成语法树并可视化
---

ANTLR (ANother Tool for Language Recognition) 

# 步骤

1.安装插件 
ANTLR v4 grammar plugin
{% img https://vikingstudyhard.github.io/images/Mac_Antlr4/1AF1FD96D44E679E6326D74866756D89.jpg 1132 %}
 
2.新建一个Maven项目，在pom.xml文件中添加ANTLR4插件和运行库的依赖。
> 查找maven依赖
搜索：org.antlr.v4.runtime maven repository

{% img https://vikingstudyhard.github.io/images/Mac_Antlr4/6C4C411C029D930D6E8218829BB28AE6.jpg 709 %}

在pom.xml里加入以下依赖
```
 <!-- https://mvnrepository.com/artifact/org.antlr/antlr4-runtime -->
<dependency>
    <groupId>org.antlr</groupId>
    <artifactId>antlr4-runtime</artifactId>
    <version>4.7.2</version>
</dependency>
```
{% label danger@注意 %}
Maven要用默认的
{% img https://vikingstudyhard.github.io/images/Mac_Antlr4/EBE9B0AA160B5CF7C39B376177370E3D.jpg 564 %}

3.在`src/main/java/antrl`中导入 Antlr 的文法文件`Java8.g4`。

4.右键`Java8.g4`，选择Configure ANTLR，配置output路径。
{% img https://vikingstudyhard.github.io/images/Mac_Antlr4/BD5A55ECAA911BF3B792E2251DD7CB04.jpg 711 %}

5.右键`Java8.g4`，选择Generate ANTLR Recognizer。
生成结果为：
{% img https://vikingstudyhard.github.io/images/Mac_Antlr4/55F0273B0BF9B1DCB4814D321E01A573.jpg 402 %}
Lexer是Antlr生成的词法分析器，Parser是Antlr生成的语法分析器。

6.`Java8.g4`中找到 compilationUnit，右键 Test Rule compilationUnit
{% img https://vikingstudyhard.github.io/images/Mac_Antlr4/BFADD5B3E1F8803FED21306A9445409C.jpg 1613 %}

可见