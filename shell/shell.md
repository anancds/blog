# shell脚本编程规范

当然最好的规范还是[google Shell Style Guide](https://google.github.io/styleguide/shell.xml)
关于shell脚本的课程的话就是：[Unix Tools & Scripting](http://www.cs.cornell.edu/courses/CS2043/2014sp/)
## 关于调试

* sh -x 可以调试
* ls 1>/dev/null 2>&1不利于调试，可以改成：>/dev/null 2>log
* ls 2>&1 | tee log
## 分隔长行

    activate some_very_long_option \
             some_other_option

## 分离复合命令

其实这里的复合命令就是指块语句，例如for/while循环, if分支结构等等。

    HEAD_KEYWORD parameters; BODY_BEGIN
      BODY_COMMANDS
    BODY_END

### if/then/elif/else分支语句

    if ...; then
      ...
    elif ...; then
      ...
    else
      ...
    fi

### for循环

    for f in /etc/*; do
      ...
    done

### while/until循环

    while [[ $answer != [YyNn] ]]; do
      ...
    done


### case分支语句

    case $input in
      hello)
        echo "You said hello"
      ;;
      bye)
        echo "You said bye"
        if foo; then
          bar
        fi
      ;;
      *)
        echo "You said something weird..."
      ;;
    esac


几点注意的地方：

* 如果不是100%需要，匹配部分左右的括号不需要写(例如写成hello)而不是(hello))
* 匹配模式与分支的终止符号;;位于同一缩进级别
* 分支内部的命令多缩进一层
* 尽管是可选的，这里还是把最后一个分支的终止符号也写上了

## 语法和编码指引

### 晦涩的语法结构

我们都喜欢一些晦涩的语法结构，因为它们很简洁。但是如果不是100%需要用到，尽量不要使用它们，否则大多数人无法理解你的代码。
所以有有时候，我们需要在代码的智能，效率与可读性之间找到一个平衡点。
如果你一定要使用这种语法结构，记得在用的地方写上一小段注释。

### 变量名

因为所有保留的变量名都是大写的，最安全的方法是仅使用小写字母作为变量名，例如读入用户的输入、循环变量等等……

* 变量名尽量选择小写字母
* 如果你使用大写的变量名，不要使用保留的变量名(一份不完全的列表参见SUS)
* 常量还是用大写字母，多个单词用下划线分隔

### 变量初始化

正如C语言一样，最好的处理是在变量声明的时候初始化。
用户可以将一个变量以环境变量的形式传递到脚本中。如果你盲目地假定你使用的所有变量都是未初始化的，其它人可以以环境变量的形式劫持一个变量。

    $ cat b.sh

    if [ -z "$var" ]; then
        echo "$var is not set"
        var=1
    fi

    echo "Now, var is equals to $var"
    var=2 sh b.sh
    Now, var is equals to 2

解决这个问题的方法很简单，将变量初始化：

    my_input=""
    my_array=()
    my_number=0

### 参数展开

除非你知道自己做的事情，请在参数展开的地方使用双引号

当然，也有一些地方并不需要使用双引号，例如：

*[[ ]]测试表达式内部是不会展开的
* 在case $WORD in语法中WORD也不会展开的
* 在变量赋值var=$WORD的地方也是不会展开的

是在这些地方使用引号并不会出错，如果你习惯于在每个可能展开参数的地方使用引号，你写得代码会很安全。

如果你要传递一个参数作为一个单词列表，你可以不使用引号，例如：

    list="one two three"

    # you MUST NOT quote $list here
    for word in $list; do
      ...
    done

### 函数名称

首字母小写的驼峰，与java的方法一样。

### 命令替换

正如文章[the article about command substitution [Bash Hackers Wiki]](http://wiki.bash-hackers.org/syntax/expansion/cmdsubst)中提及的，你应该使用$( .. )形式。
不过，如果可移植性是一个问题，你可能必须使用反引号的形式`...`。
在任何情况，如果其它展开或者单词分隔并不是你期望的，你应该将命令替换用双引号引起来。

### Eval命令

正如Greg据说的：“If eval is the answer, surely you are asking the wrong question.”。
避免它，除非绝对必要：

* eval can be your neckshot(可能是你的麻烦？)
* 很有可能有其它的方法来实现你需要的
* 如果可能，重新思考下脚本的工作过程，当eval的使用不可避免的时候
* 如果你实在需要使用，小心慎用

## 脚本基本结构

一个脚本的基本结构是这样的：

    #!SHEBANG

    CONFIGURATION_VARIABLES

    FUNCTION_DEFINITIONS

    MAIN_CODE

### Shebang

如果可能，请不要忘记shebang。
请小心使用/bin/sh作为shebang，在Linux系统中，/bin/sh就是Bash这是一个错误的观点。
于我而言，shebang有两个目的：

* 说明直接执行时以哪个解释器来执行
* 明确该脚本应该以哪个解释器来执行

### 配置变量

在这里，我将这一类变量——可以被用户更改的——叫做配置变量。
让这类变量容易找到，一般放在脚本的头部，给它们有意义的名称并且加上注释说明。正如上面说的，仅当你知道你为什么这么做的时候，才用大写的变量名形式，否则小写形式更加安全。

### 函数定义

所有函数定义应该在脚本主要代码执行之前，这样可以给人全局的印象，并且确保所有函数在使用之前它是已知的。
你应该使用可移植性高的函数定义形式，即不带function关键字的形式。

## 脚本行为和健壮性

* 当脚本检测到问题时尽早退出，以免执行潜在的问题
* 如果你需要用到的命令可能并没有安装在系统上，在脚本执行的时候最好检查命令是否存在并且提醒用户缺少什么
* 采用有意义的脚本返回值，例如0代码成功，1代码错误或者失败

## 其他

### 输出内容

* 如果脚本是交互是的，你可以将终端的内容在执行后保存下来
* 在屏幕中输出简单易理解的消息
* 使用颜色或者特别的前缀区分错误和警告信息
* 输出正常的内容到STDOUT，而输出错误、警告或者诊断的信息到STDERR
* 在日志文件中输出所有详细的信息

### 输入

不要盲目地假设任何事情，如果你希望用户输入一个数字，请在脚本中主动检查它是否真得是一个数字，检查头部是否包含0，等等。我们都应该知道这一点，用户仅仅是用户而不是程序员，他们会做他们想要的，而不是程序想要的。
