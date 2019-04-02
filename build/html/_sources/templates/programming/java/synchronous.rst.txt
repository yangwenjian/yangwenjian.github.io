

=========================================
Synchronous And Asynchronous
=========================================

Example
-----------------------------------------

.. code:: java

    public static void main(String args[]) throws InterruptedException {
        long start = System.currentTimeMillis();
        print();
        long end = System.currentTimeMillis();
        System.out.println("time is: " + (end - start));
        start = System.currentTimeMillis();
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        Callable<Integer> callable = new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                 print();
                return 1;
            }
        };
        
        executorService.submit(callable);
        end = System.currentTimeMillis();
        System.out.println("time is: " + (end - start));
    }
    public static void print() throws InterruptedException {
        Thread.sleep(3000);
        System.out.println("Hello World");
    }

