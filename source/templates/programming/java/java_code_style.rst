

=================================================
Java Code Style
=================================================

Sonar
=================================================
SonarQube为我们提供了很多关于编程规范的建议，具体可以参考sonar官方网站。
这里摘录一些常见的错误。

Sonar典型问题
-------------------------------------------------
1. *<Architecture>* Don't break the Law of Demeter. Anthor call is 'Principle of Least Knowledge'. Each unit should have only 
   limited knowledge about other units: only units "closely" related to the current unit. Each unit should only 
   talk to its friends; don't talk to strangers. Only talk to your immediate friends.
#. *<Architecture>* Classes should not be coupled to too many other classes (Single Responsibility Principle).

.. code:: java 

    //Noncompliant Code Example 错误代码
    class Foo{
        T1 t1;
        T2 t2;
        T3 t3;

        public T4 compute(T5 a, T6 b){
            T7 result = a.getResult(b);
            return result;
        }

        public static class Bar{
            T8 t8;
            T9 t9;
        }

    }

3. Classes should not have too many methods. A class that grows too much tends to aggregate too many responsibilities
    and inevitably becomes harder to understand and therefore to maintain.
#. "switch" statements should not have too many "case" clauses. A real map structuctor would be more readable and 
    maintainable.
#. *<Changability>* Magic numbers should not be used.

.. code:: java

    //Noncompliant Code Example 错误代码
    public void doSomething(){
        for(int i = 0; i < 4; i++){ // Noncompliant, 4 is magic number that no ones knows.
            ...
        }
    }

    //Compliant Code Example 正确代码
    public static final int NUMBER_OF_CYCLES = 4;
    public void doSomething(){  
        for(int i = 0; i < NUMBER_OF_CYCLES; i++){
            ...
        }
    }

6. Control flow statements "if", "for", "while", "switch" and "try" should not be nested too deeply.

.. code:: java

    //Noncompliant Code Example
    if(condition1){
        ...
        if(condition2){
            for(int i = 0; i < 10; i++){
                ...
                if(condition4){
                    if(condition5){
                        ...
                    }
                    return;
                }
            }
        }
    }



7. *<Maintainability>* A field should not duplicate the name of its containing class.

.. code:: java

    //Noncompliant Code Example 错误代码
    public class Foo{
        private String foo;
        public String getFoo(){ ... }
    }

    Foo foo = new Foo();
    foo.getFoo(); // What does this return?

8. *<Readability>* "switch case" clauses should not have too many lines.

.. code:: java
    
    //Noncompliant Code Example 错误代码
    switch(variable){
        case 0:
            methodCall1();
            methodCall2();
            methodCall3();
            ...
            break;
        case 1:
        ...
    }

    //Compliant Code Example 正确代码
    switch(variable){
        case 0:
            doSomething();
            break;
        case 1:
        ...
    }
    private void doSomething(){
        methodCall1();
        methodCall2();
        methodCall3();
        ...
    }

9. *<Readability>* Variables should not be declared before they are relevant.

.. code:: java

    //Noncompliant Code Example 错误代码
    public boolean isCondition(int a, int b){
        int difference = a - b;
        MyClass foo = new MyClass(a); // Noncompliant; not used before the first return.
        if(difference < 0){
            return false;
        }
        if(foo.isSomething()){
            return true;
        }
        return false;
    }

10. Loops should not contain more than a single "break" or "continue" statement.

.. code:: java

    //Noncompliant Code Example 错误代码
    for( int i = 0; i <= 10; i++){
        if(i % 2 == 0){
            continue;
        }
        ...
        if(i % 3 == 0){
            continue;
        }
        System.out.println("i = " + i);
    }

11. Methods should not have too many return statements.

.. code:: java

    //Noncompliant Code Example 错误代码
    public boolean myMethod(){
        if(condition1){
            return true;
        }else{
            if(condition2){
                return false;
            }else{
                return true;
            }
        }
        return false;
    }

12. Fields and methods should not have conflicting names.
#. Files should not have too many lines.
#. The ternary operator should not be used.
#. Classes should not be too complex.
#. Files should contain only one top-level class or interface each.
#. Inner classes should not have too many lines.
#. Methods should not have too many lines.
#. "switch" statements should not have too many "case" clauses.
#. Public methods should not contain selector arguments.
#. Dependencies should not have "system" scope.



Sonar其他问题
-------------------------------------------------

