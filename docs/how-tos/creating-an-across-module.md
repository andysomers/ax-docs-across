# Creating an Across Module

## Structure

The structure of an Across module is very similar to the structure of an Across application, like the ones generated by [Across Initializr](/start-across.foreach.be). To maintain clarity, we strongly advise you to include _modules_ in the artifact name and end with the name of your module. e.g. `com.foreach.example.modules.my` is the package of `MyModule`. Under com.foreach.example.modules we would then typically have the following sub-packages: _config_, _domain_ and _extensions_.

Beans in the _config_ package are automatically picked up and scanned by our module. This is most often the folder where our global configuration is found as well. The `DomainConfiguration`class often has an `@EnableAcrossJpaRepositories` annotation to pick up the repositories present in the module.

The _domain_ package typically holds several subfolders in which our domain model is defined in a Domain-Driven Design fashion. It is often practical to have a marker class to refer to from for example our @EnableAcrossJpaRepositories annotation.

The _extensions_ package is a special one, just like _config_, where we usually put our configuration classes for other modules, a typical example is `AcrossHibernateJpaModule`. The Across Framework will by default look for `@ModuleConfiguration` classes within the config and extensions sub-packages.

In the end our module structure looks something like this

```
com.foreach.example.modules.my
    | config
     \ DomainConfiguration.java // global configuration
    | domain
     \ DomainMarker.java // marker class for our domain model
    | extensions 
     \ EntityScanConfiguration.java //moduleconfiguration for AcrossHibernateJpaModule
    \ MyModule.java // Module descriptor
```

So what's different with the structure generated by [Across Initialzr](/start-across.foreach.be)? It does not contain the package _modules_, but rather an _application_ package, which holds the above sub-packages and will be automatically scanned for beans.

## Module descriptor

The module descriptor for our Across module is nothing more than a simple class that extends `AcrossModule`. `AcrossModule` provides various methods to further customize settings for your module and requires your module to at least implement `getName()`. Typically we create a `public static final String NAME` which holds our module name.

Aside of the name of our module, we often also define the `@AcrossDepends` annotation, where we can specify which modules are required and which modules are optional to use our new module.

For example MyModule just defines a simple shared domain model and as such will have repositories, so it would look something like this:

```java
@AcrossDepends(required = {
        AcrossHibernateJpaModule.NAME
})
public class MyModule extends AcrossModule{
    public static final String NAME = "MyModule";

    @Override
    public String getName() {
        return NAME;
    }
}
```

## Beans

As we have noticed earlier, by default the _config_ and _extensions_ packages are scanned for beans. To be able to scan for other beans, Services for example, we will need to provide additional configuration. We can do this in a two ways, the first one being an `@ComponentScan` annotation, which simply says where should be scanned for components, Or by overriding the `registerDefaultApplicationContextConfigurers( Set<ApplicationContextConfigurer> contextConfigurers ) `and adding a `ComponentScanConfigurer`.

```java
@Override
protected void registerDefaultApplicationContextConfigurers( Set<ApplicationContextConfigurer> contextConfigurers ) {
    contextConfigurers.add( new ComponentScanConfigurer( AdvantageModule.class ) );
}
```

Since our module will have its own application context, for beans to be visible they need to be exposed. By default all beans annotated with @Exposed or @Service will be exposed to other contexts. This means that if you define repositories in your module and you want to allow other modules or applications to use them, they will have to be annotated with @Exposed. However if you wish to expose all your repositories without annotating them with @Expose, you can also override the default expose filter.

```java
public MyModule(){
    setExposefilter( new BeanFilterComposite( AcrossModule.defaultExposeFilter(), new ClassBeanFilter( Repository.class ) ) );
}
```

## Tests

For our module to be useable by other modules and applications, it is very important that it works correctly. Hence why you should always provide tests, at the very least a few integration tests to check your module works as intended. You should definitely take a look at the [Testing section](/docs/testing/index.adoc) for more information about Across Test.
