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

   <delete dir="classes" />

   删除classes文件夹

###拷贝文件：

    <copy todir="$\{backup.dir}"> 

        <fileset dir="$\{classes.dir}"/> 

    </copy>

把fileset文件夹下面的所有文件拷贝到 backup.dir


###执行一个类：

    <java dir="$\{build}" classname="bean.ant.TestAnt" fork="true" />

* dir　: 工作文件夹
* classname: 类名。
* fork : 要设置为true。

因为你编译放class的文件夹正在使用，所以要新打开一个虚拟机。


###javadoc：

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