# volatile应用场景

关于volatile和synchronized的原理可以参考**Java并发编程的艺术**，写的非常清楚。
这里只讲应用场景，并用代码举例。

## 错误使用场景

    public class VolatileTest {

      public volatile static int count = 0;

      public static void inc() {
          try {
              Thread.sleep(10);
              } catch (InterruptedException e) {
                e.printStackTrace();
              }

              for (int i = 0; i < 100; i++) {
            count++;
          }
        }

        public static void main(String[] args) {
          new Thread(new Runnable() {
              @Override
              public void run() {
                inc();
              }
              }).start();

              VolatileTest.inc();
              System.out.println("the count is: " + count);
            }

          }

输出并不是想象中的200。因为如下图所示：
![](http://img.my.csdn.net/uploads/201302/14/1360809040_6000.JPG)

从这个图可以看出，

## volatile和synchronized的区别

synchronized表示代码具有原子性(atomicity)和可见性(visibility)。
volatile表示代码具有可见性。

所以要使用volatile要求很高，这里举出常用的使用场景。

## 状态标识