1. "equals(Object object)" and "hashCode()" should be overridden in pairs. 
2. "equals(Object object)" should test 
   argument type, because equals() takes a generic object as a parameter.

.. code:: java

    //Compliant Code Example 正确代码
    public boolean equals(Object object){
        if(object == null)
            return false;
        if(this.getClass() != object.getClass())
            return false;
        MyClass mc = (MyClass) object;
        ...
    }

3. "Iterator.hasNext()" should not call "Iterator.next()", hasNext() method should not have any side effects.

.. code:: java
    
    //Noncompliant Code Example 错误代码
    @Override //override Iterator method
    public boolean hasNext(){
        if(next() != null){
            return true;
        }
        return false;
    }

4. "Object.wait(...)" should never be called on objects that implement "java.util.concurrent.locks.Condition", 
   should use "Object.await()" method.

.. code:: java

    //Noncompliant Code Example 错误代码
    final Lock lock = new ReentrantLock();
    final Condition notFull = lock.newCondition();
    notFull.wait();

    //Compliant Code Example 正确代码
    notFull.await();

5. "PreparedStatement" and "ResultSet" methods should be called with valid indices. They all start with 1.

.. code:: java

    //Noncompliant Code Example 错误代码
    PreparedStatement ps = con.PreparedStatement("select name from employees where hireDate > ? and salary < ?");
    ps.setDate(0, date);//Noncompliant, should be 1.
    ps.setDouble(1, salary);//Noncompliant, should be 2.
    ResultSet rs = ps.excuteQuery();
    while(rs.next()){
        String name = rs.getString(0);//Noncompliant, should be 1.
    }

6. "return" statements should not occur in "finally" blocks. Returning from a finally block suppresses the propagation of any unhandled Throwable 
   which was thrown in the try or catch block.

.. code:: java

    //Noncompliant Code Example 错误代码
    public static void doSthThrowsException(){
        try{
            throw new RuntimeException();
        } finally{
            return; //NonCompliant, it will prevent the throw action.
        }
    }

7. "ScheduledThreadPoolExecutor" should not have 0 core threads.
8. A "for" loop update clause should move the counter in the right direction, or it will be infinit.
9. Blocks synchronized on fields should not contain assignments of new objects to those fields.

.. code:: java

    //Noncompliant Code Example 错误代码
    private String color = "red";
    private void doSomething(){
        synchronized(color) {
            // lock is actually on object instance "red" referred to by the color variable
            color = "green"; // Noncompliant; other threads now allowed into this block
            // ...
        }
    }

10. Conditions should not unconditionally evaluate to "TRUE" or to "FALSE", or it will not called condition.

.. code:: java

    //Noncompliant Code Example 错误代码
    if(foo == bar && ... & foo != bar){ ...  }
    if(foo == 4){
        if(foo > 4) // Noncompliant, at this point foo is equal to 4.
    }

11. There should be no cycles in style definitions as this can lead to runtime exceptions.
12. Methods "wait(...)", "notify()" and "notifyAll()" should never be called on Thread instances. For two reasons: Doing so is really confusing. What 
    is really expected when calling, for instance, the wait(...) method on a Thread? That the execution of the Thread is suspended, or that acquisition 
    of the object monitor is waited for? Internally, the JVM relies on these methods to change the state of the Thread (BLOCKED, WAITING, ...), so calling 
    them will corrupt the behavior of the JVM.
13. Null pointers should not be dereferenced.

.. code:: java

    //Noncompliant Code Example 错误代码
    @CheckForNull
    String getName(){ ... }
    public boolean isNameEmpty(){
        return getName().length() == 0; //Noncompliant; should be null-checked.
    }

    Connection conn = null;
    try{
        conn = DriverManager.getConnection(DB_URL, USER, PASS);
    } catch(Exception e){
        e.printStackTrace();
    } finally{
        conn.close();////Noncompliant, could be null if an exception is thrown.
    }

    private void merge(@NonNull Color firstColor, @NonNull Color secondColor){...}
    public void append(@CheckForNull Color color){
        merge(currentColor, color); ////Noncompliant, should be null-checked because merge() doesn't accept nullable parameters.
    }

    void paint(Color color){
        if(color == null){
            System.out.println("color " + color.getName() + " is null!");
            return;
        }
    }

14. Reflection should not used to check non-runtime annotations. Like '@Override' annotation, it is marked as RetentionPolicy.SOURCE. Reflectin can only
    read the annotation with 'RetentionPolicy.SOURCE' mark.

