# jsoup学习一
jsoup是一个html解析和查找的第三方工具包.

jsoup是我使用最方便的工具包,最为dom解析的工具包,对外暴露的接口十分简洁.和org.w3c.dom.* 中的dom树相比方便很多,本身救提供jQuery选择器使用方便.

> 相关使用文档
[官方网站](https://jsoup.org/)
[英文文档](https://jsoup.org/cookbook/)
[中文简单文档](http://www.open-open.com/jsoup/)


##  jsoup 解析
> jsoup的解析采用一个状态进行解析,代码上看的比较复杂.需要先理解状态机是什么和相应的状态转移表.采用状态模式进行状态机的实现.

### 设计模式-状态模式

1. 概述
> 当一个对象的内在状态改变时允许改变其行为，这个对象看起来像是改变了其类。
2. 解决的问题
> 主要解决的是当控制一个对象状态转换的条件表达式过于复杂时的情况。把状态的判断逻辑转移到表示不同的一系列类当中，可以把复杂的逻辑判断简单化。在软件开发过程中，应用程序可能会根据不同的情况作出不同的处理。最直接的解决方案是将这些所有可能发生的情况全都考虑到。然后使用if... ellse语句来做状态判断来进行不同情况的处理。但是对复杂状态的判断就显得“力不从心了”。随着增加新的状态或者修改一个状体（if else(或switch case)语句的增多或者修改）可能会引起很大的修改，而程序的可读性，扩展性也会变得很弱。维护也会很麻烦。那么我就考虑只修改自身状态的模式。
3. 模式中的角色
>3.1 上下文环境（Context）：它定义了客户程序需要的接口并维护一个具体状态角色的实例，将与状态相关的操作委托给当前的Concrete State对象来处理。

> 3.2 抽象状态（State）：定义一个接口以封装使用上下文环境的的一个特定状态相关的行为。

> 3.3 具体状态（Concrete State）：实现抽象状态定义的接口。

![image](http://my.csdn.net/uploads/201205/11/1336719144_5496.jpg)


### 基本讲解

TreeBuilder和HtmlTreeBuilder进行html源码的解析.针对xml有对应的XmlTreeBuilder但是本质没有变化.

html解析开始
```
// TreeBuilder中的方法进行状态解析的开始
protected void runParser() {
    while (true) {
        // 进行词法分析
        Token token = tokeniser.read();
        // 根据已得到的词进行语法分析
        process(token);
        token.reset();

        if (token.type == Token.TokenType.EOF)
            break;
    }
}
```
> Token当前词法分析结果.

> Tokeniser进行词法分析过程和状态存储.

> TokeniserState是一个枚举类.包含词法分析的各种状态和状态转移操作.

> HtmlTreeBuilder作为语法分析过程,构建根据Token构建dom树.

> HtmlTreeBuilderState是一个枚举类.包含语法分析的各种状态和状态转移操作.

### 词法分析主要状态的转移
1. Data 主要是非标签的数据并将数据状态转移到(TagOpen) 标签部分Tag状态
2. 包含Tag元素 主要是标签内数据的状态转移, 包括从TagOpen 转移到 TagName, TagName 转移到标签属性BeforeAttributeName 等多种转移
3. 包含Attr元素 标签属性和属性值 和最终转移到 标签结束 SelfClosingStartTag
4. 包含Comment元素 主要是注释代码
5. 其他的元素主要是 doctype,script,异常的错误数据的状态
6. CharacterReferenceInData html转移符状态
> 代码还是相对好看的,只是第一次硬看代码理解有点费劲,在脑海中想一想符合规范的html对应枚举元素名很快就看懂了状态转移.

以下是简单的状态转移内容
```
Data - & -> CharacterReferenceInData
Data - < -> TagOpen
CharacterReferenceInData - - Data
TagOpen - / -> EndTagOpen
TagOpen - [a-zA-Z] -> TagName
TagOpen - [^a-zA-Z]-> Data
EndTagOpen - pos>=length -> Data
EndTagOpen - [a-zA-Z] -> TagName
EndTagOpen - pos>=length && > -> Data
TagName - \s ->  BeforeAttributeName
TagName - / -> SelfClosingStartTag
BeforeAttributeName - [a-zA-Z] ->  AttributeName
AttributeName - = ->BeforeAttributeValue
BeforeAttributeValue - "|' -> AttributeValue_doubleQuoted/AttributeValue_singleQuoted
```

### Token主要
> Doctype 

> Tag 标签属性

### 语法分析主要状态和转移

1. Initial 状态作为开始状态,转移 BeforeHtml状态
2. BeforeHtml 进入 BeforeHead,如果不存在head就追加html状态在进入BeforeHead
3. BeforeHead 进入 InHead或InBody
4. InBody 处理十分负责,process方法针对所有标签都有对应处理, 回转移到多种状态
5. Text是非html标签的文本内容
5. 方法太长不在进行状态转移说明,根据枚举类型可以判断状态