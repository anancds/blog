# JDK源码分析2-Integer

## 前言

看了Integer的源码后，Double、Float、Long、Short的源码大同小异。
先来个测试代码：

      import java.util.Properties;

      public class IntegerTest {

        private static void testEqual() {
          Integer i1 = 3;
          Integer i2 = 3;
          System.out.println(i1 == i2);

          Integer i3 = 120;
          Integer i4 = 120;
          System.out.println(i3 == i4);
        }

        private static void testInitInteger() {
          Integer i = new Integer(10);
          i = 5;
          //这里i输出是5，其实是创建了一个新的对象，而且这个新的对象是从Integer的cache中取出的。
          System.out.println(i);
        }

        private static void testGetIntegrer() {
          Properties properties = System.getProperties();
          properties.put("cds", "12345");
          Integer i = Integer.getInteger("cds", 111);
          System.out.println(i);
        }

        private static void testDecode() {
          Integer bigNum = Integer.decode("-2147483648");
          Integer decimal = Integer.decode("+10");
          Integer oct = Integer.decode("-010");
          Integer hex1 = Integer.decode("-0x10");
          Integer hex2 = Integer.decode("#10");

          System.out.println(bigNum);
          System.out.println(decimal);
          System.out.println(oct);
          System.out.println(hex1);
          System.out.println(hex2);
        }

        private static void testReverse() {
          System.out.println(Integer.reverse(4));
          System.out.println(Integer.reverseBytes(1));
        }

        private static void testToString() {
          System.out.println(Integer.toString(10, 8));
        }

        private static void testParseInt() {
          System.out.println(Integer.parseInt("0", 10));
          System.out.println(Integer.parseInt("-0", 10));
          System.out.println(Integer.parseInt("-FF", 16));
          System.out.println(Integer.parseInt("10101010", 2));
          System.out.println(Integer.parseInt("-2147483648", 10));
          System.out.println(Integer.parseInt("2147483647", 10));
        }

        public static void main(String[] args) {
          //        testEqual();
          //        testToString();
          //        testInitInteger();
          //        testGetIntegrer();
          //        testDecode();
          //        testParseInt();
          testReverse();
        }
      }


Integer源码需要一点点体系结构的知识，可以参考：https://en.wikipedia.org/wiki/Locality_of_reference

## 类定义

      public final class Integer extends Number implements Comparable<Integer>

从定义可以看出integer类不能被继承；实现了Comparable接口，但是只能和Integer比较。
在这个类的实现文档中提到了一本书：Hacker's Delight，这是本神书，有空的时候得学习下。

## 属性

      //这里的变量用二进制补码表示，取反加1，如果第一个数是1那么就是负数，如果是0就是正数。
      //@native的意思就是这个字段有可能被本地底层代码引用，也就是c或者c++代码有可能会用到这个变量。
      //再具体点的话请参考stackoverflow的文章：
      //http://stackoverflow.com/questions/6101311/what-is-the-native-keyword-in-java-for
      //http://programmers.stackexchange.com/questions/218538/why-would-someone-use-native-annotations
      @Native public static final int   MIN_VALUE = 0x80000000;           //可以表示-2的31次方    即-2147483648
      @Native public static final int   MAX_VALUE = 0x7fffffff;           //可以表示2的31次方 - 1 即2147483647

      //这里有一个不安全的类型转换，所以需要抑制编译器来检查
      //基本类型int的Class实例，这个在学习Class类代码的时候再讲。
      @SuppressWarnings("unchecked")
      public static final Class<Integer>  TYPE = (Class<Integer>) Class.getPrimitiveClass("int");

      //Integer所表示的值就存储在这个变量中。注：在line 840
      //关于这个字段的文档说明中有个@serial，说明这个字段是会被序列化的。(不加static也不是transient，当然会被序列化啦！)
      private final int value;

## 静态内部类

      private static class IntegerCache {
        static final int low = -128;
        static final int high;
        //实现缓存策略
        static final Integer cache[];

        //下面的静态代码只是为了求出high这个值，默认是127
        static {
          // high value may be configured by property
          int h = 127;
          String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
              try {
                  int i = parseInt(integerCacheHighPropValue);
                  i = Math.max(i, 127);
                  // Maximum array size is Integer.MAX_VALUE
                  h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                  } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int,  ignore it.
                  }
                }
                high = h;

                cache = new Integer[(high - low) + 1];
                int j = low;
                for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

                // range [-128, 127] must be interned (JLS7 5.1.7)
                assert IntegerCache.high >= 127;
              }

              private IntegerCache() {}
              }


