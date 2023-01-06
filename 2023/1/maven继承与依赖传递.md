# 继承元素
**Maven只支持单继承**
1. groupId: 项目组ID，项目坐标的核心元素
2. version: 项目版本，项目坐标的核心元素
3. description: 项目的描述信息
4. organization: 项目的组织信息
5. properties: 自定义的maven属性
6. dependencies: 项目的依赖配置
7. repositories: 项目的仓库配置
8. build: 包括项目的源码目录配置，输出目录配置，插件配置，插件管理配置等
9. reporting: 包括项目的报告输出目录配置，报告插件配置等
10. dependencyManagement: 项目的依赖管理配置
11. pluginManagement: 插件管理

# dependencyManagement
Maven使用该元素来提供一种管理依赖版本号的方式。通常会在父项目的pom文件中出现。使用该元素可以直接在**所有**子项目中直接引用依赖而不用显示的指明版本号。Maven会沿着父子层级关系向上寻找依赖，直到找到一个拥有dependencyManagement的项目，在其中找到子项目引用的依赖的版本号。

**注意：** dependencyManagement只是声明依赖，并不实现引入，因此子项目必须显示声明需要的依赖。

# pluginManagement
plugin可以用于两个元素下，分别是pluginManagement和build。
这两者的区别在于，只有当子项目pom文件引用了某插件，pluginManagement下的对应插件才会生效。

**注意：** 有些插件会被自动调用，比如将resources插件写在pluginManagement下，无论子项目有没有使用resources插件，它都会剩下，因为process-resources阶段已经绑定并调用它。

# 依赖传递
依赖具有传递性。A->B，B->C  =》A->C

# 依赖冲突
Maven中有两种方式来避免依赖冲突：
## 短路优先
本项目->A.jar->B.jar->D.jar
本项目->C.jar->D.jar
此时只会选择C->D这条路线。

## 声明优先
若引用路径长度相同，在pom中先声明的先被引用。

# 其他乱七八糟的点
## maven依赖中的type和scope
```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-dependencies</artifactId>
	<version>${spring-boot.version}</version>
	<type>pom</type>
	<scope>import</scope>
</dependency>
```
### type

type定义引入包的方式，默认是jar，可以改成war和pom。
+ jar: 项目会被打包成jar包
+ war: 项目会被打包成war包
+ pom: 用在父类项目或聚合项目，用于jar包管理。父类打成pom包的意义在于：父工程不写代码 代码都在子工程里写， 只在父类工程里写pom.xml里写jar的版本，控制子工程所需依赖的版本。

### scope

scope控制dependency元素的使用范围
+ compile（default）： 被依赖的jar需要参与到项目的编译，测试，打包，运行等阶段，打包时经常会包含被依赖的jar，是比较强的依赖。
+ provider： 被依赖的jar可以参与到项目的编译，测试，运行等阶段，但是在打包时不会包含该jar。场景：web项目中依赖的jar包，编译时需要，但是在web容器中运行时，web容器已提供该jar包。
+ runtime：编译时不需要，但是会参与到测试和运行阶段。场景：编译时不需要jdbc驱动包，但是运行时需要。
+ test：表示jar包会参与测试阶段。场景：Junit测试。
+ system：与provided类似，但是依赖不会从maven库中查找，而是从本地系统中获取，systemPath元素用于指定依赖在系统中jar的路径。
```
<dependency>
    <groupId>com.cjf</groupId>
    <artifactId>auth-center</artifactId>
    <version>1.5</version>
    <scope>system</scope>
    <systemPath>${basedir}/MyContent/WEB-INF/lib/auth-center.jar</systemPath>
</dependency>
```
+ import：只用于dependencyManagement中。由于Maven单继承问题，如果子项目需要继承多个父项目，则需要用到import。可以把dependencyManagement放到一个专门用来管理依赖版本的pom中，然后在需要用到该依赖配置的pom中使用<scope>import</scope>引用依赖。

# Reference
https://blog.csdn.net/fxkcsdn/article/details/97620503?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-97620503-blog-126525380.pc_relevant_3mothn_strategy_and_data_recovery&spm=1001.2101.3001.4242.1&utm_relevant_index=3
