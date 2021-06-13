---
title: Golang基础：源码与包（3）
date: 2021-04-17 21:58:47
tags: golang
categories: 技术
abbrlink: 35349
---

#### import关键字

前面的文章中提到，包的概念的诞生是为了模块化。把一个项目封装成一个个相对独立的模块，使之互相隔离，然后又互相合作，可以明显提升项目的可读性、可维护性以及便于开发。在Golang中，模块之间的隔离是用“package”关键字做到的，而模块之间的合作，则需要“import”关键字。

“import”的意思是“引入”，当一个模块需要用到另一个模块的功能时，可以使用“import”关键字引用另一个模块。

##### 语法

```go
import "context"
import "fmt"

import (
	"strconv"
	"time"
)
```

import的语法很简单，就是用一个空格隔开import关键字本身和一个字符串，字符串的内容就是要引用的包的路径。每行只能引用一个包。

当需要引用多个模块时，也可以像下面那样，用括号将多个分行的包路径包含起来，不必每行写一个“import”关键字。

需要注意的是，import的是包的路径名称，而不是包名称，二者可以是不一致的；但是在代码中又必须使用包名，所以二者保持一致还是有好处的。

```shell
➜   pwd
/Users/mengshuai/projects/tmp/src
➜   cat pack/cmd.go
package main

import "test"

func main() {
	other.Other()
}
➜   cat test/other.go 
package other

func Other() {}
➜   go install pack   
➜   
```

包other在目录test中，cmd.go引入的时候用的也是“test”，而非“other”；代码中使用other包元素的时候，却只能用“other”，如果用“test”会报错：

```shell
➜   go install pack
# pack
pack/cmd.go:3:8: imported and not used: "test" as other
pack/cmd.go:6:2: undefined: test
```

##### 包的路径

包的路径实际上并不是在文件系统中的绝对路径，而只是绝对路径的后半部分，前半部分会被编译工具自动拼装——前半部分就是可能存放依赖包的位置。

```shell
➜   echo $GOPATH
/Users/mengshuai/programs/go:/Users/mengshuai/projects/tmp
➜   cat other/other.go
package other

import "test"

func other() {}
➜   go install other/other.go 
other/other.go:3:8: cannot find package "test" in any of:
	/usr/local/opt/go@1.12/libexec/src/test (from $GOROOT)
	/Users/mengshuai/programs/go/src/test (from $GOPATH)
	/Users/mengshuai/projects/tmp/src/test

```

代码中故意import了一个不存在的“test”包，然后编译报错了。根据报错信息，编译工具先去了\$GOROOT再去了\$GOPATH中寻找包体，且**\$GOPATH的搜索顺序是从后往前的**。

拼装路径的时候，也用到了Golang的工程目录约定，在\$GOROOT或$GOPATH后面添加了“src/”，再接包所在的直接目录。但是这里拼接“src”而不是“pkg”，是因为没有找到相应的二进制库文件，所以打算从源码开始编译；否则直接从“pkg”中寻找就可以了。下面验证一下：

```shell
➜   cat src/other/other.go 
package other

import "foo"

func Other() {
	foo.Main()
}
➜   cat src/foo/foo.go 
package foo

func Main() {
}
➜   ls pkg/darwin_amd64
➜   go install -v -x other   
WORK=/var/folders/c6/9j2_kn4x54bd2804lr0jbh8h0000gp/T/go-build366818321
foo
mkdir -p $WORK/b002/
cat >$WORK/b002/importcfg << 'EOF' # internal
# import config
EOF
cd /Users/mengshuai/projects/tmp/src/foo
/usr/local/opt/go@1.12/libexec/pkg/tool/darwin_amd64/compile -o $WORK/b002/_pkg_.a -trimpath $WORK/b002 -p foo -complete -buildid G5GykchxTZhfsFM9hggR/G5GykchxTZhfsFM9hggR -goversion go1.12.17 -D "" -importcfg $WORK/b002/importcfg -pack -c=4 ./foo.go
/usr/local/opt/go@1.12/libexec/pkg/tool/darwin_amd64/buildid -w $WORK/b002/_pkg_.a # internal
cp $WORK/b002/_pkg_.a /Users/mengshuai/Library/Caches/go-build/03/03235c6fe222835d860ad6bfec706a15d5724280635e318dce918acc9de61b33-d # internal
other
mkdir -p $WORK/b001/
cat >$WORK/b001/importcfg << 'EOF' # internal
# import config
packagefile foo=$WORK/b002/_pkg_.a
EOF
cd /Users/mengshuai/projects/tmp/src/other
/usr/local/opt/go@1.12/libexec/pkg/tool/darwin_amd64/compile -o $WORK/b001/_pkg_.a -trimpath $WORK/b001 -p other -complete -buildid xwd5LMAP4ULcw7IZiMOg/xwd5LMAP4ULcw7IZiMOg -goversion go1.12.17 -D "" -importcfg $WORK/b001/importcfg -pack -c=4 ./other.go
/usr/local/opt/go@1.12/libexec/pkg/tool/darwin_amd64/buildid -w $WORK/b001/_pkg_.a # internal
cp $WORK/b001/_pkg_.a /Users/mengshuai/Library/Caches/go-build/0f/0f3f115ca937e2bf6ae7840e78350195175b6f48e24d1b742e8f52874e031b67-d # internal
mkdir -p /Users/mengshuai/projects/tmp/pkg/darwin_amd64/
mv $WORK/b001/_pkg_.a /Users/mengshuai/projects/tmp/pkg/darwin_amd64/other.a
rm -r $WORK/b001/
➜   ls pkg/darwin_amd64
other.a
```

