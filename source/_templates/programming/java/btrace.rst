

=======================================
Btrace
=======================================
BTrace是Java的安全可靠的动态跟踪工具。 他的工作原理是通过 instrument + asm 来对正在运行的java程序中的class类进行动态增强。

虚拟机其实提供了一个hook，那就是Instrumentation，可以将独立于应用程序的代理程序agent程序随着着应用程序一起启动或者attach
挂载到正在运行中的应用程序。而在代理程序中可以对class进行修改或者重新定义。

其本质是利用JVM的hotspot特性，改变jvm加载的类文件，通过类似AOP的方式进行拦截，也可以想象成解释型语言的机制，动态编译。

功能
---------------------------------------
* 接口性能变慢，分析每个方法的耗时情况；
* 当在Map中插入大量数据，分析其扩容情况；
* 分析哪个方法调用了某个函数，调用栈如何；
* 执行某个方法抛出异常时，分析运行时参数；

API
---------------------------------------
.. code::

    Method Annotations
    @com.sun.btrace.annotations.OnMethod //探测点
    @com.sun.btrace.annotations.OnTimer //定时器
    @com.sun.btrace.annotations.OnError //发生异常时回调
    @com.sun.btrace.annotations.OnExit //代码调用exit
    @com.sun.btrace.annotations.OnEvent //用户端事件
    @com.sun.btrace.annotations.OnLowMemory //内存低于阈值
    @com.sun.btrace.annotations.OnProbe //支持xml格式声明探测点
    Argument Annotations
    @com.sun.btrace.annotations.Self
    @com.sun.btrace.annotations.Return
    @com.sun.btrace.annotations.CalledInstance
    @com.sun.btrace.annotations.CalledMethod
    Field Annotations
    @com.sun.btrace.annotations.Export
    @com.sun.btrace.annotations.Property
    @com.sun.btrace.annotations.TLS
    Class Annotations
    @com.sun.btrace.annotations.DTrace
    @com.sun.btrace.annotations.DTraceRef
    @com.sun.btrace.annotations.BTrace

* clazz: 全路径类名，支持正则表达式，格式为/正则表达式/ +类名 匹配子类 @前缀 匹配anotation声明的类
* method: 方法名，支持正则表达式，格式为/正则表达式/， anotation使用* @
* location: 用* @Location来表明在什么时候时候去执行脚本
* Kind.ENTRY 进入方法时
* Kind.RETURN 方法返回时，只有把拦截位置定义为Kind.RETURN，才能获取方法的返回结果@Return和执行时间@Duration
* Kind.THROW 抛出异常时
* Kind.ARRAY_SET 设置数组元素时
* Kind.ARRAY_GET 获取数组元素时
* Kind.NEWARRAY 创建新数组时
* Kind.NEW 创建新对象时
* Kind.CALL 分析方法中调用其它方法的执行情况，比如在execute方法中，想获取add方法的执行耗时，必须把where设置成Where.AFTER
* Kind.CATCH 捕获异常时
* Kind.FIELD_SET 获取对象属性时
* Kind.FIELD_SET 设置对象属性时
* Kind.ERROR 方法由于发生未被捕获的异常结束时
* Kind.SYNC_ENTRY 进入同步块时
* Kind.SYNC_EXIT 离开同步块时
* Kind.LINE 通过设置line，可以监控代码是否执行到指定的位置
* type 方法类型， 不含方法名、参数民、异常声明。

class example:

.. code::

    使用全限定名：clazz="com.metty.rpc.common.BtraceCase", method="add"
    使用正则表达式：clazz="/javax.swing../", method="/./"
    使用接口：clazz="+com.ctrip.demo.Filter", method="doFilter"
    使用注解：clazz="@javax.jws.WebService", method=""@javax.jws.WebMethod"
    如果需要分析构造方法，需要指定method=""

Kind.LINE example, who called CaseObject line 5:

.. code:: java
        
    import static com.sun.btrace.BTraceUtils.*;
    import com.sun.btrace.annotations.*;
     
    @BTrace public class TraceMethodLine{
       @OnMethod(
          clazz="CaseObject",
          location=@Location(value=Kind.LINE,line=5)
       )
       public static void traceExecute(@ProbeClassName String pcn,@ProbeMethodNameString pmn,int line){
         println(strcat(strcat(strcat("call ",pcn),"."),pmn));
       }
    }


限制
---------------------------------------
* 不能创建对象
* 不能创建数组
* 不能抛出和捕获异常
* 不能调用任何对象方法和静态方法
* 不能给目标程序中的类静态属性和对象的属性进行赋值
* 不能有外部、内部和嵌套类
* 不能有同步块和同步方法
* 不能有循环(for, while, do..while)
* 不能继承任何的类
* 不能实现接口
* 不能包含assert断言语句

这些限制其实是可以使用unsafe模式绕过。通过声明*@BTrace(unsafe = true) annotation 并且以unsafe 模式-u运行btrace

实际使用非安全模式跟踪时，发现一个问题，一个进程如果被安全模式btrace探测过一次， 后面再使用非安全模式进行探测时非安全模式不生效。

最佳实践
=======================================

