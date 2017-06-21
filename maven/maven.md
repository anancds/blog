maven有效地帮助我们管理项目的整个生命周期，包括项目清理、编译、构建、测试、打包、安装、发布、生成报告等，并且通过依赖和插件管理，实现开发环境的统一，避免冲突及管理无序现象。

## 生命周期
Maven的核心思想基于生命周期管理。这意味着构建和发布一个项目的步骤是有很清晰的定义的。构建项目的人，只需要知道一小部分构建命令和POM配置，以获得期望结果。
Maven内置3种生命周期：
* Default： 默认生命周期处理项目发布。
* Clean： 负责项目清理。
* Site： 负责项目文档发布。

还有一些别的生命周期：
* Validate： 确定项目是否配置正确，必要资源是否可用。
* Compiler： 编译项目源码。
* test： 使用合适的单元测试框架，测试编译结果。测试要求代码被打包或者发布。
* package: 讲代码打包成发行版本，如：jar。
* verify: 运行集成测试检验，确保软件质量达标。
* install： 安装包到本地仓库，作为其他本地项目依赖。
* deploy: 完成所有构建、打包阶段，发布到远程仓库。

生命周期汇总：

    validate,

    initialize,

    generate-sources,

    process-sources,

    generate-resources,

    process-resources,

    compile,

    process-classes,

    generate-test-sources,

    process-test-sources,

    generate-test-resources,

    process-test-resources,

    test-compile,

    process-test-classes,

    test,

    prepare-package,

    package,

    pre-integration-test,

    integration-test,

    post-integration-test,

    verify,

    install,

    deploy,

    pre-clean,

    clean,

    post-clean,

    pre-site,

    site,

    post-site,

    site-deploy

## 两个文件
### pom.xml

该文件是maven项目的核心，POM（Project Object Model项目对象模型）包含了一个project所需要的所有信息，独立于实际代码，避免了两者之间互相影响。文件中定义了项目的基本信息，描述项目如何构建，申明项目依赖等。同时，对插件的引用也是在此文件中申明，包括项目构建过程中所需要的插件的配置信息。

### setting.xml

默认文件的位置为\${M2_HOME}/conf/setting.xml,是全局配置，我们通常将它复制到${user.home}/.m2/setting.xml，为当前用户配置。如果两者都存在，以用户范围的settings.xml优先权更高。

## 坐标与依赖

### 坐标元素

一组maven坐标包括的坐标元素有：groupId、artifactId、version、packaging、classifier。

*  groupId——定义当前Maven项目隶属的实际项目。
*  artifactId——定义实际项目中的一个Maven模块。
* version——当前Maven项目的版本。
* packaging——打包方式。
* classifier ——用来帮助定义构建输出的一些附属构件。
举例如下：


    <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-project</artifactId>
      <version>2.7.2</version>
      <description>Apache Hadoop Project POM</description>
      <name>Apache Hadoop Project POM</name>
    <packaging>pom</packaging>

### 依赖
在<scope>字段配置依赖的范围，不同的配置分别控制依赖与classpath（编译classpath，测试classpath，运行classpath）之间不同的关系。

* compile: 表示dependency都可以在生命周期中使用。 而且，这些dependencies 会传递到依赖的项目中。适用于所有阶段，会随着项目一起发布。
* provided： 表明了dependency 由JDK或者容器提供，例如Servlet  AP和一些Java EE APIs。这个scope 只能作用在编译和测试时，同时没有传递性。
* runtime: 表示dependency不作用在编译时，但会作用在运行和测试时，如JDBC驱动，适用运行和测试阶段。
* test: 表示dependency作用在测试时，不作用在运行时。  只在测试时使用，用于编译和运行测试代码。不会随项目发布。
* system: 跟provided  相似，但是在系统中要以外部JAR包的形式提供，maven不会在repository查找它。

### 依赖排除

可通过exclusions标签，排除不需要的依赖，通过一下配置即可排除spark中的jersey-server依赖。依赖排除是dependency级别，而非pom文件，也即，只要在一个dependency中排除了该依赖，其他地方调用则无需再配置同样的内容，仅需要groupId和artifactId标签即可。

    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-core_2.10</artifactId>
        <version>${spark.version}</version>
        <exclusions>
            <exclusion>
                <groupId>com.sun.jersey</groupId>
                <artifactId>jersey-server</artifactId>
            </exclusion>
        </exclusions>
    </dependency>


  通过dependencyManagement可以指定我们使用的软件包版本，通常情况下我们仅指定了自己引入的包版本。但实际上，dependencyManagement可以将版本依赖传递到引入包的依赖上面，同时可以统一多个版本。

### 配置标签Properties
Properties 标签下可以有多个自定义名称的property配置，这类配置可以被子项目继承，通常配置在父项目中，用于全局配置或者版本约束。
配置示例：

    <project>
    ...
    <properties>
    <scala.version>2.11.8</scala.version>
    </properties>
    ...
    </project>


### 构建标签build

#### finalName

finalName标签可定义最终打包的jar名称，在maven-jar-plugin插件中，会自动读取该属性，jar打包生成的包名前缀即读取自finalName标签。

#### pluginManagement

插件管理有子标签 plugins，形如插件调用方式。插件管理目的主要有：版本统一，简化通用配置。
通过version标签，可以统一插件版本，并且在调用的时候无需重复指定版本。
通过configuration标签，可进行通用的配置，在调用的时候会自动继承配置属性。

    <pluginManagement>
          <plugins>
            <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-dependency-plugin</artifactId>
              <version>${maven-dependency-plugin.version}</version>
            </plugin>
            <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-enforcer-plugin</artifactId>
              <version>${maven-enforcer-plugin.version}</version>
              <configuration>
                <rules>
                  <requireMavenVersion>
                    <version>[3.0.2,)</version>
                  </requireMavenVersion>
                  <requireJavaVersion>
                    <version>[1.7,)</version>
                  </requireJavaVersion>
                </rules>
              </configuration>
            </plugin>
          </plugins>
        </pluginManagement>