.. code:: java
    
    //Noncompliant Code Example 错误代码
    Method m = String.class.getMethod("getBytes", new Class[]{int.class, int.class, byte[].class, int.class});
    if(m.isAnnotationPresent(Override.class)){...} //Noncompliant, will alaways return false, even @Override is present.

15. Resources should be closed. Like connections, streams, files and other classes implement the Closeable must be manually close after creation.

.. code:: java

    //Noncompliant Code Example 错误代码
    OutputStream stream = null;
    try{
        for(String property : propertyList){
            stream = new FileOutputStream("myfile.txt");
        }
    } catch(Exception e){
    } finally{
        stream.close();//Mutiple streams were opened. Only the last is closed.
    }

16. super.finalize() should be called at the end of Object.finalize() implementations.

.. code:: java

    //Noncompliant Code Example 错误代码
    protected void finalize(){
        super.finalize();   //Noncompliant; this call should come last.
        releaseSomeResources();
    }

17. Synchronization should not be based on Strings or boxed primitives. It should use object.

.. code:: java

    //Noncompliant Code Example 错误代码
    private static final Boolean bLock = Boolean.FALSE;
    private static final Integer iLock = Integer.valueOf(1);
    private static final String sLock = "LOCK";
    public void doSomething(){
        synchronized(bLock){... }//Noncompliant
        synchronized(iLock){... }//Noncompliant
        synchronized(sLock){... }//Noncompliant
    }

18. "equals" should not be used to test values of "Atomic" classes.

.. code:: java

    //Noncompliant Code Example 错误代码
    AtomicInteger aInt1 = new AtomicInteger(0);
    AtomicInteger aInt2 = new AtomicInteger(0);
    if(aInt1.equals(aInt2)) { //AtomicClass do not override equals() method.
        ...
    }

19. The value returned from a stream read should be checked.

.. code:: java

    //NonCompliant Code Example 错误代码
    public void doSomething(){
        try{
            InputStream istream = new InputStream(file);
            byte[] buffer = new byte[1000];
            istream.read(buffer); //Noncompliant
            /**
            *   正确代码
            *   while(count  = istream.read(buffer) > 0)
            */
        }
    }

20. "read" and "readLine" return values should be used.

.. code:: java

    //Noncompliant Code Example 错误代码
    buffReader = new BufferedReader(new FileReader(fileName));
    while(BufferedReader.readLine() != null){//Noncompliant should use string to get the return value.
        ...
    }

21. Short-circuit logic should be used to prevent null pointer dereferences in conditionals.

.. code:: java

    //Noncompliant Code Example 错误代码
    if(str == null && str.length() == 0){
        System.out.println("String is empty.");
    }
    if(str != null || str.length() > 0){
        System.out.println("String is no empty.");
    }

22. Throwable and Error should not be caught.

23. "BigDecimal(double)" should not be used. The result can be unpredictable. We should use 'BigDecimal.valueOf(1.1)' instead.
24. "Calendars" and "DataFormats" should not be static. Because neither of them is thread safe.
25. "ConcurrentLinkedQueue.size()" should not be used. It is an expensive operation, and may be inaccurate if the queue is modified during execution.
26. "equals" methods should be symmetric and work for subclasses.

.. code:: java

    //Noncompliant Code Example 错误代码
    public class Fruit extends Food{
        private Season ripe;
        public boolean equals(Object object){
            if(obj == this){
                return true;
            }
            if(obj == null){
                return false;
            }
            if(Fruit.class == obj.getClass()){ //Noncompliant, broken fro child class.
                return ripe.equals(((Fruit)obj).getRipe()); 
            }
            if(obj instanceof Fruit){ //Noncompliant, broken fro child class.
                return ripe.equals(((Fruit)obj).getRipe());
            }
        }
    }

27. "equals(Object obj)" should be overriden along with the "compareTo(T obj)" method. In java documentation, it strongly recommanded 
    that (x.compareTo(y) == 0) == (x.equals(y)).
28. "Exception" should not be caught when not required by called methods. Catching "Exception" traps all exception types and so both 
    checked and runtime exceptions, casting too broad a net. The exceptions should be explicitly listed in the catch clause.
29. "hashCode" and "toString" should not be called on array instances. 

.. code:: java

    public static void main(String[] agrs){
        //Noncompliant Code Example 错误代码
        String argstr = args.toString();
        int argHash = args.hashCode();
        //Compliant Code Example 正确代码
        String argstr1 = Arrays.toString(args);
        int argHash1 = Arrays.hasCode(args);
    }