.. code:: java

   /* BTrace Script Template */
    import com.sun.btrace.annotations.*;
    import static com.sun.btrace.BTraceUtils.*;

    @BTrace
    public class TracingScript {
    /* put your code here */
    /*指明要查看的方法，类*/
        @OnMethod(
            clazz="com.linlong.platform.versionmanage.managerservice.VersionService",
            method="deleteVersion",
            location=@Location(Kind.RETURN)
        )
    /*主要两个参数是对象自己的引用 和 返回值，其它参数都是方法调用时传入的参数*/
        public static void traceExecute(@Self com.linlong.platform.versionmanage.managerservice.VersionService object,Long id, @Return boolean result){
            println("调用堆栈！！");
            println(strcat("返回结果是：",str(result)));//打印结果
            jstack();
            println(str(id));//打印参数
            println("over");
       }

    }
    
    import static com.sun.btrace.BTraceUtils.*;
    import com.sun.btrace.BTraceUtils;
    import com.sun.btrace.annotations.*;

    @BTrace
    public class TraceAllDelegate {
        
        @TLS
        private static long startTime = 0;
        
        @OnMethod(clazz = "/com//.crm//.components//..*Delegate.*/", method = "/.*/")
        public static void startMethod(){
            startTime = BTraceUtils.timeMillis();
        }
        
        @OnMethod(clazz = "/com//.crm//.components//..*Delegate.*/", method = "/.*/", location = @Location(Kind.RETURN))
        public static void endMethod(){
            println(strcat("time taken=>",str(BTraceUtils.timeMillis()-startTime)));
            println("--------------------------------------");
        }
        
        @OnMethod(clazz = "/com//.crm//.components//..*Delegate.*/", method = "/.*/", location = @Location(Kind.RETURN))
        public static void print(@ProbeClassName String pcn,@ProbeMethodName String pmn) {
            println(pcn);
            println(pmn);
        }
    }
    //Kind.LINE
    import com.sun.btrace.annotations.*;
    import static com.sun.btrace.BTraceUtils.*;

    @BTrace
    public class AllLines {
        @OnMethod(
            clazz="com.linlong.platform.versionmanage.managerservice.VersionService",
            location=@Location(value=Kind.LINE, line=0)
        )
        public static void online(@ProbeClassName String pcn, @ProbeMethodName String pmn, int line) {
            println(pcn + "." + pmn +  ":" + line);
        }
    }

    //print field example 
        public static void traceExecute(@Self com.linlong.platform.versionmanage.managerservice.VersionService object, Long id, @Return com.linlong.platform.entity.VersionManagerEntity result){
            println("调用堆栈！！");
            println(strcat("返回结果是：",str(result)));
            println(result);
            printFields(result);
            println(get(field("com.linlong.platform.entity.VersionManagerEntity","versionName"), result));
            //jstack();
            //println(str(id));
            println("over");
        }
    // call method:
    @BTrace
    public class ExpandCapacityBtrace {
        @OnMethod(clazz = "java.util.HashMap", method = "resize",
                location = @Location(value = Kind.CALL, clazz = "/.*/", method = "/.*/"))
        public static void traceMapExpandCapacity(@ProbeClassName String probeClass, @ProbeMethodName String probeMethod,
                                                  @Self Object self,int newCapacity) {
            String point = Strings.strcat(Strings.strcat(probeClass, "."), probeMethod);//java/util/HashMap.resize
            Class clazz = classForName("java.util.HashMap");
            println(Strings.strcat(point, "======"));
            //获取实例protected变量
            Map.Entry[] table= (Map.Entry[]) get(field(clazz, "table", true), self);
            int threshold = getInt(field(clazz, "threshold", true), self);
            int size = getInt(field(clazz, "size", true), self);
            println(Strings.strcat("newCapacity:", str(newCapacity)));
            println(Strings.strcat("table.length:", str(table.length)));
            println(Strings.strcat("size:", str(size)));
            println(Strings.strcat("threshold:", str(threshold)));
            println(Strings.strcat(point, "------------"));
        }
    }



执行命令：

.. code::
    
    #btrace [-I <include-path>] [-p <port>] [-cp <classpath>] <pid> <btrace-script> [<args>]
    #include-path指定头文件的路径，用于脚本预处理功能，可选；
    #port指定BTrace agent的服务端监听端口号，用来监听clients，默认为2020，可选；
    #classpath用来指定类加载路径，默认为当前路径，可选；
    #pid表示进程号，可通过jps命令获取；
    #btrace-script btrace脚本如果以.java结尾，会先编译再提交执行。可使用btracec命令对脚本进行预编译。
    #args是BTrace脚本可选参数，在脚本中可通过$和$length获取参数信息

    ./bin/btrace -classpath /opt/apache-tomcat-8.0.50/webapps/CLL-Platform-Manager/WEB-INF/classes/ 31374 TracingScript.java
    #必须将自己的class文件加入到classpath中，否则会出现找不到包或者类的情况

    #btracec [-I <include-path>] [-cp <classpath>] [-d <directory>] <one-or-more-BTrace-.java-files>
    btracer <pre-compiled-btrace.class> <application-main-class> <application-args>