other.go引用了foo.go的代码，当pkg目录下没有foo包的库文件时，安装other包会先去源代码目录寻找foo包并编译（“go install”的参数“-x -v”可以显示出编译过程）：

“cd /Users/mengshuai/projects/tmp/src/foo”

如果foo.a已经存在，则直接使用即可，不会再去重新编译foo包，除非foo的代码发生了改变。如下：

```shell
➜   go clean -cache
➜   rm -rf pkg/darwin_amd64/*.a
➜   go install -v -x foo  
WORK=/var/folders/c6/9j2_kn4x54bd2804lr0jbh8h0000gp/T/go-build877130085
foo
mkdir -p $WORK/b001/
cat >$WORK/b001/importcfg << 'EOF' # internal
# import config
EOF
cd /Users/mengshuai/projects/tmp/src/foo
/usr/local/opt/go@1.12/libexec/pkg/tool/darwin_amd64/compile -o $WORK/b001/_pkg_.a -trimpath $WORK/b001 -p foo -complete -buildid G5GykchxTZhfsFM9hggR/G5GykchxTZhfsFM9hggR -goversion go1.12.17 -D "" -importcfg $WORK/b001/importcfg -pack -c=4 ./foo.go
/usr/local/opt/go@1.12/libexec/pkg/tool/darwin_amd64/buildid -w $WORK/b001/_pkg_.a # internal
cp $WORK/b001/_pkg_.a /Users/mengshuai/Library/Caches/go-build/03/03235c6fe222835d860ad6bfec706a15d5724280635e318dce918acc9de61b33-d # internal
mkdir -p /Users/mengshuai/projects/tmp/pkg/darwin_amd64/
mv $WORK/b001/_pkg_.a /Users/mengshuai/projects/tmp/pkg/darwin_amd64/foo.a
rm -r $WORK/b001/
➜   go install -v -x other
WORK=/var/folders/c6/9j2_kn4x54bd2804lr0jbh8h0000gp/T/go-build164428318
other
mkdir -p $WORK/b001/
cat >$WORK/b001/importcfg << 'EOF' # internal
# import config
packagefile foo=/Users/mengshuai/projects/tmp/pkg/darwin_amd64/foo.a
EOF
cd /Users/mengshuai/projects/tmp/src/other
/usr/local/opt/go@1.12/libexec/pkg/tool/darwin_amd64/compile -o $WORK/b001/_pkg_.a -trimpath $WORK/b001 -p other -complete -buildid xwd5LMAP4ULcw7IZiMOg/xwd5LMAP4ULcw7IZiMOg -goversion go1.12.17 -D "" -importcfg $WORK/b001/importcfg -pack -c=4 ./other.go
/usr/local/opt/go@1.12/libexec/pkg/tool/darwin_amd64/buildid -w $WORK/b001/_pkg_.a # internal
cp $WORK/b001/_pkg_.a /Users/mengshuai/Library/Caches/go-build/0f/0f3f115ca937e2bf6ae7840e78350195175b6f48e24d1b742e8f52874e031b67-d # internal
mkdir -p /Users/mengshuai/projects/tmp/pkg/darwin_amd64/
mv $WORK/b001/_pkg_.a /Users/mengshuai/projects/tmp/pkg/darwin_amd64/other.a
rm -r $WORK/b001/
```

使用“go clean -cache”清除编译缓存，并删除所有pkg目录下的库文件后，先安装foo包，那么在安装other包时，发现打印了这一行：
“packagefile foo=/Users/mengshuai/projects/tmp/pkg/darwin_amd64/foo.a”

说明此时没有去寻找源码，直接引用了编译好的库文件。

关于Golang编译缓存，可以参考[这里](https://tonybai.com/2018/02/17/some-changes-in-go-1-10/)。

#### 下载第三方依赖：go get

##### 原理

pkg目录下的库文件不一定非得是自己写的代码，也可以是别人写好的。互联网的世界里，“拿来主义”是常态。我们可以很方便的使用别人分享出来的代码，只要简单的使用“go get”命令从指定的url获取即可。这些url一般是业界比较知名的代码仓库托管平台，比如github，其他支持的平台包括：Google Code、BitBucket和Launchpad。

在内部实际上分成了两步操作：第一步是下载源码包，第二步是执行 go install。下载源码包的 go 工具会自动根据不同的域名调用不同的源码工具，对应关系如下：

BitBucket (Mercurial Git)
GitHub (Git)
Google Code Project Hosting (Git, Mercurial, Subversion)
Launchpad (Bazaar)

所以需要提前安装好git等版本管理工具。

##### 使用参数

​    示例：go get github.com/davyxu/cellnet

- -d 只下载不安装
- -f 只有在你包含了 -u 参数的时候才有效，不让 -u 去验证 import 中的每一个都已经获取了，这对于本地 fork 的包特别有用
- -fix 在获取源码之后先运行 fix，然后再去做其他的事情
- -t 同时也下载需要为运行测试所需要的包
- -u 强制使用网络去更新包和它的依赖包
- -v 显示执行的命令

##### 私有仓库

假设远程仓库的地址是：git.garena.com/

需要在~/.gitconfig中添加以下配置：

​    [url "[ssh://gitlab@git.garena.com:2222/](ssh://gitlab@git.garena.com:2222/)"]                           

​        insteadOf = https://git.garena.com/

完。