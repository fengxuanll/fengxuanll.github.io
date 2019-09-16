---
title: go语言推送文件到git
tags:
  - go
  - git
categories:
  - go
toc: false
date: 2019-09-08 22:04:35
---

这段时间在找一个比较好用的Markdown编辑器软件，然后找到了Tea、Typora这两个，但是Tea的文件不是直接存储在文件夹中的，是存储在数据库里头的，所以就考虑使用Typora了，而Typora又没有同步文件的功能，只能单机使用，所以这两天就自己用go写了个自动推送文件到git的小工具。

<!-- more -->

需求比较简单，执行程序时，自动将指定目录下的所有文件推送到git上去。需求虽然比较简单，要做的事情相当少，但是在做的过程中发现，一个小东西想要做得比较完善，其实也还是有不少东西需要考虑的。

最初级版本，当然只是需要按顺序执行每个命令即可，刚开始为了快速测试，也就是这么实现了，在一切正常的情况下，这样也没问题，能正常使用。但是在测试的时候，如果执行时有任何错误，程序就跑不下去了，有问题也不知道。

然后就加上异常处理，错误提示，因为对go、git不太熟悉，在这个过程中还是碰到不少问题，当然也学习到了新的知识。



### 学习到的新知识

1.  在go语言中怎么执行外部程序

    ```go
    cmd := exec.Command(path, args...)	// 实例化一个执行程序，指定执行程序路径和参数
    cmd.Dir = ""	// 指定执行目录
    output, err := cmd.CombinedOutput()	// 执行程序，并返回程序的输出内容
    result := string(output)	// 将输出内容转换为字符串
    ```

    cmd里还有一些其它的方法是执行命令的，CombinedOutput是要等一个命令执行完成后，才会返回执行结果。Run是直接执行，不返回执行结果，CombinedOutput其实内部也是调用的Run方法，只是加了异常的处理。

    在Mac中如果直接执行go语言编译后的执行文件，默认路径会在当前登录用户的文件夹下，所以执行本程序的时候，会提示当前目录不是git仓库，所以需要指定cmd的Dir。
    
    这里需要注意下，目前碰到有Windows下部分命令不能直接运行，会提示：`exec: "appcmd.exe": executable file not found in %PATH%`，解决这种情况需要使用`cmd`命令来执行，比如，本来要执行的是：`appcmd list apppool`，改成执行：`cmd /C appcmd list apppool`。

2.  字符串拆分

    ```go
    strings.Split("", "")
    ```

3.  字符串去除前后空行

    ```go
    strings.Trim("", "\n")
    ```

4.  获取当前执行文件的目录

    ```go
    dir, _ := filepath.Abs(filepath.Dir(os.Args[0]))
    ```

5.  在Mac中隐藏和显示隐藏文件的快捷键：`command+shift+.` ，这玩意以前不知道，老是得去命令行执行一串老长的命令，有这个快捷键可就方便多了。

6.  解析命令行参数

    ```go
    type sliceValue []string
    
    func (s *sliceValue) Set(val string) error {
    	*s = sliceValue(strings.Split(val, ","))
    	return nil
    }
    
    func (s *sliceValue) String() string { return strings.Join([]string(*s), ",") }
    
    var (
    	dir string	// 定义全局变量，方便其它方法中使用
        dirs sliceValue
    )
    
    func init() {
        // 参数1：解析到哪个变量上
        // 参数2：命令参数
        // 参数3：默认值
        // 参数4：该参数的说明
    	flag.StringVar(&dir, "d", "默认值", "指定执行命令的文件夹路径，必须为绝对路径。")
        // 自定义解析参数
        flag.Var(&dirs, "p", "指定多个执行命令的文件夹路径，必须为绝对路径")
    
    	flag.Usage = func() {
    		fmt.Fprintf(os.Stdin, `Usage: push2git [-d dir]`)
    	}
    }
    
    func main() {
        // 解析参数
        flag.Parse()
        
        fmt.Println(dir)
    }
    ```

    这样就可以使用解析到的命令行提供的参数了。自定义解析类，必须实现flag.Value接口的Set、String方法。

