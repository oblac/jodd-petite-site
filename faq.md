# FAQ

Everything you always wanted to know about _Petite_ \(but was afraid to ask\).

### Do I really have to use annotations?

Of course NOT! By default _Petite_ relies on annotations, but you are not forced to use them. There are two annotation-free alternatives:

* _automatic registration_, where all fields and methods and arguments are potential injection points. If there is a bean with matching reference name it will be injected as a dependency.
* _manual registration_, where you do all the wiring by yourself. You even have a fluent interface for that :\) Power-to-the-people!

