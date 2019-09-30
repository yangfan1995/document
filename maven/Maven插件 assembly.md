# 项目打包和maven assembly插件

## 项目打包

针对Java项目来说，线上部署的时候都是将项目打成后缀为`.jar`或者`.war`的方式部署到服务器上。有些时候项目的代码中不仅仅用到了自己的代码，还会用到第三方的jar包或者公司内部封装的jar包，打包的时候就需要注意打包方式，是否将这些引用的jar一起打包到一个jar中，如果没有就会出现ClassNotFound的问题。

> 打包过程：编译->jar包合并->最终可运行jar

### java 默认打包方式

#### 代码编译

主要有两种方式，参数添加或者MANIFEST.MF配置方式，参数方式的话，如果引用了第三方包，就要在命令里添加各种参数比较繁琐，推荐第二种MANIFEST.MF配置方式。

1. 如果代码中不使用第三方的包可以直接使用`javac F:\assembly_test\src\main\java\com\icongtai\assembly\StringTest.java`，如果使用了第三方包，则需要添加`classpath`主动添加jar包的路径，`javac -encoding UTF-8 -classpath D:/maven/apache-maven-3.6.1/repo/org/apache/commons/commons-lang3/3.9/commons-lang3-3.9.jar F:\assembly_test\src\main\java\com\icongtai\assembly\StringTest.java`

2. 项目根目录下添加META-INF文件夹，新建MANIFEST.MF文件，添加

   ```
   Manifest-Version: 1.0
   Main-Class: com.icongtai.assembly.StringTest
   Class-Path: commons-lang3-3.9.jar
   
   ```

   使用命令`javac -encoding UTF-8 -cp D:/maven/apache-maven-3.6.1/repo/org/apache/commons/commons-lang3/3.9/commons-lang3-3.9.jar  F:\assembly_test\src\main\java\com\icongtai\assembly\StringTest.java`

   > 配置文件需要一个空行

#### 代码打包

第二种方式可以使用命令`jar -cvfm assemblyTest.jar src\main\java\META-INF\MANIFEST.MF F:\assembly_test\src\main\java\com\icongtai\assembly\StringTest.class`

### maven 插件打包

对比java打包方式，maven的打包方式更加的简单明了。