30. "indexOf" checks should not be for positive numbers. Because "indexOf" can return 0 if it is found at the first place. "indexOf"
    should check ' > -1 ' or ' >= 0 ' if you need.
31. "notifyAll" should be used. Since "notify" only rouses one, might not wake up the right thread, "notifyAll" should be used instead.
32. "object == null" should be used instead of "object.equals(null)".
33. "Object.wait(...)" and "Condition.await(...)" should be called inside a "while" loop.

.. code:: java

    //Noncompliant Code Example 错误代码
    synchronized(obj){
        if(!suitableCondition()){
            obj.wait(); //the thread can wake up whereas the condition is still false. We should use 'while'
        }
    }

34. "public static" fields should be constant. It is used to share a state among several objects.If we don't make such variables final, 
    any object can do whatever it wants, such as making it null.
35. "Serializable" inner classes of non-serializable classes should be "static". Because serializing a non-static inner class will result
    in an attempt at serializing the outer class as well.
36. "toString()" and "clone()" methods should not return null.
37. "wait(...)" should be used instead of "Thread.sleep(...)" when a lock is held. Calling "Thread.sleep(...)" to hold lock cloud lead to
    performance and scalability issues, or deadlocks.

.. code:: java

    //Noncompliant Code Example 错误代码
    public void doSomething(){
        synchronized(monitor){
            while(notReady()){
                Thread.sleep(2000); // Noncompliant, should be "monitor.wait(2000)" instead.
            }
            process();
        }
    }

38. "wait(...)", "notify()" and "notifyAll()" methods should only be called when a lock is obviously held on an object. By contract, the method 
    Object.wait(...), Object.notify() and Object.notifyAll() should be called by a thread that is the owner of the object's monitor.

.. code:: java

    //Noncompliant Code Example 错误代码
    private void removeElement(){
        while(!suitableCondition()){
            obj.wait(); //Noncompliant
            wait(); //Noncompliant
        }
    }
    
    //Compliant Code Example 正确代码
    private void removeElement(){
        synchronized(obj){
            while(!suitableCondition()){
                obj.wait(); // Compliant, this thread is the owner of obj.
            }
        }
    }

39. Collections should not be passed as argument to their own methods.

.. code:: java

    //Compliant Code Example 正确代码
    List<Object> objs = new ArrayList<Object>();
    objs.add("Hello");
    objs.add(objs); // Noncompliant, will throw StackOverflowException if obj.hashCode() called.
    objs.addAll(objs); // Noncompliant, behavior undefined.
    objs.containsAll(objs); //Noncompliant, always true.
    objs.removeAll(objs); //Noncompliant, confusing. Use clear() instead.
    objs.retainAll(objs); //Noncompliant, NOOP

40. Dissimilar primitive wrappers should not be used with the ternary operator without explicit casting.

.. code:: java

    //Noncompliant Code Example 错误代码
    Integer i = 123456789;
    Float f = 1.0f;
    Number n = condition ? i : f; //Noncompliant, i is coeredto float. n = 1.23456792E8
    //Compliant Code Example 正确代码
    Number n = condition ? (Number) i : f;

41. Exception handlers should preserve the orginal exception.

.. code:: java

   //Noncompliant Code Example 错误代码
   try{ ... }catch(Exception e){ LOGGER.info("context"); } // Noncompliant, exception is lost.
   try{ ... }catch(Exception e){ LOGGER.info(e.getMessage()); } // Noncompliant, only message is preserved.
   try{ ... }catch(Exception e){ throw new RuntimeException("context"); } // Noncompliant, exception is lost.
   //Compliant Code Example 正确代码
   try{ ... }catch(Exception e){ LOGGER.info(e); }
   try{ ... }catch(Exception e){ throw new RuntimeException(e); }

42. Exceptions should not be thrown from servlet methods. Failure to catch exceptions in a servlet cloud leave 
    system in a vulnerable state, and send debug message to users.
43. Exceptions should not be thrown in finally blocks. Finally blocks should be used to do clean up works.
44. Fields in a "Serializable" class should either be tranient or serializable.
45. Floating point numbers should not be tested for equality.
46. Instance methods should not write to "static" fields. Static fields are only updated from synchronized static 
    methods.