## String转Integer或者int

### getInteger

首先分析getInteger，看调用关系，关键是decode函数，这个函数的作用就是讲String转换成Integer，但是只支持十进制、十六进制、八进制。

    //nm由三部分组成：符号、基数说明符和字符序列，符号就是加好减号
    //基数说明符(0表示八进制，0x,0X，#表示十六进制，什么都不写则表示十进制)
      public static Integer decode(String nm) throws NumberFormatException {
          int radix = 10;
          int index = 0;
          boolean negative = false;
          Integer result;

          if (nm.length() == 0)
              throw new NumberFormatException("Zero length string");
          char firstChar = nm.charAt(0);
          // Handle sign, if present
          //先判断符号
          if (firstChar == '-') {
              negative = true;
              index++;
          } else if (firstChar == '+')
              index++;

          // Handle radix specifier, if present
          //再判断基数，即radix，这个顺序不能颠倒，特别是0x和0
          if (nm.startsWith("0x", index) || nm.startsWith("0X", index)) {
              index += 2;
              radix = 16;
          }
          else if (nm.startsWith("#", index)) {
              index ++;
              radix = 16;
          }
          else if (nm.startsWith("0", index) && nm.length() > 1 + index) {
              index ++;
              radix = 8;
          }

          //符号出现的位置出错，那么就抛出异常
          if (nm.startsWith("-", index) || nm.startsWith("+", index))
              throw new NumberFormatException("Sign character in wrong position");

          //取出实际的数字，然后加上符号
          try {
              result = Integer.valueOf(nm.substring(index), radix);
              result = negative ? Integer.valueOf(-result.intValue()) : result;
          } catch (NumberFormatException e) {
              // If number is Integer.MIN_VALUE, we'll end up here. The next line
              // handles this case, and causes any genuine format error to be
              // rethrown.
              String constant = negative ? ("-" + nm.substring(index))
                                         : nm.substring(index);
              result = Integer.valueOf(constant, radix);
          }
          return result;
      }

### valueOf

如果入参是int型，那么先去Integer的cache中查找，如果找到就返回，如果没有找到，那么就new 一个Integer

    //先从缓存中查找
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }

如果入参是String，那么就转成10进制的int型。
如果入参是String，和int，那么就通过parseInt转换。

### parseInt

    int result = 0;
    boolean negative = false;
    int i = 0, len = s.length();
    int limit = -Integer.MAX_VALUE;
    int multmin;
    int digit;

    //更加ascii码表，比0字符小的字符如果想要转换成String，那么只有加号和减号是符合要求的，所以只要判断这两个符合即可。
    if (len > 0) {
        char firstChar = s.charAt(0);
        if (firstChar < '0') { // Possible leading "+" or "-"
            if (firstChar == '-') {
                negative = true;
                limit = Integer.MIN_VALUE;
            } else if (firstChar != '+')
                throw NumberFormatException.forInputString(s);

            if (len == 1) // Cannot have lone "+" or "-"
                throw NumberFormatException.forInputString(s);
            i++;
        }
        multmin = limit / radix;
        while (i < len) {
            // Accumulating negatively avoids surprises near MAX_VALUE
            digit = Character.digit(s.charAt(i++),radix);
            if (digit < 0) {
                throw NumberFormatException.forInputString(s);
            }
            if (result < multmin) {
                throw NumberFormatException.forInputString(s);
            }
            result * = radix;
            if (result < limit + digit) {
                throw NumberFormatException.forInputString(s);
            }
            result -= digit;
        }
    } else {
        throw NumberFormatException.forInputString(s);
    }
    return negative ? result : -result;

### parseUnsignedInt