[插件官网](https://maven.apache.org/plugins/maven-assembly-plugin/)

#### 引用方式

在pom.xml文件`build->plugins`中添加assembly插件的配置

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <executions>

        <execution><!-- 配置执行器 -->
            <id>make-assembly</id>
            <phase>package</phase><!-- 绑定到package生命周期阶段上 -->
            <goals>
                <goal>single</goal><!-- 只运行一次 -->
            </goals>
        </execution>
    </executions>
    <configuration>
        <finalName>${project.name}</finalName>
        <archive>
            <manifest>
                <mainClass>com.icongtai.assembly.StringTest</mainClass>
            </manifest>
        </archive>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>

        <!--                    <descriptors>-->
        <!--                        <descriptor>src/main/assembly/package.xml</descriptor>&lt;!&ndash;配置描述文件路径&ndash;&gt;-->
        <!--                    </descriptors>-->
    </configuration>
</plugin>
```

该配置文件中主要的配置参数有一下几点

1. phase(阶段)

   声明周期是由阶段组成的，assembly的phase参数配置的就是该插件在运行到哪个阶段时候才会去触发该插件的操作。如当前配置的为package阶段，则运行`mvn package`命令时就会触发assembly插件的执行。

2. goal(目标)

   多个goal组成了一个阶段，是执行的最小单元。每个goal可以绑定在阶段上，也可以单独运行。

详细信息参考[官方配置文档](https://maven.apache.org/plugins/maven-assembly-plugin/single-mojo.html)

#### 打包方式

> note that the parameters `descriptors`, `descriptorRefs`, and `descriptorSourceDirectory` are disjoint.表示三种配置方式都可以，如果三种都配置了，会按照各自的配置进行打包，所以根据具体情况一般一种配置方式就可以满足。

##### assembly.xml配置文件

比如DataX使用assembly打包方式，是通过配置具体xml文件规定打包的路径和方式。

pom.xml文件

```xml
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <encoding>${project-sourceEncoding}</encoding>
    </configuration>
</plugin>
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <configuration>
        <descriptors>
            <descriptor>src/main/assembly/package.xml</descriptor>
        </descriptors>
        <finalName>datax</finalName>
    </configuration>
    <executions>
        <execution>
            <id>dwzip</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

其中descriptors标签指定了打包的配置文件`src/main/assembly/package.xml`，package.xml主要如下

```xml
	<fileSets>
		<fileSet>
			<directory>src/main/resources</directory>
			<includes>
				<include>plugin.json</include>
				<include>plugin_job_template.json</include>
			</includes>
			<outputDirectory>plugin/reader/hbase11sreader</outputDirectory>
		</fileSet>
		<fileSet>
			<directory>target/</directory>
			<includes>
				<include>hbase11sreader-0.0.1-SNAPSHOT.jar</include>
			</includes>
			<outputDirectory>plugin/reader/hbase11sreader</outputDirectory>
		</fileSet>
	</fileSets>

	<dependencySets>
		<dependencySet>
			<useProjectArtifact>false</useProjectArtifact>
			<outputDirectory>plugin/reader/hbase11sreader/libs</outputDirectory>
			<scope>runtime</scope>
		</dependencySet>
	</dependencySets>
```

1. fileSets指定了文件的打包路径，比如资源配置，sql，json之类文件
2. dependencySets则是指定依赖的jar包的打包路径和声明周期

##### pom.xml文件直接指定

这种方式是直接在pom文件中指定打包的方式，如下配置

```
<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <executions>

                    <execution><!-- 配置执行器 -->
                        <id>make-assembly</id>
                        <phase>package</phase><!-- 绑定到package生命周期阶段上 -->
                        <goals>
                            <goal>single</goal><!-- 只运行一次 -->
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <finalName>${project.name}</finalName>
                    <archive>
                        <manifest>
                            <mainClass>com.icongtai.assembly.StringTest</mainClass>
                        </manifest>
                    </archive>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
            </plugin>
```

配置文件中

1. manifest标签配置了项目的主程序入口，类似于java打包方式中MANIFEST.MF指定Main-Class，
2. descriptorRef标签配置了是否将项目依赖的jar打到一个最终的jar包中，类似于java打包方式中MANIFEST.MF指定Class-Path，主要value有 `bin`, `jar-with-dependencies`, `src`,`project`，
   1. `bin`：打包源码
   2.  `jar-with-dependencies`：打包源码和所依赖的包
   3.  `src`：src目录下所有文件
   4. `project`：当前项目路径下所有文件

当前项目就可以使用`mvn package`命令进行打包，就会出发assembly插件的运行，等待打包完成，jar包在target文件夹下

> `jar-with-dependencies`打包出来的会有一个后缀为with-dependencies.jar的包，是包含所依赖的jar包

1. ### 实现原理

   ####  源码编译

   该项目中使用了modello插件，通过xml，json等配置文件生成相关的Java code，可以通过`mvn modello:java`将源码中缺省的源码生成。

   ####  运行机制

   所有的maven插件都会继承一个maven最基础的类`AbstractMojo`，不过assembly多封装了一个基类进行操作，基类中会初始化一些基础的配置文件信息，比如项目的名称，是否跳过assembly等参数。

   1. 在`org.apache.maven.plugins.assembly.mojos.AbstractAssemblyMojo#execute`会调用具体的xml配置的信息。`assemblies = assemblyReader.readAssemblies( this );` 
   2. 遍历配置文件中每个设计好的assembly配置和项目配置的各种文件路径，分别添加到`LocatorStrategy`这个策略中，`org.apache.maven.plugins.assembly.io.DefaultAssemblyReader#mergeComponentWithAssembly`中对配置文件进行加载FileSet，dependencySet等设置。
   3. 遍历处理好的assembly数组，生成打包文件的名称，根据打包类型处理打包逻辑，通过`org.apache.maven.plugins.assembly.archive.DefaultAssemblyArchiver.createArchive`生成具体的打包文件，比如zip文件需要进行压缩处理，如果是jar则会处理配置清单之类的信息。
   4.  `org.apache.maven.project.DefaultMavenProjectHelper#attachArtifact(org.apache.maven.project.MavenProject, java.lang.String, java.lang.String, java.io.File)`方法判断，如果需要生成文件则直接输出文件，如果是pom类型，则会添加到项目的artifact上。
   5.  `org.apache.maven.project.DefaultMavenProjectHelper#attachArtifact(org.apache.maven.project.MavenProject, java.lang.String, java.lang.String, java.io.File)`进行关联，在执行指定phase或者goal时候触发。

