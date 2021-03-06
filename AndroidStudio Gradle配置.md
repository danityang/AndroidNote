每个项目的编译至少有一个 Project,一个 build.gradle就代表一个project,每个project里面包含了多个task,
task 里面又包含很多action，action是一个代码块，里面包含了需要被执行的代码。

在编译过程中， Gradle 会根据 build 相关文件，聚合所有的project和task，执行task 中的 action。
因为 build.gradle文件中的task非常多，先执行哪个后执行那个需要一种逻辑来保证。这种逻辑就是依赖逻辑，
几乎所有的Task 都需要依赖其他 task 来执行，没有被依赖的task 会首先被执行。
所以到最后所有的 Task 会构成一个 有向无环图（DAG Directed Acyclic Graph）的数据结构。



一个项目有一个setting.gradle、包括一个顶层的 build.gradle文件、每个Module 都有自己的一个build.gradle文件。

**setting.gradle**: 这个 setting 文件定义了哪些module 应该被加入到编译过程，对于单个module 的项目可以不用需要这个文件，但是对于 multimodule 的项目我们就需要这个文件，否则gradle 不知道要加载哪些项目。这个文件的代码在初始化阶段就会被执行。
顶层的

**build.gradle**: 顶层的build.gradle文件的配置最终会被应用到所有项目中

**buildscript**： 定义了 Android 编译工具的类路径。repositories中,jCenter是一个著名的 Maven 仓库。

**allprojects**: 中定义的属性会被应用到所有 module 中，但是为了保证每个项目的独立性，我们一般不会在这里面操作太多共有的东西。
 repositories： 我们平时的添加的一些 dependency 就是从这里下载的，Gradle 支持三种类型的仓库：Maven,Ivy和一些静态文件或者文件夹。在编译的执行阶段，gradle 将会从仓库中取出对应需要的依赖文件，当然，gradle 本地也会有自己的缓存，不会每次都去取这些依赖。
 gradle 支持多种 Maven 仓库，一般我们就是用共有的jCenter就可以了。
 
 有一些项目，可能是一些公司私有的仓库中的，这时候我们需要手动加入仓库连接, 如果仓库有密码，也可以同时传入用户名和密码
  repositories {
        maven { url "https://xxxxx" }
        credentials{
            username:'user'
            password:'password'
        }
    }
    
我们也可以使用相对路径配置本地仓库，我们可以通过配置项目中存在的静态文件夹作为本地仓库：
  repositories {
        flatDir {
            dirs 'filedirs'
        }
    }

每个项目单独的 build.gradle：针对每个module 的配置，如果这里的定义的选项和顶层build.gradle定义的相同，后者会被覆盖。


**app(modlue)里的gradle配置文件**：
apply plugin:第一行代码应用了Android 程序的gradle插件，作为Android 的应用程序，
这一步是必须的，因为plugin中提供了Android 编译、测试、打包等等的所有task

  如果我们要写一个library项目让其他的项目引用，我们的bubild.gradle的plugin 就不能是andrid plugin了，需要引用如下plugin
  apply plugin: 'com.android.library'
  引用的时候在settings.gradle文件中include即可。

  如果我们不方便直接引用项目，需要通过文件的形式引用，我们也可以将项目打包成aar文件，注意，
  这种情况下，我们在项目下面新建arrs文件夹，并在build.gradle 文件中配置 仓库：
   repositories {
          flatDir {
              dirs 'filedirs'
         }
     }
  当需要引用里面的某个项目时，通过如下方式引用：  
  dependencies {
        compile(name:'libname', ext:'aar') 
  }


**android**: 这是编译文件中最大的代码块，关于android 的所有特殊配置都在这里，这就是又我们前面的声明的 plugin 提供的

**defaultConfig** 就是程序的默认配置，注意，如果在AndroidMainfest.xml里面定义了与这里相同的属性，会以这里的为主。

applicationId 在我们曾经定义的AndroidManifest.xml中，那里定义的包名有两个用途：一个是作为程序的唯一识别ID,防止在同一手机装两个一样的程序；
另一个就是作为我们R资源类的包名。在以前我们修改这个ID会导致所有用引用R资源类的地方都要修改。
但是现在我们如果修改applicationId只会修改当前程序的ID,而不会去修改源码中资源文件的引用

**buildTypes**: 定义了编译类型，针对每个类型我们可以有不同的编译配置，不同的编译配置对应的有不同的编译命令。默认的有debug、release 的类型。
**dependencies**: 是属于gradle 的依赖配置。它定义了当前项目需要依赖的其他库。
  我们在引用库的时候，每个库名称包含三个元素：组名:库名称:版本号, 如果我们要保证我们依赖的库始终处于最新状态，我们可以通过添加通配符的方式，比如：
  dependencies {
        compile 'org.eclipse.paho:org.eclipse.paho.client.mqttv3:1.2.+'
  }
 
 通过files()方法可以添加文件依赖，如果有很多jar文件，我们也可以通过fileTree()方法添加一个文件夹，除此之外，我们还可以通过通配符的方式添加，如下：
 dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
  }
  
 **Native libraries**
 配置本地 .so库。在配置文件中做如下配置，然后在对应位置建立文件夹，加入对应平台的.so文件。
  
  
  
  
  
  
###Gradle Wrapper
gradlw wrapper 包含一些脚本文件和针对不同系统下面的运行文件。wrapper 有版本区分，但是并不需要你手动去下载，当你运行脚本的时候，如果本地没有会自动下载对应版本文件。
在不同操作系统下面执行的脚本不同，在 Mac 系统下执行./gradlew ...，在windows 下执行gradle.bat进行编译。
wrapper 就是我们使用命令行编译的开始。