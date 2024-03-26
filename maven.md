



```xml
<project>
    <dependencies>
        <dependency>
            <groupId></groupId>
            <artifactId></artifactId>
            <version></version>
            <type>...</type>
            <scope>...</scope>
            <optional>...</optional>
            <exclusions>
                <exclusion>
                  <groupId>...</groupId>
                  <artifactId>...</artifactId>
                </exclusion>
          </exclusions>
        </dependency>
      </dependencies>
</project>
```



**配置说明**：

- dependencies：一个 pom.xml 文件中只能存在一个这样的标签，是用来管理依赖的总标签。
- dependency：包含在 dependencies 标签中，可以有多个，每一个表示项目的一个依赖。
- groupId,artifactId,version(必要)：依赖的基本坐标，对于任何一个依赖来说，基本坐标是最重要的，Maven 根据坐标才能找到需要的依赖。我们在上面解释过这些元素的具体意思，这里就不重复提了。
- type(可选)：依赖的类型，对应于项目坐标定义的 packaging。大部分情况下，该元素不必声明，其默认值是 jar。
- scope(可选)：依赖的范围，默认值是 compile。
- optional(可选)：标记依赖是否可选
- exclusions(可选)：用来排除传递性依赖,例如 jar 包冲突



# 坐标



**groupId**(必须): 定义了当前 Maven 项目隶属的组织或公司	

**artifactId**(必须)：定义了当前 Maven 项目的名称，项目的唯一的标识符，对应项目根目录的名称。

**version**(必须)：定义了 Maven 项目当前所处版本。

**packaging**（可选）：定义了 Maven 项目的打包方式（比如 jar，war...），默认使用 jar。

**classifier**(可选)：常用于区分从同一 POM 构建的具有不同内容的构件，可以是任意的字符串，附加在版本号之后

















# 依赖范围

**classpath** 用于指定 `.class` 文件存放的位置，类加载器会从该路径中加载所需的 `.class` 文件到内存中。

Maven 在编译、执行测试、实际运行有着三套不同的 classpath：

- **编译 classpath**：编译主代码有效
- **测试 classpath**：编译、运行测试代码有效
- **运行 classpath**：项目运行时有效

Maven 的依赖范围如下：

- **compile**：编译依赖范围（默认），使用此依赖范围对于编译、测试、运行三种都有效，即在编译、测试和运行的时候都要使用该依赖 Jar 包。
- **test**：测试依赖范围，从字面意思就可以知道此依赖范围只能用于测试，而在编译和运行项目时无法使用此类依赖，典型的是 JUnit，它只用于编译测试代码和运行测试代码的时候才需要。
- **provided**：此依赖范围，对于编译和测试有效，而对运行时无效。比如 `servlet-api.jar` 在 Tomcat 中已经提供了，我们只需要的是编译期提供而已。
- **runtime**：运行时依赖范围，对于测试和运行有效，但是在编译主代码时无效，典型的就是 JDBC 驱动实现。
- **system**：系统依赖范围，使用 system 范围的依赖时必须通过 systemPath 元素显示地指定依赖文件的路径，不依赖 Maven 仓库解析，所以可能会造成建构的不可移植



# 依赖传递性



==直接==依赖冲突: 

```sh
X(1.0)
X(2.0)
```

**后声明的优先**	相同groupId+artifactId的情况下, 后声明的依赖优先



==间接==依赖冲突: 两个依赖包间接引入了同一个依赖

```sh
链路一：A -> B -> C -> X(1.0)
链路二：A -> D -> X(2.0)
这两条依赖路径上都有X,并且X的版本号不同
```

**路径最短优先**	对于X来说, 链路二的路径最短, X(2.0)会被优先采用

**声明顺序优先**	在间接依赖路径长度相等的情况下, 先声明的会被优先采用



# 生命周期

Maven 定义了 3 个生命周期`META-INF/plexus/components.xml`：

- `default` 生命周期
- `clean`生命周期
- `site`生命周期

这3个生命周期是相互独立的, 并且包含多个子阶段(phase), 在执行某个阶段时, 会先执行它前面的阶段





## default

`default`生命周期是在没有任何关联插件的情况下定义的，是 Maven 的主要生命周期，用于构建应用程序，共包含 23 个阶段

