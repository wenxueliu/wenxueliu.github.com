ant的编译文件默认为build.xml，一般无需改变。

###project 

build.xml的根节点为 < project >，一般格式如下：

   <project name="AntStudy" default="init" basedir=".">

* name    : 工程名称；
* default : 默认的 target，就是任务；
* basedir : 基路径。一般为"."


###property 

可以定义变量，一般格式如下：

   <property name="test" value="shit" />

* 引用方式 "$\{test}"

###target

    <target name="compile" depends="init"><!--other command--></target>

* name : target的名称，可以在编译的时候指定是完成哪个target，否则采用project那里定义的default。
* depends : 定义了依赖关系，值为其他target的name。多个依赖关系用","隔开，顺序执行完定义的依赖关系，才会执行这个target。
* target 在 build.xml 中定义的顺序无所谓，但是 depends 中的顺序必须正确。
* if 用于验证指定的属性是否存在，若不存在，所在 target 将不会被执行。
* unless 与 if 功能正好相反，它也用于验证指定的属性是否存在，若不存在，所在 target 将会被执行。
* description 关于 target 功能的简短描述和说明。

###编译源代码

    <javac srcdir="src" destdir="classes">

         <classpath> 

             <fileset dir="lib"> 

                 <include name="**/*.jar"/> 

             </fileset>

         </classpath> 

     </javac>

这个标签自动寻找src中以.java为扩展名的文件，并且调用javac命令。
这个任务有个特点，它仅仅编译那些需要编译的源文件。如果没有更新，就不需要编译，速度就加快。
编译文件和ant使用的同一个 jvm，大大减少资源浪费。还可以指定　classpath。classpath　中指定文件夹，然后指定包含的文件的规则。

###创建目录

   <mkdir dir="classes" />

创建dir的文件夹。

###删除目录

   对文件或目录进行删除

eg1. 删除某个文件：

	<delete file="/home/photos/philander.jpg"/>

eg2. 删除某个目录：

	<delete dir="/home/photos"/>

eg3. 删除所有的备份目录或空目录：

	<delete includeEmptyDirs="true">
		   <fileset dir="." includes="**/*.bak"/>
	</delete>
	   <delete dir="classes" />

   删除classes文件夹

###拷贝文件：

   	主要用来对文件和目录的复制功能

eg1. 复制单个文件：

	<copy file="original.txt" tofile="copied.txt"/>

eg2. 对文件目录进行复制：

	<copy todir="../dest_dir">
		  <fileset dir="src_dir"/>
	</copy>

eg3. 将文件复制到另外的目录：

	<copy file="source.txt" todir="../home/philander"/>
		
eg4. 利用 property

		<copy todir="$\{backup.dir}"> 

		    <fileset dir="$\{classes.dir}"/> 

		</copy>

把fileset文件夹下面的所有文件拷贝到 backup.dir

###移动文件或目录

eg1. 移动单个文件：

<move file="sourcefile" tofile=”destfile”/>

eg2. 移动单个文件到另一个目录：

<move file="sourcefile" todir=”movedir”/>

eg3. 移动某个目录到另一个目录：

<move todir="newdir"> <fileset dir="olddir"/></move>

###echo 命令

    该任务的作用是根据日志或监控器的级别输出信息。它包括 message 、 file 、 append 和 level 四个属性，举例如下

<echo message="Hello,ANT" file="/home/philander/logs/ant.log" append="true">

###java

    <java dir="$\{build}" classname="bean.ant.TestAnt" fork="true" />

* dir　: 工作文件夹
* classname: 类名。
* fork : 要设置为true。

 该标签用于执行编译的.class文件。

* classname 表示将执行的类名。
* jar表示包含该类的JAR文件名。
* classpath所表示用到的类路径。
* fork表示在一个新的虚拟机中运行该类。
* failonerror表示当出现错误时自动停止。
* output 表示输出文件。
* append表示追加或者覆盖默认文件。

因为你编译放class的文件夹正在使用，所以要新打开一个虚拟机。

###javac

    用户编译一个或一组java文件。

* srcdir表示源程序的目录。
* destdir表示class文件的输出目录。
* include表示被编译的文件的模式。
* excludes表示被排除的文件的模式。
* classpath表示所使用的类路径。
* debug表示包含的调试信息。
* optimize表示是否使用优化。
* verbose 表示提供详细的输出信息。
* fileonerror表示当碰到错误就自动停止。


###jar

	该标签用来生成一个jar文件。

* destfile表示JAR文件名。
* basedir表示被归档的文件名。
* includes表示别归档的文件模式。
* exchudes表示被排除的文件模式。

###javadoc：

* packagenames="com.lcore.*"
* sourcepath="${basedir}/src"
* destdir="api"
* version="true"
* use="true"
* windowtitle="Docs API"
* encoding="UTF-8"
* docencoding="GBK">  

###path

可以定义path对象，在其他地方可以直接复用。

    <path id="1"> 

       <pathelement location="."/> 

       <pathelement location="./lib/junit.jar"/> 

    </path>

    <path id="2"> 

       <fileset dir="lib"> 

       <include name="**/*.jar"/> 

       </fileset> 

    </path>

    <javac srcdir="./src" destdir="./classes"> 

       <classpath refid="1"/> 

    </javac>

    <javac srcdir="./src" destdir="./classes"> 

          <classpath refid="1"> 

              <pathelement location="."/> 

              <pathelement location="./lib/junit.jar"/> 

          </classpath> 

    </javac>