47. Lazy initialization of "static" fields should be "synchronized".In a multi-threaded situation, un-synchronized 
    lazy initialization of non-volatile fields could mean that a second thread has access to a half-initizliaed 
    object while the first thread is still creating it. Allowing such access could cause serious bugs. Instead. 
    The initizliation block should be synchronized or the variable made volatile.

.. code:: java

    //Noncompliant Code Example 错误代码
    protected static Object instance = null;
    /**
    * 正确代码应为： protected static volatile Object instance = null;
    */
    public static Object getInstance(){
        if(instance != null){
            return instance;
        }
        instance = new Object(); // Noncompliant
        return instance;
    }

48. Locks should be released. The logic in a method should ensure that locks are released in the methods in which
    they were acquired.

.. code:: java

    //Noncompliant Code Example 错误代码
    Lock lock = new Lock();
    public void acquireLock(){
        lock.lock();
    }
    public void releaseLock(){
        lock.unlock();
    }
    public void doSomething(){
        acquireLock();
        ...
        releaseLock();
    }

    //Compliant Code Example 正确代码
    Lock lock = new Lock();
    public void doSomething(){
        lock.lock();
        ...
        lock.unlock();
    }
    
49. Math operands should be cast before assignments.

.. code:: java

    //Noncompliant Code Example 错误代码
    float twoThirds = 2/3;
    long millisInYear = 1000*3000*24*365;
    long bigNum = Integer.MAX_VALUE + 2; // Yields -2147483647
    long bigNegNum = Integer.MIN_VALUE -1; // positive result
    public long compute(int factor){
        return factor * 10000; // if factor > 214748
    }

50. Math should not be performed on floats.

.. code:: java

    //Noncompliant Code Example 错误代码
    float a = 16777216.0f;
    float b = 1.0f;
    float c = a + b; // Noncompliant; yields 1.6777216E7
    double d = a + b; // Noncompliant; between two floats.
    //Compliant Code Example 正确代码
    BigDecimal c = BigDecimal.valueOf(a).add(BigDecimal.valueOf(b));
    double d = (double) a + (double) b;

51. Modulus results should not be checked for direct equality.

.. code:: java

    //Noncompliant Code Example 错误代码
    public boolean isOdd(int x){
        return x % 2 ==1;
    }
    //Compliant Code Example 正确代码
    public boolean isOdd(int x){
        return x % 2 != 0; // 或者 return Math.abs(x%2) != 1;
    }

52. Mutable members should not be stored or returned directly. Or the value will be changed at other class.
53. Non-serializable classes should not be written.
54. Non-serializable objects should not be stored in "HttpSessions". Some servers automatically write their
    active sessions out to file at shutdown & deserialize any such sessions at startup.
55. Objects should not be created to be dropped immediately without being used.
56. Return values should not be ignored when function calls don't have any side effects.

.. code:: java

    //Noncompliant Code Example 错误代码
    public void handle(String command){
        command.toLowerCase(); // Noncompliant, result will be thrown away.
    }

57. Servlets should not have mutable instance fields. Because the fields are shared by all users.

.. code:: java

    //Noncompliant Code Example 错误代码
    public Class MyServlet extends HttpServlet{
        private String userName; // Noncompliant.
    }

58. The Array.equals(Object obj) method should not be used. Because array doesn't override equals(), 'equals()' 
    is equal to ' == '. Use Arrays.equals(array1, array2) instead.
59. The non-serializable super class of a "Serializable" class should have a no-argument constructor.
    When a Serializable object has a non-serializable ancestor in its inheritance chain, object deserialization 
    (re-instantiating the object from file) starts at the first non-serializable class, and proceeds down the 
    chain, adding the properties of each subsequent child class, until the final object has been instantiated.

60. Thread.run() and Runnable.run() should not be called directly. Calling those methods will cause their code 
    to be excuted in the current thread.
61. Throwable.printStackTrace(...) should not be called. Use 'LOGGER.log("context", e)' instead.
62. Values passed to OS commands should be sanitized. Values passed to LDAP queries should be sanitized. Values
    passed to SQL commands should be sanitized.
63. Values should not be uselessly incremented.

.. code:: java

    //Noncompliant Code Example 错误代码
    publi int incr(){
        int i = 0;
        int j = 0;
        i = i++; // Noncompliant, i is 0.   
        return j++; // Noncompliant, return 0.
    }

64. "@Override" annotation should be used on any method overriding.

参考资料
=================================================
http://nemo.sonarqube.org/coding_rules#languages=java
https://google.github.io/styleguide/javaguide.html