例如执行mvn package, 就会依次执行validate->package的所有阶段, 所以打包的时候项目的代码能够自动编译

```xml
<phases>
  <!-- 验证项目是否正确，并且所有必要的信息可用于完成构建过程 -->
  <phase>validate</phase>
  <!-- 建立初始化状态，例如设置属性 -->
  <phase>initialize</phase>
  <!-- 生成要包含在编译阶段的源代码 -->
  <phase>generate-sources</phase>
  <!-- 处理源代码 -->
  <phase>process-sources</phase>
  <!-- 生成要包含在包中的资源 -->
  <phase>generate-resources</phase>
  <!-- 将资源复制并处理到目标目录中，为打包阶段做好准备。 -->
  <phase>process-resources</phase>
  <!-- 编译项目的源代码  -->
  <phase>compile</phase>
  <!-- 对编译生成的文件进行后处理，例如对 Java 类进行字节码增强/优化 -->
  <phase>process-classes</phase>
  <!-- 生成要包含在编译阶段的任何测试源代码 -->
  <phase>generate-test-sources</phase>
  <!-- 处理测试源代码 -->
  <phase>process-test-sources</phase>
  <!-- 生成要包含在编译阶段的测试源代码 -->
  <phase>generate-test-resources</phase>
  <!-- 处理从测试代码文件编译生成的文件 -->
  <phase>process-test-resources</phase>
  <!-- 编译测试源代码 -->
  <phase>test-compile</phase>
  <!-- 处理从测试代码文件编译生成的文件 -->
  <phase>process-test-classes</phase>
  <!-- 使用合适的单元测试框架（Junit 就是其中之一）运行测试 -->
  <phase>test</phase>
  <!-- 在实际打包之前，执行任何的必要的操作为打包做准备 -->
  <phase>prepare-package</phase>
  <!-- 获取已编译的代码并将其打包成可分发的格式，例如 JAR、WAR 或 EAR 文件 -->
  <phase>package</phase>
  <!-- 在执行集成测试之前执行所需的操作。 例如，设置所需的环境 -->
  <phase>pre-integration-test</phase>
  <!-- 处理并在必要时部署软件包到集成测试可以运行的环境 -->
  <phase>integration-test</phase>
  <!-- 执行集成测试后执行所需的操作。 例如，清理环境  -->
  <phase>post-integration-test</phase>
  <!-- 运行任何检查以验证打的包是否有效并符合质量标准。 -->
  <phase>verify</phase>
  <!-- 	将包安装到本地仓库中，可以作为本地其他项目的依赖 -->
  <phase>install</phase>
  <!-- 将最终的项目包复制到远程仓库中与其他开发者和项目共享 -->
  <phase>deploy</phase>
</phases>
```



## clean

包含 3 个阶段：

1. pre-clean
2. clean
3. post-clean

```xml
<phases>
  <!--  执行一些需要在clean之前完成的工作 -->
  <phase>pre-clean</phase>
  <!--  移除所有上一次构建生成的文件 -->
  <phase>clean</phase>
  <!--  执行一些需要在clean之后立刻完成的工作 -->
  <phase>post-clean</phase>
</phases>
<default-phases>
  <clean>
    org.apache.maven.plugins:maven-clean-plugin:2.5:clean
  </clean>
</default-phases>
```





### site 生命周期

建立和发布浏览仓库的网站，共包含 4 个阶段：

1. pre-site
2. site
3. post-site
4. site-deploy

```xml
<phases>
  <!--  执行一些需要在生成站点文档之前完成的工作 -->
  <phase>pre-site</phase>
  <!--  生成项目的站点文档作 -->
  <phase>site</phase>
  <!--  执行一些需要在生成站点文档之后完成的工作，并且为部署做准备 -->
  <phase>post-site</phase>
  <!--  将生成的站点文档部署到特定的服务器上 -->
  <phase>site-deploy</phase>
</phases>
<default-phases>
  <site>
    org.apache.maven.plugins:maven-site-plugin:3.3:site
  </site>
  <site-deploy>
    org.apache.maven.plugins:maven-site-plugin:3.3:deploy
  </site-deploy>
</default-phases>
```

















