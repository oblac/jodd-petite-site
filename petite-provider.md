# Petite Provider

_Petite_ providers are methods that provide bean instances for injection into targets when needed. Providers are defined by their name, that can be used on injection points.

`@PetiteProvider` annotation may be used to annotate provider methods.

### Provider types

There are two provider types: _static_ and _instance_.

**Static providers** are defined on static methods. For now, they can not be registered via annotations, but just with manual registration.

**Instance providers** are defined on instance methods of some _Petite_ bean. Here you can use annotations to mark the method as a provider.

### Example

Lets define instance provider on this bean:

```java
    @PetiteBean
    public class Solar {

        @PetiteProvider("planet")
        public Planet planetProvider() {
            return new Planet();
        }
    }
```

This _Petite_ bean defines provider with name '`planet`'. Here the name has been specified manually. You may omit provider name in annotation value - the provider name will be equal to method name, with stripped suffix `Provider` \(if exist\).

Providers are used by specifying their name on injection point. For example:

```java
    @PetiteBean
    public class Sun {

        @PetiteInject
        Planet planet;

        @Override
        public String toString() {
            return "Sun{" + planet + '}';
        }
    }
```

Here we specify the injection point '`planet`'. Since there is no other _Petite_ bean named the same, _Petite_ will lookup for provider and use provider method to get instance that will be injected into the field.

Note that provider method is registered in _Petite_ bean. Therefore, on the place of injection _Petite_ will lookup first for the bean that defines provider method! This way, for example, you can make providers that are different for each HTTP session, just by specifying that scope in provider method class.