java1.8后新增加的一个函数

    public static int parseUnsignedInt(String s, int radix)
                throws NumberFormatException {
        if (s == null)  {
            throw new NumberFormatException("null");
        }

        int len = s.length();
        if (len > 0) {
            char firstChar = s.charAt(0);
            //如果是负数，直接抛异常
            if (firstChar == '-') {
                throw new
                    NumberFormatException(String.format("Illegal leading minus sign " +
                                                       "on unsigned string %s.", s));
            } else {
                //如果这个无符号整数满足有符号整数的条件，那么就直接调用parseInt函数，否则就调用Long的parseLong了，下面的这个if语句是精髓。
                if (len <= 5 || // Integer.MAX_VALUE in Character.MAX_RADIX is 6 digits
                    (radix == 10 && len <= 9) ) { // Integer.MAX_VALUE in base 10 is 10 digits
                    return parseInt(s, radix);
                } else {
                    long ell = Long.parseLong(s, radix);
                    //下面的十六进制的下划线的写法只是为了看着方便
                    if ((ell & 0xffff_ffff_0000_0000L) == 0) {
                        return (int) ell;
                    } else {
                        throw new
                            NumberFormatException(String.format("String value %s exceeds " +
                                                                "range of unsigned int.", s));
                    }
                }
            }
        } else {
            throw NumberFormatException.forInputString(s);
        }
    }

### 总结

* parseInt方法返回的是基本类型int;
* 其它方法返回的是Integer;
* valueOf(String)方法调用的是valueOf(int)方法;
* 如果只需要返回一个基本类型，而不需要一个对象，可以直接使用Integert.parseInt("123");
* 如果需要一个对象，那么建议使用valueOf(),因为该方法可以借助缓存带来的好处。
* 如果是从系统配置中取值，那么就是用getInteger

## int或者Integer转String

这里的stringSize用到了一个数组sizeTable，就是利用局部性空间原理。
代码中需要将负数转成整数，因为-2147483648转成整数会溢出，所以最小负数需要单独的判断。

//JDK作者对于10进制数转字符串单独实现，为的是提高性能，因为一般人的想法无非就是循环对10求余，得到对应的char型数组后就得到了最后的字符串，那么我们来看看作者的实现思路。

      public static String toString(int i) {
      //这里把最小值单独拿出来是因为执行下面的方法会溢出。
          if (i == Integer.MIN_VALUE)
              return "-2147483648";
          int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);
          char[] buf = new char[size];
          getChars(i, size, buf);
          return new String(buf, true);
      }

getChars函数是关键，有两个问题：
* 为什么在getChars方法中，将整型数字写入到字符数组的过程中为什么按照数字65536分成了两部分呢？这个65535是怎么来的？
* 在上面两段代码的部分二中，在对i进行除十操作的过程中为什么选择先乘以52429在向右移位19位。其中52429和19是怎么来的？

先理解以下代码：
r = i - ((q << 6) + (q << 5) + (q << 2));表示的其实是r = i - (q * 100);，i-q*2^6 - q*2^5 - q*2^2= i-64q-32q-4q = i-100q。
q = (i * num2) >>> (num3);中，>>>表示无符号向右移位。代表的意义就是除以2^num3。 所以q = (i * 52429) >>> (16+3); 可以理解为：q = (i * 52429) / 524288;,那么就相当于 q= i * 0.1也就是q=i/10，这样通过乘法和向右以为的组合的形式代替了除法，能提高效率。
再来回答上面两个问题中，部分一和部分二中最大的区别就是部分一代码使用了除法，第二部分只使用了乘法和移位。因为乘法和移位的效率都要比除法高，所以第二部分单独使用了乘法加移位的方式来提高效率。那么为什么不都使用乘法加移位的形式呢？为什么大于num1(65536)的数字要使用除法呢？原因是int型变量最大不能超过(2^31-1)。如果使用一个太大的数字进行乘法加移位运算很容易导致溢出。那么为什么是65536这个数字呢？第二阶段用到的乘法的数字和移位的位数又是怎么来的呢？
我们再回答第二个问题。
既然我们要使用q = (i * num2) >>> (num3);的形式使用乘法和移位代替除法，那么n和m就要有这样的关系：

> num2= (2^num3 /10 +1)

