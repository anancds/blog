# JDK源码分析7-Iterable

Iterable这个接口只有三个方法，其中

    Iterator<T> iterator();

这个方法其实就是为了实现java的foreach这种语法。测试代码和反编译的代码如下，用javap -c：

    List<String> list = new ArrayList<>(Arrays.asList("a", "b"));
    for (String s : list) {
        System.out.println(s);
    }

    Iterator iterator = list.iterator();
    if (iterator.hasNext()) {
        System.out.println(iterator.next());
    }

反汇编后的字节码:

    Code:
       0: new           #29                 // class java/util/ArrayList
       3: dup
       4: iconst_2
       5: anewarray     #30                 // class java/lang/String
       8: dup
       9: iconst_0
      10: ldc           #31                 // String a
      12: aastore
      13: dup
      14: iconst_1
      15: ldc           #32                 // String b
      17: aastore
      18: invokestatic  #33                 // Method java/util/Arrays.asList:([Ljava/lang/Object;)Ljava/util/List;
      21: invokespecial #34                 // Method java/util/ArrayList."<init>":(Ljava/util/Collection;)V
      24: astore_1
      25: aload_1
      26: invokeinterface #35,  1           // InterfaceMethod java/util/List.iterator:()Ljava/util/Iterator;
      31: astore_2
      32: aload_2
      33: invokeinterface #36,  1           // InterfaceMethod java/util/Iterator.hasNext:()Z18
      38: ifeq          61
      41: aload_2
      42: invokeinterface #37,  1           // InterfaceMethod java/util/Iterator.next:()Ljava/lang/Object;
      47: checkcast     #30                 // class java/lang/String
      50: astore_3
      51: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
      54: aload_3
      55: invokevirtual #8                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      58: goto          32
      61: aload_1
      62: invokeinterface #35,  1           // InterfaceMethod java/util/List.iterator:()Ljava/util/Iterator;
      67: astore_2
      68: aload_2
      69: invokeinterface #36,  1           // InterfaceMethod java/util/Iterator.hasNext:()Z
      74: ifeq          89
      77: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
      80: aload_2
      81: invokeinterface #37,  1           // InterfaceMethod java/util/Iterator.next:()Ljava/lang/Object;
      86: invokevirtual #38                 // Method java/io/PrintStream.println:(Ljava/lang/Object;)V
      89: return

我们看到18行到51行和 55行到86行原来是一样的，也就是jvm把foreach这种语法结构解释成了iterator。
