[Gradlede 的版本库](http://services.gradle.org/distributions/)

[Gradle 官网](https://gradle.org/)

[Gradle DSL 教程](https://docs.gradle.org/current/dsl/)


# Gradle学习

构建，可以叫做build或make. 就是根据输入信息然后干一堆事情，然后得出几个产出物。

Gradle 就是一个构建工具，发展历史从`ANT->MAVEN->GRADLE`

Gradle 因为采用的Groovy语言，所以具备Groovy的特点DSL(`Domain Specific Language`)

学习Gradle需要掌握俩个点：1是Groovy的语法，2是Gradle的使用方法

Gradle版本在AS中的位置：项目名`\gradle\wrapper\gradle-wrapper.properties`


## 1.Gradle基本概念

Gradle脚本在被执行时,根据脚本作用不同会被配置一个特定的类型

- `build.gradle` 又被称作构建脚本

- 每一个待编译的工程都可以成为一个Project，每一个Project在构建的时候都包含一系列的Task，例如：一个Android APK的编译可能包含：Java源码编译Task，资源编译Task，JNI编译Task，Lint检查Task，打包生成apk的Task，签名的Task

- 一个Project包含多少个Task是由编译脚本添加的插件所决定的，插件就是定义了Task，并具体执行这些Task的东西

- **Gradle是一个框架，作为框架，它负责定义流程和规则。而具体的编译工作则是通过插件的方式来完成的**

- gradle为我们提供了许多默认的配置以及通常的默认值

- Gradle是一种基于Groovy的动态DSL，而Groovy是一种基于jvm的动态语言

- **方法中最后一个参数为Closure时，可以把闭包放在方法调用之后！（这是groovy特性）**
		
		//方法定义，以下三种方法都是一样的效果
		def method(Closure cl){
			cl()
		}
		//调用方式1
		method(){	
			println 'method--1'
		}
		//调用方式2
		method({println 'method--2'})
		//调用方式3
		method{println 'method--3'}


- gradle 自动帮开发者导入了 一大堆的库，gradle可以通过 tooling api 来标记不同的任务结果

	- 标签： no label or EXECUTED     说明：任务已经执行完成
	情况： 

		1. 当任务有动作且gradle确定task 是作为构建的一部分

	- 标签：UP-TO-DATE    说明：任务输出没有改变
	情况： 

	 	1. 当一个任务有输入有输出并且这些没有改变
	 	2. 当一个任务有action，并且task输出没有改变
	 	3. 当一个任务没有action，但是又 dpendencies 。并且这些dependencies 已经是UP-TO-DATE,SKIPPED or from CACHE
	 	4. 当一个任务没有acton，也没有dependencies

	- 标签:FROM-CACHE 说明：任务的输出是从之前的执行结果中获得
	情况:

		1. 当任务存有构建输出缓存

	- 标签：SKIPPED 说明：任务action'被跳过
	情况:

		1. 当一个任务被明确的从 命令行中排除在外
		2. 当一个任务 返回了 false

	- 标签：NO-SOURCE 说明：任务无需执行其action
	情况: 
		
		1. 任务有输入有输出，但是没有source ，例如：source files are .java files for JavaCompile


- Gradle文件包含了一些`Script Block`(Build script structure) 用来配置相关的信息。这些SB通常都是一个函数，并要求传入一个`Closure`

- **Groovy支持函数调用传入`map`,会自动转换格式**

		apply plugin:'com.android.library'

	- `参数名1：参数值1，参数名2：参数值2` 会被转成map


- 根据Groovy的语法，一个Property会自动生成get/set方法

### 1.1 Project和tasks和action
- 每个项目的编译至少有一个project

- 每个project至少有一个task,task里面又包含了很多action，action就是一个代码块，里面包含了需要被执行的代码

- `build.gradle`就是对应Project的编译脚本，**`project`和`build.gradle`是一种一对一的关系**

- 每一个Library和每一个module都是单独的Project。根据Gradle的要求，每一个Project在其根目录下都需要有一个build.gradle

- task在build.gradle中被定义


### 1.2 Gradle工作流程

在编译过程中， Gradle 会根据 build 相关文件，聚合所有的project和task，执行task 中的 action。因为` build.gradle`文件中的task非常多，先执行哪个后执行那个需要一种逻辑来保证。这种逻辑就是依赖逻辑，几乎所有的Task 都需要依赖其他 task 来执行，没有被依赖的task 会首先被执行。所以到最后所有的 Task 会构成一个 有向无环图（DAG Directed Acyclic Graph）的数据结构。

![](http://ww1.sinaimg.cn/large/6ab93b35gy1fjnug3z25hj20q7065t9d.jpg)

#### 1.2.1编译过程分为三个阶段

1.  **初始化阶段**：执行`settings.gradle`.**创建 Project 对象**，如果有多个`build.gradle`，也会创建多个project.

	- Hook:gradle.beforeProject{project->}

2.  **配置阶段**：在这个阶段，会解析每个Project中的`build.gradle`(执行所有的编译脚本)，同时还会创建project的所有的task，并创建一个有向图来描述Task之间的依赖关系.作为构建一部分的构建脚本会被执行

	- Hook:gradle.taskGraph.whenReady{graph->}

3.  **执行阶段**：在这个阶段，gradle 会根据传入的参数决定如何执行这些task（在配置阶段被创建和配置的）,真正action的执行代码就在这里`.gradle`按顺序执行每个任务。

	- Hook:gradle.buildFinished{result ->}

- Gradle有一个初始化流程,这个时候`settings.gradle`会执行

- 在配置阶段，每个Project都会被解析，其内部任务也被添加到一个有向图里，用于解决执行过程中的依赖关系

- 执行阶段，执行任务，gradle会将这个任务链上的所有任务按依赖顺序执行一遍


	//settings.gradle文件
	println "this is executed during the initialization phase"

	//根项目的build.gradle文件
	task configured{

		println "this is executed during the Configuration phase"

		doLast{
			println "this is executed during the Execution phase"
		}

		doFirst{
			println "this is executed during the Execution phase"
		}

	}


#### 1.2.2 Gradle编程模型
- Gradle Build Language Reference ：[Gradle各种模型的介绍](https://docs.gradle.org/current/dsl/)

- Gradle基于Groovy，Groovy基于java。所以Gradle执行的时候和Groovy一样，会把脚本转换成Java对象。

- Gradle主要有三种对象，这三种对象和三种不同的脚本文件对应，在gradle执行的时候，会将脚本转换成对应的对象：

	- **Gradle对象**：当我们执行gradle xxx或者什么的时候，gradle会从默认的配置脚本中构造出一个Gradle对象。在整个执行过程中，只有这么一个对象。Gradle对象的数据类型就是Gradle。我们一般很少去定制这个默认的配置脚本。

	- **Project对象**：每一个`build.gradle`会转换成一个Project对象。

	- **Setting对象**：每一个`settings.gradle`都会转换成一个Settings对象
	
	>**对于其他的gradle文件，除非定义了class,否则会转换成一个实现了Script接口的对象(与Groovy类似)**

##### 1.2.2.1 Gradle对象

	//在settings.gradle中，则输出"In settings,gradle id is"  
	println "In settings.gradle, gradle id is " +gradle.hashCode()  
	println "Home Dir:" + gradle.gradleHomeDir  
	println "User Home Dir:" + gradle.gradleUserHomeDir  
	println "Parent: " + gradle.parent  

- `settings.gradle`和module下的`build.gradle` ,得到的gradle实例对象是一样的(根据hashCode判断)

- HomeDir是gradle可执行程序的路径

- User Home Dir:gradle配置的目录,存储了一些配置文件,以及编译过程中的缓存文件，生成的类文件，编译过程中依赖的插件等

- **Gradle对象 默认是Settings和Project的成员变量.**

- Gradle对象是根据`init.gradle`生成的

##### 1.2.2.2 Project对象
- 每个`build.gradle`文件会转换成一个Project对象.

- 在Gradle术语中，Project对象对应的是`BuildScript`

- Project包含若干Task.由于Project对应具体的工程，所以需要为Project加载所需要的插件，比如为Java工程加载Java插件。其实，**一个Project包含多少Task往往是插件决定的**。

- `build.gradle`中所有未定义的方法/属性，都会委派给Project对象去使用
		println "name = $name"

-  局部变量 用def 声明，且只能在被定义的地方可见(Groovy特征)


- **通常Project需要执行的内容：**

	1. 加载插件

	通过Project的`apply(key:value)`函数来加载插件，`apply plugin:'com.android.library'`
	
		- 除了加载二进制文件,还可以加载gradle文件
		> **from**: 被添加的脚本. Accepts any path supported by Project.uri(java.lang.Object).
		>
		> **plugin**: Plugin的Id或者是插件的具体实现类 

		> **to**: The target delegate object or objects. The default is this plugin aware object. Use this to configure objects other than this object.
	
	2. 配置插件。例如设置哪里读取源文件。
	
	3. 设置属性

		- 如果是单个脚本，则不需要考虑属性的跨脚本使用。但是Gradle往往包含不止一个`build.gradle`文件！例如,`build.gradle`,`settings.gralde` 和自定义的`build.gradle`.**Gradle提供了一种名为`extra property`的方法**

		- **`extra property`是额外属性的意思**，在第一次定义该属性的时候需要通过`ext`前缀来标示它是一个额外的属性。定义好之后，后面的存取就不需要ext前缀了。**ext属性支持Project和Gradle对象即Project和Gradle对象都可以设置ext属性**

		- 属性值可以从`local.properties`中读取

				Properties p = new Properties()
				File pF = new File(rootDir.getAbsolutePath()+'/local.properties')
				properties.load(pF.newDataInputStream())

		- **可以直接获取`ext`前缀，表明操作的是外置属性**.定义属性或设置属性时需要ext前缀。读取时就不需要ext前缀了

				gradle.ext.api = p.getProperty('sdk.api')
				println gradle.api 

		除了`ext.xxx=value`这种定义方式之外，还可以使用`ext{}`这种书写方式。**ext{}不是ext函数传入Closure，但是ext{}中的{}的确是`Closure`**

				ext{
					    getVersionNameAdvanced = this.&getVersionNameAdvanced  
				}

		- **加载`utils.gradle`的Project对象**和**`utils.gradle`对象本身所代表的Script对象**的关系。

			- 当一个Project apply一个gradle文件时，这个gradle文件会转换成一个Script对象

			- Script中有一个delegate对象，这个delegate默认是被设置为 加载Script的Project对象(即调用apply的project)

			- 在apply中有一个to参数，可以将delegate指定为别的对象

			- **delegate作用**：当Script中操作一些不是Script自己定义的变量或函数时，gradle会到Script的delegate对象去找，看看有没有定义这些变量或函数

		- `utils.gradle`对应的project就是加载`utils.gradle`的project

		- utils中的ext 就是对应project的ext。

#### 1.2.2.4 Lifecycle
>There is a one-to-one relationship between a Project and a build.gradle file. During build initialisation, Gradle assembles a Project object for each project which is to participate in the build, as follows:

- Create a **Settings instance** for the build.

- Evaluate the settings.gradle script, if present, against the Settings object to configure it.

- Use the configured Settings object to create the hierarchy of Project instances.

- Finally, evaluate each Project by executing its build.gradle file, if present, against the project. The projects are evaluated in breadth-wise order, such that a project is evaluated before its child projects. This order can be overridden by calling Project.evaluationDependsOnChildren() or by adding an explicit evaluation dependency using Project.evaluationDependsOn(java.lang.String).



#### 1.2.3 Task介绍
>Task 是Gradle中的一种数据类型，代表了一些要执行或todo的工作。不同插件可以添加不同的Task。每一个Task都需要和一个Project关联

- 一个Task包含若干Action.所以Task提供了`doFirst`和`doLast`俩个函数 方便开发者使用，这俩个函数分别是用于最先执行的和最后执行的action。**Action就是一个闭包**

- Task创建的时候可以指定Type，通过`type:typeName`表达。作用就是告诉Gradle，该Task是从哪个基类Task 派生。 则新建的Task也具有基类Task的功能。例如：`task mTask(type:Copy)`，mTask也是一个Copy Task

-  **`task mTask{configure closure}`。花括号代表一个Closure，会在Gradle创建这个Task之后返回给用户之前，先执行这个Closure的内容,这个Closure可以被当做一个配置项的存在，去做一些配置，例如设置分组，添加描述等**

-  `task mTask<<{xxx}`,意思是把closure作为一个action添加到Task的action队列，并且最后才去执行它(`<<`符号是doLast的代表)

- doLast的快捷键`<<`,会在Gradle5.0中遗弃

- 通过group设置分组，通过description 设置描述

		task taskB{
			group = 'test'
			description = 'desc'
		}
	
		taskA.group = 'test'
		taskA.description = 'desc'


##### 1.2.3.1 定义Task
- Task是和Project关联的，所以要利用Project的task函数来创建一个Task  

- 在创建task时，通常可以传入一个`Closure`,**这个Closure是用来配置task的，会在task返回之前执行。**

		task myTask  <==myTask是新建Task的名字  
		
		task myTask { configure closure } //closure用来设置配置
		
		task myType << { task action } <==注意，<<符号是doLast的缩写 ，用来添加action
		
		task myTask(type: SomeType)  
		
		task myTask(type: SomeType) { configure closure }

		task('task1')<< { println 'task1 is created'}

		task(task2,type:Copy){
			from 'xxxx'
			into 'yyyy'
		}

		task task3<<{println 'task3 is created'}

- **Task另外一种创建方式**

		tasks.create(name:'task4'){
			group 'test'
			description 'i am task4'
			doLast{
				println 'task4 is created'
			}	
		}
		
		tasks.create(name:'task5',type:Copy){
			group 'test'
			from 'xxx'
			into 'yyy'
		}
	
		// 实际上是通过TaskContainer创建
		TaskContainer getTasks();
		

##### 1.2.3.2 Task依赖
- task 可以依赖于另外一个task 通过`dependsOn`

		task funcX()
		task funcY(dependsOn:funcX)		

- **`Lazy DependsOn` task依赖task时可以在task定义之前**

		task funcX(dependsOn:funcY)<<{
		}

		task funcY()<<{
		}

- Gradle可以动态创建 Task

		4.times{
			task "task$it"{
				doLast{
					println "i am task $it"
				}
			}
		}

- 创建任务之后,可以在运行时动态添加依赖关系

		task0.dependsOn(task1,task2,task3)

- 可以为task添加来自其他project的依赖
	
		project(':moduleA'){
			task task1(dependsOn:':moduleB:tasks2')<<{
				println 'moduleA task1 is run'
			}
		}

		project(':moduleB'){
			task task2<<{
				println 'moduleB task2 is run'
			}
		}

- 依赖可以使用一个闭包来返回Task

		task6.dependsOn {
			tasks.findAll{
				it.name.startWidth('task')
			}
		}


##### 1.2.3.3 设置默认Task
Gradle可以通过 `defaultTasks 'tasks1','tasks2'`来设置默认执行的task(当没有其他task明确被执行时),例如:`gradle -q`时，会去执行task `clean`
		
	defaultTasks 'clean'

	task clean<<{
		println 'default cleaning'
	}

##### 1.2.3.4 Task额外属性
通过`ext.xxxx`来替task设置额外属性

	task func{
		ext.nameProperty = 'ryan'
	}

	task func1<<{
		println "hello my name is $func.nameProperty"
	}

##### 1.2.3.5 Task的使用

- 可以通过`tasks.getByPath()`方法 来获取  使用任务名称 ，相对路径 或者绝对路径调用该方法

		project(':moduleA'){
			task taskA
		}

		println tasks.getByPath('tasksA').path
		println tasks.getByPath(':script:tasksA').path

- 可以将Task作为属性来使用

		println task1.name
		println project.task1.name

- 可以通过tasks collection来访问Task
		
		println tasks.tasks1.name
		println tasks['tasks1'].name

##### 1.2.3.6 配置Task

- 方式1

		Copy copy1 = task(task10,type:Copy)
		copy1.from '/'
		copy1.into 'task10'
		copy1.include('**.txt')

- 方式2
		task task11(type:Copy)

		task11{
			from '/'
			into 'task11'
			include '**.txt'
		}

- 方式3
		task (task12,type:Copy){
			from '/'
			into 'task12'
			include '**.gradle'
		}

##### 1.2.3.7 Task的Action
可以通过API来访问task，用来添加action

	task func<<{
		println 'normal'
	}

	func.doFirst{
		println 'before normal'
	}

	func.doLast{
		println 'after normal'
	}

	func{
		doLast{
			println 'after after normal'
		}
	}

##### 1.2.3.8 覆盖任务

可以通过`overwrite`覆盖任务，如果任务已经存在且不添加`overwrite`,会抛出一个异常，表示任务已经存在

	task taskA<<{
		println 'hello'
	}

	task taskA(overwrite:true)<<{
		println 'overwrite hello'
	}

##### 1.2.3.9 设置任务执行条件

	task taskA<<{
		println 'hello gradle'
	}

	tasksA.onlyIf{
		!project.hasProperty('xxxx')
	}

##### 1.2.3.10 中断Task

- 通过`throw new StopExcutionException()`抛出异常

		taskA.doFirst{
			throw new StopExcutionException()
		}

- Task用有一个`enabled` 的属性

		taskA.enabled = false


### 1.3 项目结构

 	MyApp
		├── build.gradle
		├── settings.gradle
		└── app
			├── build.gradle
			├── build
			├── libs
			└── src
				└── main
               	├── java
               	│   └── com.package.myapp
               	└── res
                   	├── drawable
                   	├── layout
                   	└── etc.

### 1.4 Gradle Wrapper

Gradle Wrapper 提供了一个batch文件，当使用脚本时，当前的gradle版本会被下载下来 并使用，避免了开发者去下载不同版本的gradle，解决兼容性！

	 myapp/
   	├── gradlew
   	├── gradlew.bat
   	└── gradle/wrapper/
       	├── gradle-wrapper.jar
       	└── gradle-wrapper.properties

- bat文件针对window系统，shell脚本针对mac系统，一个jar文件，一个配置文件。配置文件如下：  

		#Sat May 30 17:41:49 CEST 2015
   	distributionBase=GRADLE_USER_HOME
   	distributionPath=wrapper/dists
   	zipStoreBase=GRADLE_USER_HOME
   	zipStorePath=wrapper/dists
   	distributionUrl=https\://services.gradle.org/distributions/
   	gradle-2.4-all.zip
	
	- **可以改变distributionUrl 来改变gradle版本**

### 1.5 基本构建命令
- gradle projects 

	**查看工程信息**,直接查看setting.gradle也可以得到结果

- gradle tasks
	**获取所有有分组的可运行task**

	- 查看指定Project的任务，`gradlew project-path:tasks`,project-path 是目录名，这是在根目录的情况。如果已经在某个Project的目录下了 ，则不需要指定

			gradlew hmt_sdk:tasks
		
			cd XXXModule
			gradlew tasks

	- `gradlew tasks`会列出每个任务的描述

	- 添加--all参数  来查看task的依赖关系

- gradlew task-name
	执行指定名称的任务

	- task 与task之间往往是有依赖关系的


- 获取所有有分组的可运行tasks，可以
		gradlew tasks

- 创建一个指定buildType的apk
		gradlew assembleXXX

- 移除所有的编译输出文件
		gradlew clean

- 同时执行assemble和check命令
		gradlew build 

- 执行lint监测编译
		gradlew check

这些都是基本的命令，在实际项目中会根据不同的配置，会对这些task 设置不同的依赖。比如 默认的 assmeble 会依赖 assembleDebug 和assembleRelease，如果直接执行assmeble，最后会编译debug，和release 的所有版本出来。如果我们只需要编译debug 版本，我们可以运行assembleDebug。

很多命令除了会输出结果到命令行，还会在`build`文件夹下下生成运行报告，例如`check`命令会生成lint-result.html在`build/outputs`


### 1.6 构建脚本的构成

	MyApp
   	├── build.gradle
   	├── settings.gradle
   	└── app
       	└── build.gradle

- **setting.gradle **

	这个 setting 文件定义了哪些module 应该被加入到编译过程，对于单个module 的项目可以不用需要这个文件，但是对于 multimodule 的项目我们就需要这个文件，否则gradle 不知道要加载哪些项目。这个文件的代码在初始化阶段就会被执行。

- **根目录的build.gradle**

	顶层的build.gradle文件的配置最终会被应用到所有项目中。它典型的配置如下：
		buildscript {
    		repositories {
        		jcenter()
    		}

    		dependencies {
        		classpath 'com.android.tools.build:gradle:1.2.3'
    		}
		}

		allprojects{
    		repositories{
        		jcenter()
    		}
		}

	- **buildscript**:定义了Adnroid编译工具的类路径.repositories中，jCenter是一个仓库

	- **allprojects**:定义的属性会被应用到所有的module中，但是为了保证每个项目的独立性，我们一般不会在这里操作太多共有的东西


- **每个项目单独的build.gradle**:仅针对每个module的配置,这里的配置优先级最高

#### 1.6.1 module中的build.gradle介绍

		apply plugin: 'com.android.application'
	
		android {
    		compileSdkVersion 25
    		buildToolsVersion "25.0.3"

    		defaultConfig {
        		applicationId "com.hmt.analytics.customizeplugin"
        		minSdkVersion 16
        		targetSdkVersion 25
        		versionCode 1
        		versionName "1.0"
        		testInstrumentationRunner 	"android.support.test.runner.AndroidJUnitRunner"
    			}

    		buildTypes {
        		release {
            		minifyEnabled false
            		proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        				}
    			}
		}

		dependencies {
    		compile fileTree(dir: 'libs', include: ['*.jar'])
    		compile 'com.android.support:appcompat-v7:25.3.1'
		}



- **apply plugin:**添加了Android程序的gradle插件,plugin提供了Android编译，测试，打包等等task


- **android：** 编译文件中最大的代码块，关于android的所有特殊配置都在这里，这里就是前面plugin所提供的

- **defaultConfig**：程序的默认配置,如何和AndroidMainfest.xml定义了重复的属性，会以这里为主


- **applicationId**:在我们曾经定义的AndroidManifest.xml中，那里定义的包名有两个用途：一个是作为程序的唯一识别ID,防止在同一手机装两个一样的程序；另一个就是作为我们R资源类的包名。在以前我们修改这个ID会导致所有用引用R资源类的地方都要修改。但是现在我们如果修改applicationId只会修改当前程序的ID,而不会去修改源码中资源文件的引用。

- **buildTypes:**定义了编译类型,针对每个类型可以有不同的编译配置,不同的编译配置对应不同的编译命令。默认debug,release


- **dependencies:**属于gradle的依赖配置。定义当前module需要依赖的三方库
	- 引用库时,每个库名称包含三个元素:`组名`：`库名称`：`版本号`
	- 可以通过添加通配符来保证依赖库处于最新状态，但是建议不要这么做，因为这样每次编译都要去请求网络判断是否有最新版本
	- 通过`files()`方法可以添加文件依赖，如果有很多文件，可以通过`fileTree()`方法
- **native libraries**
	配置本地.so库。在配置文件中做如下配置，然后在对应位置建立对应文件夹，并加入对应平台的.so文件即可
		android{
			sourceSets.main{
				jniLibs.srcDir  'src/main/jniLibs'
			}
		}

- **BuildConfig**
	这个类是根据gradle配置文件生成的，其中的参数例如BuildConfig.DEBUG 可以用来判断当前版本是否是debug版本。

	我们可以在defaultConfig中 或buildTypes中具体的类型中 定义一些key-value对，这些key-value对在不同的编译类型的apk下的值不同，例如我们可以为debug,release设置不同的请求地址
		buildTypes{
			debug{
				buildConfigField "String","API_URL","www.google.com"
			}

			release{
				buildConfigField "String","API_URL","www.irs01.com"
			}
		}

	此外还可以为不同编译类型设置不同的资源文件
		buildTypes{
			debug{
				resValue "String","app_name","example_demo"
			}

			release{
				resValue "String","app_name","demo"
			}
		}
		

- **repositories**
	Repositories 就是代码仓库,平时的添加的一些 dependency 就是从这里下载的，Gradle 支持三种类型的仓库：Maven,Ivy和一些静态文件或者文件夹。在编译的执行阶段，gradle 将会从仓库中取出对应需要的依赖文件，当然，gradle 本地也会有自己的缓存，不会每次都去取这些依赖。

	gradle支持多种Maven仓库，一般为公共的Jcenter，可以通过手动添加一些私人库并添加账号密码
		repositories{
			maven{
				url "http://repo.xxx.xx/maven"
				creadentials{
					username 'user'
					password 'password'
				}
			}
		}

	也可以使用相对路径配置本地仓库，可以通过配置项目中存在的静态文件夹作为本地仓库
		repositories{
			flatDir{
				dirs 'aars'
			}
		}


- **library projects**
	需要写一个library项目给其他项目引用，那么apply plugin 就需要改成 'com.android.library',另外还需要在setting.gradle 中include。 默认生成的话as会做好这些

	如果不方便直接引用module ，可以将module打包成aar形式，然后通过文件的形式引用。这种情况需要在项目下新建aars文件夹，并在根目录下的build.gradle配置**本地仓库**，然后在dependencies中添加`compile name:'libraryname',ext:'aar'`

- **build variants-build type**
	在编译的时候动态根据当前的编译类型输出不同样式的apk文件等情况时就需要用到了buiildtypes

		buildTypes{
			staging.initWith buildTypes.debug
			staging{
				applicationIdSuffix '.staging'
				versionNameSuffix '-staging'
				debuggable = false
			}
		}
- **Source sets**
	每当新建一个build type时，gradle默认会创建一个新的source set。可以建立与`main`同级的文件夹，这样在编译时 会根据不同的编译类型 选择某些对应文件夹下的源码。不止文件夹可以替换，资源也可以替换

	另外dependencies中也可以通过 `buildType+compile` 来指定 编译类型去添加指定三方框架


- **product flavors**
	如果我们需要针对同一份代码编译不同的程序(包名不同)，就需要`product flavors`
	- 注意product flavors和build type是不一样的，而且他们的属性也不一样。所有的product flavor和defaultConfig共享属性

	像Build type 一样，product flavor 也可以有自己的source set文件夹。除此之外，product flavor 和 build type 可以结合，他们的文件夹里面的文件优先级甚至高于 单独的built type 和product flavor 文件夹的优先级。如果你想对于 blue类型的release 版本有不同的图标，我们可以建立一个文件夹叫blueRelease，注意，这个顺序不能错，一定是 flavor+buildType 的形式。

	更复杂的情况下，我们可能需要多个product 的维度进行组合，比如我想要 color 和 price 两个维度去构建程序。这时候我们就需要使用flavorDimensions：

			android{
				flavorDimensions 'color','price'

				productFlavors{
					red{
						flavorDimension 'color'
					}
					blue{
						flavorDimension 'color'
					}
					free{
						flavorDimension 'price'
					}
					paid{
						flavorDimension 'price'
					}
				}
			}
	这样gradle会自动进行组合，得出类似blue+free+debug blue+paid+release red+free+debug red+paid+release

	BuildType中定义资源优先级最大，Library中定义的资源优先级最低

- **signing configurations**
	首先我们需要在android{}中配置
		android{
			signingConfigs{
				storeFile file("release.jks")
				storePassword "password"
				keyAlias 'rellease-jks'
				keyPassword "123456"
			}
		}

	配置之后需要在build type中使用
		buildTypes{
			release{
				signingConfig signingConfigs.release
			}
		}
	

#### 1.6.2 Speeding up multimodule build
通过以下方式加快gradle的编译

- **开启并行编译：** 在项目根目录下的`gradle.properties`中设置
		org.gradle.parallel = true

- **开启编译守护进程:** 该进程在第一次启动后会一直存在，接下来每次编译都可以重用该进程，也是在项目根目录下的`gradle.properties`中设置

- **加大可用编译内存： ** 同样是在项目根目录下的`gradle.properties`中设置
		org.gradle.jvmargs=-Xms256m -Xmx1024m



#### 1.6.3 Reducing apk file
在编译的时候，有许多的资源并没有用到，可以通过`shrinkResources`来优化资源文件，除去不必要的资源。

	android{
		buildTypes{
			release{
				minifyEnabled = true //只有当俩者都为true 才会真正的删除无效代码和无银用资源
				shrinkResources = true//
			}
		}
	}

在某些情况下，一些资源是通过动态加载的方式载入的，这时候需要像Progard一样对资源进程**keep**操作。操作方式就是：在`res/raw/`下创建一个`keep.xml`文件，使用如下方式keep资源

	<?xml version="1.0" encoding="utf-8"?>  
	<resources xmlns:tools="http://schemas.android.com/tools"  tools:keep="@layout/activity_four,@drawable/no_reference_but_keep"/>  

	

对于一些尺寸文件，我们可以这么做去防止被去除：

	android{
		defaultConfig{
			resConfigs "hdpi","xhdpi","xxhdpi"
		}
	}

#### 1.6.4 命令参数(指令)
- 执行`task`的时候可以通过添加`--profile`参数生成一份执行报告在`reports/profile`中

- `-q`可以抑制gradle日志消息

- 执行Task时， 添加`--continue` 可以在任务失败之后 继续执行

- 通过`-P`设置属性，注意大小写
		gradle -q taskA -P xxxx

## 2.文件
### 2.1 获取File对象
- 使用相对路径

		File file1 = file('hello.txt')
		println "file1 = ${file1.getText()}"

- 使用绝对路径

		File file2 = file(file1.absolutePath)
		println "file2 = ${file2.getText()}"		

- 使用具有相对路径的File对象

		File file3 = file(new File('hello.txt'))
		println "file3 = ${file3.getText()}"

### 2.2 获取FileCollection

- 通过`files()`获取，可以将 集合，迭代 映射 和数组传给此方法。这些将会被展开并转换成实例
		FileCollection collection1 = files('hello.txt',new File('other.gradle'))
		FileCollection collection2 = files('hello.txt')
		collection1.each{
			println "file name =${it.name}"
		}

- 将FileCollection转换成　各种类型
		Set set1 = collection1.files
		Set set2 = collection1 as Set
		List list1 = collection1 as List
		String path1 = collection1.asPath

- 以下俩个方法当FileCollection只存在一个 File时可以使用
		File file4 = collection2.singleFile
		File file5 = collection2 as File



- 添加和减去 FileCollection
		def union1 = collection1 + files('build.gradle')
		union1.each{
			println "union1 file name = ${it.name}"
		}
		def different1 = collection1 - files('hello.txt')
		different1.each{
			println "different1 file name = ${it.name}"
		}

- 通过`listFiles()`方法可以将`dir` 转换成`FileCollection`


### 2.3 文件树 FileTree

- 文件树是按照层次排列文件的集合,由FileTree 表示，其扩展了FileCollection ,`Project.fileTree(Map)`

		FileTree tree1 = fileTree(dir:'src')

- 添加 包含和不包含的规则

	- 添加方式1

			tree1.include '*.txt'
			tree1.exclude '*.gradle'
			tree1.each{
				println "FileTree $it.name"
			}

	- 添加方式2
	
			tree1 = fileTree('src').include('*.gradle')

	- 添加方式3

			tree1 = fileTree('src'){
				include '*.txt'
			}

	- 添加方式4 通过map创建

			tree1 = fileTree(dir:'src',include:'*.txt')
			tree1 = fileTree(dir:'src',include:['*.txt','*.gradle'])
			tree1 = fileTree(dir:'src',include:'*',exclude:'')

- 筛选FileTree

		FileTree filtered1 = tree1.matching{
			include '*.txt'
		}

- 遍历FileTree
		filtered1.each{
			println "file name = $it.name"
		}


- Add trees together

		FileTree sum = filtered1 + fileTree(dir: 'src',include:'*.txt')

- Visit the elements of the tree
		sum.visit {element ->
    		println "$element.relativePath => $element.file"
		}



- 可以将 zip or tar 作为文件树 ，以下俩个方法会返回FileTree
		Project.zipTree(Object)
		Project.tarTree(Object)

		FileTree someTar1 = tarTree(resources.gzip('xxx.ext'))
		someTar1.each{
			println it.name
		}

- 指定一组输入文件
		task compile1(type:JavaCompile){
			source = file('src')
		}
		source= file('src') 接受一个file对象
		source = 'src' 接收路径
		source = ['src','dest']  可以使用集合来指定多个源目录
		source = fileTree(dir:'src').matching{ include '*.gradle'} 可以接收一个fileTree
		source = { file('src').listFiles().findAll{ it.name.endsWith('.zip')}.collect{zipTree(it)}   }
 


### 2.4 Copying file

- 复制文件时，可以过滤内容 ，**需要提供 from   into  ** 

		task copy2(type:Copy){
			from 'src'
			into 'dest'
		}

- from 可以接收一个files()方法作为参数
	- 当参数被解析时，如果是目录，则该目录下的所有内容将被递归复制到目标目录下（注意：不是目录本身）
	- 当参数被解析后，不存在，则会忽略
	- 当参数是task时，即取 task 的输出结果，并且该task会被添加为Copy task的依赖

			task copy3(type:Copy){
 			//目录
 			from 'src'
 			//单个文件
 			from 'src/b.txt'
 			//task的输出
 			// from copyTask
 			//明确指定任务的输出
 			// from copyTaskWithPatterns.outputs
 			//指定zip file
 			// from zipTree('src/xx.zip')
 			// into { getDestDir()}
 			}




- 在copy时，对输入文件进行筛选

		task copy4(type:Copy){
			from 'src'
			into 'copy4'
			include '*.txt'
			//传入闭包的话 会给闭包一个 FileTree 参数
			exclude {
				it.file.name.startsWith 'a'
			}
		}



- 复制文件除了使用 定义一个task 指定Copy 类型外，  Project 也提供了copy 方法

		task copy5<<{
 			copy{
 				from 'src'
 				into 'copy5'
 			}
 		}

		task copy6{
			inputs.file copy2
			outputs.dir 'copy6'
			doLast{
				copy{
					from copy2
					into 'copy6'
				}
			}
		}

- 重命名 file

		task rename1(type:Copy){
			from 'hello.txt'
			into '/'
			rename{
				it.replace('hello','hi')
			}
		}

- 嵌套输出 

		task copy7(type:Copy){
			into 'copy7'
			exclude '*.txt'
			from('src'){
				include '*'
			}

			into('copy7') {
				exclude '*.java'
				from 'src'
			}
		}

### 2.5 Sync
- Sync 是扩展自Copy
- 与copy的区别就是， Sync 会将文件先全部复制到 目标目录下，然后再将不需要的删除

		task copy8(type:Sync){
			from 'src'
			into 'copy8'
		}



### 2.6 zip
- 创建zip，默认 应该是生成在buiild下，命名规则应该是 `projectName-version.type` 
	- version 可以在task中指定。。 貌似在 全局设置version 没用！
	- baseName 可以替换掉 projectName 
	- baseName - appendix - version - classifier .type

			task zip1(type:Zip){
				version = 1.5
				baseName = 'xixixixi'
				appendix = 'appendix'
				classifier = 'classifier'
				from 'src'
			}



### 2.7 Settings file
- 除了构建脚本 build.gradle之外，gradle 还提供了一个 settings.gradle 

- settings.gradle 在初始化阶段执行。另外 多项目构建 必须有settings.gradle 

- build.gradle中属性访问 和 方法调用 被project 代理. 同理 属性访问 和 方法调用 在settings.gradle 中 被 settings 代理


## 3.实例
### 3.1 keystore 保护
如果我们将store的密码明文的写在signingConfigs里面，对安全性不好，所以需要构建一个动态加载任务，在编译release源码的时候从本地文件(git忽略名单中的文件)获取keystore信息

	task getReleasePsw << {
   	 def psw = ''
   	 println 'getReleasePsw is running'
    	if (rootProject.file('local.properties').exists()) {
     	   java.util.Properties properties = new Properties();
     	   properties.load(rootProject.file('local.properties').newDataInputStream())
    	    psw = properties.getProperty('release.psw')
  	  } else if (psw.trim().isEmpty()) {
    	    //TODO 在获取不到配置文件时 从控制台输入
   	     psw = java.lang.System.in.each {
   	     }
  	  }	
	}

光创建这个task还不行，需要为task添加依赖 这样执行打包task时 自动执行getReleasePsw任务
	
	tasks.whenTaskAdded{
		if(it.name.equals 'packageRelease'){
			it.dependsOn 'getReleasePsw'
		}
	}


### 3.2 hook Android编译插件 重命名apk

	android.applicationVariants.all{variant->
		variant.outputs.each{output->
			def file = output.outputFile

			output.outputFile = new File(file.parent,
			file.name.replace(".apk","-${variant.versionName}.apk"))
		}
	}

生成类似 `app-debug-1.0.apk`


### 3.3 设置默认值

- 通过以下设置 可以在 task被添加到project时立刻接收到通知
- 这可以用来设置一些默认值或行为（task在Build file中可用之前）


	tasks.whenTaskAdded{
		it.ext.srcDir = 'src'
	}

	task task22<<{
		println "task22 srcDir =$srcDir "
	}

### 3.4 配置结束回调
- task 执行图绘制完毕，应该是配置结束

		gradle.taskGraph.whenReady{
			println "taskGraph.whenReady =  $it"
		}

### 3.5 buildTypesScriptBlock

- buildTypes和上面的signingConfigs，当我们在build.gradle中通过{}配置它的时候， 其背后的所代表的对象是NamedDomainObjectContainer< BuildType>和NamedDomainObjectContainer < SigningConfig> 

- 注意，NamedDomainObjectContainer< BuildType/SigningConfig>是一种容器，容器的元素是BuildType或者SigningConfig。

- 我们在debug{}要填充BuildType或者SigningConfig所包的元素，比如storePassword就是SigningConfig类的成员。而proguardFile等是BuildType的成员。 


- 为什么要使用NamedDomainObjectContainer这种数据结构呢？因为往这种容器里添加元素可以采用这样的方法： 
	比如signingConfig为例 
    	signingConfig{//这是一个NamedDomainObjectContainer<SigningConfig> 
       		test1{//新建一个名为test1的SigningConfig元素，然后添加到容器里 
        		 //在这个花括号中设置SigningConfig的成员变量的值 
       		} 
      		test2{//新建一个名为test2的SigningConfig元素，然后添加到容器里 
         		//在这个花括号中设置SigningConfig的成员变量的值 
      		} 
    	} 

- 在buildTypes中，Android默认为这几个NamedDomainObjectContainer添加了debug和release对应的对象。如果我们再添加别的名字的东西，那么gradleassemble的时候也会编译这个名字的apk出来。比如，我添加一个名为test的buildTypes，那么gradle assemble 就会编译一个xxx-test-yy.apk。在此，test就好像debug、release一样。 

### 3.6 NamedDomainObjectContainer使用说明


## 4 引用说明
[深入理解Android之Gradle](http://blog.csdn.net/innost/article/details/48228651)

[Gradle之完整指南](http://www.jianshu.com/p/9df3c3b6067a)

[Android-Script Block-DSL](https://developer.android.com/tools/building/plugin-for-gradle.html)

[Gradle-旧版本文档](https://sites.google.com/a/android.com/tools/tech-docs/new-build-system/user-guide#TOC-Advanced-Build-Customization)

[Android-Dsl-APi](https://google.github.io/android-gradle-dsl/current/index.html)

[Gradle深入与实战（六）Gradle的背后是什么？](http://benweizhu.github.io/blog/2015/03/31/deep-into-gradle-in-action-6/)