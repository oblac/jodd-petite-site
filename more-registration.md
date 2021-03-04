# More Registration

Some more registration topics.

### Registering implementations

_Petite_ register beans by their names. When working with simple POJOs, it is convenient to have bean names automatically generated from bean's class name. However, when there is an interface or abstract class to implement or extend with custom implementation, it is wise to name implementing bean with the interface name.

Here is some interface:

```java
    public interface Biz {
        void calculate();
    }
```

As said, implementation would be registered into _Petite_ using interface name `biz`:

```java
    @PetiteBean("biz")
    public class DefaultBiz implements Biz {
        public void calculate() {}
    }
```

Now injection reference may be defined simply as:

```java
    public class BizUsage {

        @PetiteInject
        Biz biz;
        ...
    }
```

_Petite_ will inject the implementation: `DefaultBiz`.

### Duplicated bean names

By default, when newly registered bean has the same bean name as one of already registered beans, the old bean registration will be simply discarded and the new one will be used. This might be important when providing custom implementations - the only important thing is the order of registration.

Nevertheless, _Petite_ may be configured to detect duplicated bean names by setting this flag to `true`.

### Manual registration

_Petite_ \(i.e. `PetiteContainer`\) offers methods for registering beans and for defining injection points and initial methods. Therefore, it is possibly to register and define everything in _Petite_ using just Java, i.e. using manual registration.

_Petite_ container configuration consist of:

* beans,
* scopes,
* init methods,
* injection points,
* provider definitions, and
* properties.

For each part of configuration, there is at least one method that registers it, like: `registerPetiteBean`, `registerPetitePropertyInjectionPoint`, `registerPetiteInitMethods`, etc.

When manually registering beans, there is one important thing to be aware of. There are two ways how a bean can be registered:

* **default** registration - on first lookup, registered beans will

  scanned for init methods, provider definitions and injection points

  \(using annotations, if any found\). This is, therefore, semi-manual

  registration in case if you have _Petite_ annotations in your bean

  classes.

* **defined** - beans will be registered completely empty and all

  annotations \(if exist\) will be ignored.

#### PetiteRegistry

_Petite_ provides helper class with only purpose to provide fluent registration: `PetiteRegistry`.

Here is how manual registration may look like:

```java
    PetiteContainer pc = new PetiteContainer();
    PetiteRegistry r = pc.createContainerRegistry();

    r.bean(SomeService.class).register();
    r.bean(PojoBean.class).name("pojo").register();

    r.wire("pojo").ctor().bind();
    r.wire("pojo").property("service").ref("someService").bind();
    r.wire("pojo").method("injectService").ref("someService").bind();
    r.init("pojo").invoke(POST_INITIALIZE).methods("init").register();
```

### Various ways of registration

Full, manual registration in plain Java may be unmaintainable and hard to follow. Because of _Petite_ offers API for registration, there is unlimited number of ways how beans may be registered into the container. It is easy to build new system for beans registration, based on XML or on some other way, or to use different annotations and so on. Moreover, it is possible to influence the way how beans are registered and to utilize the whole process, as it will be shown next.

One real-life example is the following situation: some module consist of business components that are wired together using internal _Petite_ container. User of this module is and should not be aware of _Petite_, but it still should be able to register custom versions of components and the new one as well. When registering new versions, the module prefers overriding of existing components instead of writing the completely new class, since logic behind components is a bit complex. In one word, for this module it is preferable to `extends` than to `implement`.

To hide _Petite_ from module user, the following registration logic is being used. Module offers registration of the component types. Each time, module resolves the name of base class, i.e. the first class in the class hierarchy \(not including `Object`, obviously\). So base name is used when registering module component:

```java
    ...
    private String resolveBaseComponentName(Class component) {
        while(true) {
            Class superClass = component.getSuperclass();
            if (superClass.equals(Object.class)) {
                break;
            }
            component = superClass;
        }
        return PetiteUtil.resolveBeanName(component);
    }

    public final void registerComponent(Class component) {
        String name = resolveBaseComponentName(component);
        pc.removeBean(name);
        pc.registerBean(name, component);
    }
    ...
```

Custom version of existing component are registered with the names of their base classes. In case of this example, that was sufficient and yet simple solution. Later the above code was enhanced to skip abstract classes, but this is trivial thing to do and out of scope of this document.

More enhanced solution may be created from above example: one that performs more thoughtful checks of all super classes and/or interfaces, or to check what is the topmost annotated component and so on.

### Configure and register, then use

There is nothing that prevents from using _Petite_ before all beans are registered. However, registering later some bean that replaces existing one might lead to unpredictable results. Although Petite will remove deprecated bean from internal structures as well from its scope, already injected instances of deprecated bean would stay alive.

It is strongly recommended to first configure _Petite_ and to register all beans prior the usage.

### Adding objects

_Petite_ allows any external object instance to be added as singleton bean into the container. Such instance is created outside of container and than assigned manually to it. As said, it becomes part of singleton scope, since _Petite_ doesn't know how instance was created.

Usually, after adding, bean should be wired and its init method should be invoked:

```java
    PetiteContainer pc = new PetiteContainer();
    pc.registerBean(Foo.class, null, null, null, false);
    pc.registerBean(Zoo.class, null, null, null, false);
    Boo boo = new Boo();
    pc.addBean("boo", boo).wire(boo, true);
    ...
    Boo boo2 = (Boo) pc.getBean("boo");
```

In this example, `boo` and `boo2` points to the same instance. Furthermore, _Petite_ performs injection into `boo` instance.

### Container self-registration

It is possible to registers _Petite_ container instance into itself, simply by using `addSelf()` method.

