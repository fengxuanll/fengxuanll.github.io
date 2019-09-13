---
title: 解析PowerDesigner文件生成C#实体类
date: 2019-09-13 10:30:11
tags:
    - go
    - pdm
categories:
    - go
---


一直在用PowerDesigner设计数据库，但是每次需要在代码里去写实体类的时候，就比较麻烦，只能一个一个字段的去敲，所以这两天尝试使用go语言做了个小工具，解析pdm文件，将需要的表自动生成实体类。

需求：

1.  指定pdm文件路径
2.  指定需要生成实体类的表名
3.  根据模板生成`.cs`实体类文件


<!-- more -->

以下为本次学习到的小知识点：

### 解析xml

因为xml里包含了很多层级的节点，而我需要的节点只是在里面的一部分，刚开始是准备用go自带的包去解析xml，但是找了一圈也没有找到需要的方法，然后再Github上发现了这个库：github.com/beevik/etree，还是挺好用的。

```go
doc := etree.NewDocument()
err := doc.ReadFromFile(path)	// 从文件加载XML
// err := doc.ReadFromString("xml")  // 从字符串加载XML
if err != nil {
	return nil, err
}

tables := doc.FindElements("Model/RootObject/Children/Model/Tables/Table")
for _, t := range tables {
	var commentNode = t.SelectElement("Comment")
	if commentNode != nil {
        comment := commentNode.Text()
	}
}
```

这样就可以把所有的Table节点查找出来。

1.  `FindElements` 是根据xpath直接查找的需要的节点，如果存在多个就会返回多个结果。

2.  如果是查找单个节点的话，就用`FindElement`方法。

    

### 切片

1.  增加元素

    如果要在切片后增加元素，得使用系统函数`append()`，该函数会将从第一个参数开始的元素，添加到第一个参数的后面，并返回一个新的切片。使用方式如下：

    ```go
    var tables []string
    tables = append(tables, "test")
    ```

2.  删除元素

    切片本身不提供直接删除元素的方法，只能先查找到元素在切片中的索引，然后再重新构造一个新的切片。

    ```go
    func main() {
    	s := []string{"a", "a", "b", "c", "a"}
    	fmt.Println(s)  // 输出：[a a b c a]
    
    	s = sliceRemove(s, "a")
    	fmt.Println(s)	// 输出：[b c]
    }
    
    // 删除切片中的指定元素
    func sliceRemove(s []string, item string) []string {
    	totalLength := len(s) - 1
    	for i := totalLength; i >= 0; i-- {
    		if s[i] == item {
    			s = append(s[:i], s[i+1:]...)
    		}
    	}
    	return s
    }
    ```



### 模板

模板用的是`text/template`包。

1.  自定义处理函数，自定义函数可以接收从模板中传过来的参数，并返回结果值。
2.  按模板文件定义模板处理类，并将自定义函数添加上。
3.  模板中使用函数，go代码里调用正常的函数格式是：`funcName(arg1, arg2)`，在模板里就不能这么使用了，得`funcName arg1 arg2`，调用函数不需要括号，传的参数也不需要逗号隔开。
4.  `Execute`是最终执行函数，第一个参数是`io.Writer`类型，可以输出到文件，也可以输出到控制台，第二个参数是传给模板的参数。

```go
funcMap := template.FuncMap{
	"ProcessComment": ProcessComment,
}

t, err := template.New("class.tmpl").Funcs(funcMap).ParseFiles("class.tmpl")
if err != nil {
	return err
}

fileName := table.Code + ".cs"
f, err := os.OpenFile(fileName, os.O_RDWR|os.O_CREATE|os.O_TRUNC, 0644)
if err != nil {
	return err
}
defer f.Close()

err = t.Execute(f, table)
if err != nil {
	return err
}
```

### 字典

这段示例代码里是直接定义字典，也可以使用make函数来定义，语法为：`make(map[key-type]val-type`。

```go
// 定义一个字典，并初始化了两个元素
m := map[string]string{
	"int":    "int",
	"bigint": "long"}
fmt.Println(m)	// 结果：map[bigint:long int:int]

// 添加一个元素
m["varchar"] = "string"
fmt.Println(m)	// 结果：map[bigint:long int:int varchar:string]

// 删除一个元素
delete(m, "int")
fmt.Println(m)	// 结果：map[bigint:long varchar:string]

// 尝试获取一个元素，如果不存在，ok会返回false
_, ok := m["int"]
fmt.Println(ok)	// 结果：false

item, ok := m["varchar"]
fmt.Println(fmt.Sprintf("%t: %s", ok, item))	// 结果：true: string这是一种定义
​```


### 其它

1. 最好是只在主方法里panic异常，其它方法只要判断异常并返回就行了，这样子貌似清晰一点。
2. 个人觉得go还是没有像C#那么成熟，很多语法糖层面的东西都没有提供，当然go本来就还比较年轻，或者也不想做那么多的语法糖。