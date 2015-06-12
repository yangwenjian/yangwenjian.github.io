


=============================================
Junit Test
=============================================
Junit测试是我最喜欢的测试，因为其测试覆盖度高，能最直接了当的检查我的代码。

Junit介绍
=============================================
最新的版本是junit4,比之前的不同，这次写测试更灵活方便，不用再继承自TestCase基类，名称也不用以test开头，而是只需要注解就可以实现了。
这是Java目前的发展方向，降低耦合性和配置的复杂度，增加灵活性，更多的面向切面编程，利用注解实现各种功能，spring也是再向这个方向发展。

源代码分析
=============================================

运行关键代码
---------------------------------------------

.. code:: java

    @Override
    public void run(final RunNotifier notifier) {
        EachTestNotifier testNotifier= new EachTestNotifier(notifier,
			getDescription());
	    try {
		    Statement statement= classBlock(notifier);
		    statement.evaluate();
	    } catch (AssumptionViolatedException e) {
		    testNotifier.fireTestIgnored();
	    } catch (StoppedByUserException e) {
		    throw e;
	    } catch (Throwable e) {
		    testNotifier.addFailure(e);	
	    }
    }	


参考资料
==============================================
http://panpei.net.cn/2013/11/01/analyze-junit-source-code/
http://www.ibm.com/developerworks/cn/java/j-lo-junit-src/
