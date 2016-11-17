# JDK源码分析6-Observer、Observable

## 定义

观察者模式是对象的行为模式，又叫发布-订阅(Publish/Subscribe)模式、模型-视图(Model/View)模式、源-监听器(Source/Listener)模式或从属者(Dependents)模式。
观察者模式定义了一种一对多的依赖关系，让多个观察者对象同时监听某一个主题对象。这个主题对象在状态上发生变化时，会通知所有观察者对象，使它们能够自动更新自己。

### 意图

定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

### 主要解决

一个对象状态改变给其他对象通知的问题，而且要考虑到易用和低耦合，保证高度的协作。

### 何时使用

一个对象（目标对象）的状态发生改变，所有的依赖对象（观察者对象）都将得到通知，进行广播通知。

### 使用场景

* 有多个子类共有的方法，且逻辑相同。
* 重要的、复杂的方法，可以考虑作为模板方法。

### 使用观察者模式的好处

观察者模式（Observer）完美的将观察者和被观察的对象分离开。举个例子，用户界面可以作为一个观察者，业务数据是被观察者，用户界面观察业务数据的变化，发现数据变化后，就显示在界面上。面向对象设计的一个原则是：系统中的每个类将重点放在某一个功能上，而不是其他方面。一个对象只做一件事情，并且将他做好。观察者模式在模块之间划定了清晰的界限，提高了应用程序的可维护性和重用性。

### 注意事项

* JAVA 中已经有了对观察者模式的支持类。
* 避免循环引用。
* 如果顺序执行，某一观察者错误会导致系统卡壳，一般采用异步方式。

### 实现

观察者模式有很多实现方式，从根本上说，该模式必须包含两个角色：观察者和被观察对象。在刚才的例子中，业务数据是被观察对象，用户界面是观察者。观察者和被观察者之间存在“观察”的逻辑关联，当被观察者发生改变的时候，观察者就会观察到这样的变化，并且做出相应的响应。如果在用户界面、业务数据之间使用这样的观察过程，可以确保界面和数据之间划清界限，假定应用程序的需求发生变化，需要修改界面的表现，只需要重新构建一个用户界面，业务数据不需要发生变化。

### Observer和Observable

Observer是一个接口，就是一个观察者，源码中只有一个方法：update。当观察的目标改变时，被观察者就会调用这个方法。

Observable其实就是一个被观察者。源码中用Vector保存观察者，因为Vector是线程安全的，并且里面所有的方法都用synchronized关键字来保证线程安全。
源码中最关键的方法就是notifyobservers：

    //通知所有订阅此主题的观察者对象。
    //可以看出，通过bool类型变量changed的赋值，可以判断主题是否更新，以作为是否将信息推送给观察者的依据。changed的改变必需是线程安全的，
    //这也是为什么setChanegd方法和clearChanged方法都用synchronized关键字声明。
    //使用了Vector容器来保存观察者的引用，而不是ArrayList，同样是出于线程安全的考虑,ArrayList线程不安全，Vector线程安全。
    public void notifyObservers(Object arg) {
        /*
         * a temporary array buffer, used as a snapshot of the state of
         * current Observers.
         */
        Object[] arrLocal;

        //出于线程安全的需要，在将obs转为数组时，需要用同步控制块来做处理，这也是为了防止由于多个观察者线程并发对obs改变造成的线程异常，
        //尽管这里是线程安全的，但是jdk源码的注释中指出，这里有可能出现刚刚加入的观察者无法通知到更新或者刚刚删除的观察者接收到了不该接受到的消息的情况。
        synchronized (this) {
            /* We don't want the Observer doing callbacks into
             * arbitrary code while holding its own Monitor.
             * The code where we extract each Observable from
             * the Vector and store the state of the Observer
             * needs synchronization, but notifying observers
             * does not (should not).  The worst result of any
             * potential race-condition here is that:
             * 1) a newly-added Observer will miss a
             *   notification in progress
             * 2) a recently unregistered Observer will be
             *   wrongly notified when it doesn't care
             */
            if (!changed)
                return;
            arrLocal = obs.toArray();
            clearChanged();
        }

        for (int i = arrLocal.length-1; i>=0; i--)
            //调用所有观察者的update方法。
            ((Observer)arrLocal[i]).update(this, arg);
    }


### 示例
