# objc-0905
@class @import
#import与@class的区别
1.import会包含这个类的所有信息，包括实体变量和方法，而@class只是告诉编译器，其后面声明的名称是类的名称，至于这些类是如何定义的，暂时不用考虑，后面会再告诉你。

2.在头文件中， 一般只需要知道被引用的类的名称就可以了。 不需要知道其内部的实体变量和方法，所以在头文件中一般使用@class来声明这个名称是类的名称。 而在实现类里面，因为会用到这个引用类的内部的实体变量和方法，所以需要使用#import来包含这个被引用类的头文件。

3.在编译效率方面考虑，如果你有100个头文件都#import了同一个头文件，或者这些文件是依次引用的，如A–>B, B–>C, C–>D这样的引用关系。当最开始的那个头文件有变化的话，后面所有引用它的类都需要重新编译，如果你的类有很多的话，这将耗费大量的时间。而是用@class则不会。

4.如果有循环依赖关系，如:A–>B, B–>A这样的相互依赖关系，如果使用#import来相互包含，那么就会出现编译错误，如果使用@class在两个类的头文件中相互声明，则不会有编译错误出现。

所以，一般来说，@class是放在interface中的，只是为了在interface中引用这个类，把这个类作为一个类型来用的。 在实现这个接口的实现类中，如果需要引用这个类的实体变量或者方法之类的，还是需要import在@class中声明的类进来.

 

 

举个例子说明一下。

在ClassA.h中
#import ClassB.h 相当于#include整个.h头文件。如果有很多.m文件#import ClassA.h，那么编译的时候这些文件也会#import ClassB.h增加了没必要的#import，浪费编译时间。在大型软件中，减少.h文件中的include是非常重要的。

如果
只是 ClassB 那就没有include ClassB.h。仅需要在需要用到ClassB的.m文件中 #import ClassB.h

那么什么时候可以用呢？
如果ClassA.h中仅需要声明一个ClassB的指针，那么就可以在ClassA.h中声明
@ClassB
...
ClassB *pointer;





假设，有两个类：ClassA和ClassB，两个之间相互使用到，即构成了circular dependency（循环依赖）。如果在头文件里面只用#import把对方的头文件包含进来（构成circular inclusions，循环包含），则编译器会报错：

Expected specifier-qualifier-list before ‘ClassA’

或者

Expected specifier-qualifier-list before ‘ClassB’

为了避免循环包含，在ClassA.h文件里面用@class classB把classB包含进来，同样，在ClassB.h文件里面用@class ClassA把ClassA包含进来。@class指令只是告诉编译器，这是个类，保留个空间来存放指针就可以了。

接下来，很可能在ClassA.m和ClassB.m中会有访问包含进来对象的成员的情况，这时必须让编译器知道更多信息，比如那个类有些什么方法可以调用，就必须用#import，再次把用到的类包含进来，告诉编译器所需要的额外信息。

否则，编译器会警告：

warning: receiver ‘ClassA’ is a forward class and corresponding @interface may not exist

还有另一种情况，使用有Categories的类，要在.h头文件里用#import把Categories包含进来。

总之，使用原则是：

头文件里面只#import超类 消息文件里面#import需要发消息过去的类 其他地方就用@class转向声明