只有这样才能保证(i * num2) >>> (num3)结果接近于0.1。
那么52429这个数是怎么来的呢?来看以下数据：

    2^10=1024, 103/1024=0.1005859375
    2^11=2048, 205/2048=0.10009765625
    2^12=4096, 410/4096=0.10009765625
    2^13=8192, 820/8192=0.10009765625
    2^14=16384, 1639/16384=0.10003662109375
    2^15=32768, 3277/32768=0.100006103515625
    2^16=65536, 6554/65536=0.100006103515625
    2^17=131072, 13108/131072=0.100006103515625
    2^18=262144, 26215/262144=0.10000228881835938
    2^19=524288, 52429/524288=0.10000038146972656
    2^20=1048576, 104858/1048576=0.1000003815
    2^21=2097152, 209716/2097152 = 0.1000003815
    2^22= 4194304, 419431/4194304= 0.1000001431

超过22的数字我就不列举了，因为如果num3越大，就会要求i比较小，因为必须保证(i * num2) >>> (num3)的过程不会因为溢出而导致数据不准确。那么是怎么敲定num1=65536,num2= 524288, num3=19的呢？ 这三个数字之间是有这样一个操作的：

> (num1 * num2) >>> num3

因为要保证该操作不能因为溢出导致数据不准确，所以num1和num2就相互约束。两个数的乘积是有一定范围的，不成超过这个范围，所以，num1增大，num2就要随之减小。
我觉得有以下几个原因：

* 52429/524288=0.10000038146972656精度足够高。
* 下一个精度较高的num2和num3的组合是419431和22。2^31/2^22 = 2^9 = 512。512这个数字实在是太小了。65536正好是2^16，一个整数占4个字节。65536正好占了2个字节，选定这样一个数字有利于CPU访问数据。

不知道有没有人发现，其实65536* 52429是超过了int的最大值的，一旦超过就要溢出，那么为什么还能保证（num1* num2）>>> num3能得到正确的结果呢？
这和>>>有关，因为>>>表示无符号右移，他会在忽略符号位，空位都以0补齐。
一个有符号的整数能表示的范围是-2147483648至2147483647，但是无符号的整数能表示的范围就是0-4,294,967,296（2^32），所以，只要保证num2*num3的值不超过2^32次方就可以了。65536是2^16,52429正好小于2^16,所以，他们的乘积在无符号向右移位就能保证数字的准确性。

getChars使用了的体系结构知识：

* 乘法比除法高效：q = ( i * 52429) >>> (16+3); => 约等于q0.1,但i52429是整数乘法器，结合位移避免除法。
* 重复利用计算结果:在获取r(i%100)时，充分利用了除法的结果，结合位移避免重复计算。
* 位移比乘法高效:r = i – (( q << 6) + ( q << 5) + ( q << 2)); = >等价于r = i – (q * 100);
* 局部性原理之空间局部性
(1).buf[–charPos] =DigitOnes[r];buf[–charPos] =DigitTens[r];通过查找数组，实现快速访问,避免除法计算
(2).buf [–charPos ] = digits [ r];

      static void getChars(int i, int index, char[] buf) {
          int q, r;
          int charPos = index;
          char sign = 0;

          if (i < 0) {
              sign = '-';
              i = -i;
          }

           // 每次循环过后，都会将i中的走后两位保存到字符数组buf中的最后两位中，读者可以将数字i设置为12345678测试一下，
           //第一次循环结束之后，buf[7] = 8,buf[6]=7。第二次循环结束之后，buf[5] = 6,buf[4] = 5。
          while (i >= 65536) {
              q = i / 100;
          // really: r = i - (q * 100);
              r = i - ((q << 6) + (q << 5) + (q << 2));
              i = q;
              //取DigitOnes[r]的目的其实取数字r%10的结果
              buf [--charPos] = DigitOnes[r];
              //取DigitTens[r]的目的其实是取数字r/10的结果
              buf [--charPos] = DigitTens[r];
          }

          // Fall thru to fast mode for smaller numbers
          // assert(i <= 65536, i);
          //循环将其他数字存入字符数组中空余位置
          for (;;) {
                //这里其实就是除以10。取数52429和16+3的原因在后文分析。
              q = (i * 52429) >>> (16+3);
              // r = i-(q*10) ...
              r = i - ((q << 3) + (q << 1));  
              //将数字i的最后一位存入字符数组，
              //还是12345678那个例子，这个for循环第一次结束后，buf[3]=4。
              buf [--charPos] = digits [r];
              i = q;
              //for循环结束后，buf内容为“12345678”；
              if (i == 0) break;
          }
          if (sign != 0) {
              buf [--charPos] = sign;
          }
      }
