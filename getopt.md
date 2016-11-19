# C语言中的getopt函数

## 函数原型介绍

首先需要包含#include <unistd.h>

函数原型是： int getopt(int argc, char * const argv[], const char * optstring);

第一个参数就是参数个数，第二个参数就是指向参数的数组，第三个参数就是就是规定的参数选项。每调用一次就返回一个参数选项。

字符串optstring可以下列元素：

* 单个字符，表示选项，
* 单个字符后接一个冒号：表示该选项后必须跟一个参数。参数紧跟在选项后或者以空格隔开。该参数的指针赋给optarg。
* 单个字符后跟两个冒号，表示该选项后必须跟一个参数。参数必须紧跟在选项后不能以空格隔开。该参数的指针赋给optarg。（这个特性是GNU的扩张）。

在这个头文件unistd.h中与getopt函数相关的一些参数的信息如下：

       extern char * optarg;  //选项的参数指针

       extern int optind,   //下一次调用getopt的时，从optind存储的位置处重新开始检查选项。 这里，程序名的位置是0，当前参数的位置是1，所以下一个参数的位置是2。

       extern int opterr,  //当opterr=0时，getopt不向stderr输出错误信息。

       extern int optopt;  //当命令行选项字符不包括在optstring中或者选项缺少必要的参数时，该选项存储在optopt中，getopt返回'？’

示例代码如下：

    #include <stdio.h>
    #include <unistd.h>
    int main(int argc,char *argv[])
    {
      int ch;
      opterr=0;

      while((ch=getopt(argc,argv,"a:b::cde"))!=-1)
      {
        printf("optind:%d\n",optind);
        printf("optarg:%s\n",optarg);
        printf("ch:%c\n",ch);
        switch(ch)
        {
          case 'a':
            printf("option a:'%s'\n",optarg);
            break;
          case 'b':
            printf("option b:'%s'\n",optarg);
            break;
          case 'c':
            printf("option c\n");
            break;
          case 'd':
            printf("option d\n");
            break;
          case 'e':
            printf("option e\n");
            break;
          default:
            printf("other option:%c\n",ch);
        }
        printf("optopt+%c\n",optopt);
      }
    }


当输入是-a1234 -b432 -c -d时，程序返回的结果：

    optind:2
    optarg:1234
    ch:a
    option a:'1234'
    optopt+
    optind:3
    optarg:432
    ch:b
    option b:'432'
    optopt+
    optind:4
    optarg:(null)
    ch:c
    option c
    optopt+
    optind:5
    optarg:(null)
    ch:d
    option d
    optopt+
