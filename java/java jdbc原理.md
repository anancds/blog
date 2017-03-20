### Java jdbc流程

Java应用程序访问数据库需要通过JDBC API方法、JDBC的数据库驱动才能完成，那么这些都分别由谁提供的呢？ 其中：JDBC API的主要功能有：建立数据库连接、执行SQL语句、处理查询结构等，相关的类和接口（斜体代表接口，需驱动程序提供者来具体实现）包括：

* DriverManager：负责加载各种不同驱动程序（Driver），并根据不同的请求，向调用者返回相应的数据库连接（Connection）
* Connection：数据库连接，负责与进行数据库间通讯，SQL执行以及事务处理都是在某个特定Connection环境中进行的。可以产生用以执行SQL的Statement。
* Statement：由 Connection 产生，用以执行SQL查询和更新（针对静态SQL语句和单次执行）。
* PreparedStatement：由Connectoin产生，用以执行包含动态参数的SQL查询和更新（在服务器端编译，允许重复执行以提高效率）。
* CallableStatement：用以调用数据库中的存储过程。
* SQLException：代表在数据库连接的创建和关闭和SQL语句的执行过程中发生了异常情况（即错误）。
* ResultSet：负责保存Statement执行后所产生的查询结果

首先加载数据库的JDBC驱动，然后建立数据库连接，通过Statement接口执行SQL语句，将查询的结果交给ResultSet进行处理，从而得到我们想要的数据。

* 注册驱动（Driver）
* 建立连接（创建Connection）
* 创建执行sql语句（通常是创建Statement或者其子类）
* 执行语句
* 处理执行结果（在非查询语句中，该步骤是可以省略的）
* 释放相关资源

### Statement与PreparedStatment区别

PreparedStatement是带预编译功能的，所以参数是sql字符串，但Statement是没有参数的。

用Statement相当于每次都把SQL语句发送给数据库编译执行，但是PrepareStatement在创建时就已经制定了SQL语句，这时就会把SQL语句发送给数据库编译，然后执行的时候就直接执行编译后的SQL语句了。好处就是当反复使用同一个SQL语句时，减少了编译时间，提高了效率。

当然这种方式也能防止了一部分的SQL注入。比如在PrepareStatement中可以有相关的过滤，比如过滤类型。而Statement是没有任何判断，直接拼接传入数据库，所以容易被SQL注入。