### 待完善的点

1.  ~~允许指定目录执行该程序，这样就不用每个目录下都放一个执行程序了~~
2.  ~~允许指定提交到git的修改描述~~

### 以下是源码

```go
package main

import (
	"errors"
	"flag"
	"fmt"
	"os"
	"path/filepath"
	"strconv"
	"strings"

	"os/exec"
)

var (
	dir           string
	commitMessage string
)

func init() {
	defaultDir := getCurrentDir()
	flag.StringVar(&dir, "d", defaultDir, "指定执行命令的文件夹路径，必须为绝对路径。")

	flag.StringVar(&commitMessage, "m", "updated", "指定本次提交的描述内容。")

	flag.Usage = func() {
		fmt.Fprintf(os.Stderr, `Usage: push2git [-d 指定git目录] [-m 提交描述]`)
	}
}

func main() {
	/****
	    依次执行以下命令:
		git remote get-url --push origin
		git status -s
		git add .
		git commit -m "update"
		git push
	 ****/

    // 解析命令行参数
	flag.Parse()

	_, err := os.Stat(dir)
	if (err != nil) && os.IsNotExist(err) {
		showMessage("执行命令文件夹不存在：" + dir)
		return
	}

	fmt.Println("执行命令文件夹：" + dir)

	content, err := execCommand("git remote get-url --push origin")
	if err != nil {
		showMessage("")
		return
	}
	// git在Mac、Windows上该命令的错误提示不一致，Not Or not
	// 所以这里全部按小写判断
	if strings.Contains(strings.ToLower(content), "not a git repository") {
		showMessage("当前目录不是git仓库目录。")
		return
	}
	fmt.Println("远程git地址为：" + content)

	content, err = execCommand("git status -s")
	if err != nil {
		showMessage("")
		return
	}
	if len(content) == 0 {
		showMessage("没有找到修改文件，不需要推送。")
		return
	}
	filesLen := len(strings.Split(content, "\n"))
	fmt.Println(strconv.Itoa(filesLen) + "个文件被修改。")

	content, err = execCommand("git add .")
	if err != nil {
		showMessage("")
		return
	}
	fmt.Println("添加到暂存区成功。")

	content, err = execCommand("git commit -m \"" + commitMessage + "\"")
	if err != nil {
		showMessage("")
		return
	}
	fmt.Println("提交到本地存储库成功。")

	fmt.Println("开始推送到远程服务器......")
	content, err = execCommand("git push")
	if err != nil {
		showMessage("")
		return
	}
	// 推送错误，提示出来
	if strings.Contains(content, "error:") {
		showMessage(content)
		return
	}

	showMessage("推送到远程服务器成功。")
}

// 执行命令，并去除掉前后的空行
func execCommand(input string) (string, error) {
	path, args, _ := resolveCommand(input)

	cmd := exec.Command(path, args...)
	cmd.Dir = dir
	output, err := cmd.CombinedOutput()
	result := strings.Trim(string(output), "\n")
	if err != nil {
		fmt.Println("执行命令发生错误：" + input)
		fmt.Println("异常内容：" + err.Error())
		fmt.Println("执行结果：" + result)
	}
	return result, err
}

// 解析命令，按空格拆分
func resolveCommand(input string) (string, []string, error) {
	if len(input) == 0 {
		return "", nil, errors.New("参数错误：" + input)
	}

	array := strings.Split(input, " ")
	return array[0], array[1:], nil
}

// 提示消息到控制台
func showMessage(content string) {
	if len(content) > 0 {
		fmt.Println(content)
	}
	fmt.Println("按回车退出程序......")
	fmt.Scanln()
}

// 获取当前执行文件的文件夹路径
func getCurrentDir() string {
	dir, _ := filepath.Abs(filepath.Dir(os.Args[0]))
	return dir
}

```