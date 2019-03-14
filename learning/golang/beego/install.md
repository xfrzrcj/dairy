# **安装**
直接运行 `go get github.com/beego/bee` 此时 src/github.com/beego/bee 下会多了beego项目，并自动编译出 bee 执行文件文件。注意 bee 在 GOPATH/bin 下，最好将 GOPATH/bin 加入 PATH 中否则后续执行bee命令时容易出问题。  
至GOPATH下对应文件位置运行 `bee new appname`（创建web项目） 或  `bee api appname`,就会在当前目录下生成一个appname的项目，已经给了基本的项目模板。注意一定要到GOPATH下的位置执行，否则会提示无法生成项目。  
//TODO bee 命令解析 
