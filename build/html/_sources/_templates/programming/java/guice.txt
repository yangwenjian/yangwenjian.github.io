


=============================================
Guice Introduction and Analysis
=============================================


Basic Information
============================================
The essence of DI is to have your classes accept their dependencies through references to interfaces instead of constructing them (or using static references).

Motivation
--------------------------------------------
Object come to you, instead of 'new' and factory(factories aren't fun);
Reusable Modules;
First-class scopes;
Easier tests and more confidence;
Less boilerplate.

Using guice
--------------------------------------------
@inject is the new 'new'


Personal comprehension
--------------------------------------------
When we need to create an instance, we know that we have to use 'new' to create one. But it will lead our code is dependent by the real implemation, so we create design pattern called 'Factory Mode', but the factory mode still has some shortcoming, Some bigger man in google works on the project called guice.
When we use facotry we still have some dependency, such as class or some name.But when we use guice, we can bind implementation class to abstract class. If we change implementation code, we only have to change the dependency binding. In this case, some class extends AbstractModule(from guice).

.. code:: java

    public class BillingModule extends AbstractModule {
    @Override 
        protected void configure() {
            bind(TransactionLog.class).to(DatabaseTransactionLog.class);
            bind(CreditCardProcessor.class).to(PaypalCreditCardProcessor.class);
            bind(BillingService.class).to(RealBillingService.class);
        }
    }

So, we don't use new or factory, or we even don't have to specify the instance name in String. All work is done in guice, we only have to make bind and use some annotaion('@') in the implementation class

.. code:: java

    public static void main(String[] args) {
        Injector injector = Guice.createInjector(new BillingModule());
        BillingService billingService = injector.getInstance(BillingService.class);
        ...
     }


For more information:
https://code.google.com/p/google-guice/wiki/
