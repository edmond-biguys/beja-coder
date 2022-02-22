### internal
internal 关键字可以修饰方法、属性，被internal关键字修身的方法、属性只能在当前的module中被访问，无法跨module使用。
在kotlin中，跨module时，Android studio无法找到被internal关键字修饰的方法、属性。
如果你在java中调用时会出现如下情况，可以调用，但编译报错，
```
imageOperations.getAbcInternal$lib_debug(); //编译报错 Usage of Kotlin internal declaration from different module
imageOperations.getAbcNormal(); //正常
```
### open 
在kotlin中，所有的class、方法，默认都是final的，也就是不能背继承，不能重写，需要使用open关键字修饰后，才能被继承、重写。

### inline
inline关键字修饰的方法，在编译时，会被直接写到代码中，而不是作为方法调用，普通方法也可以加inline关键字，但系统不建议这么做，普通方法加inline对于性能几乎没有提升。
inline用于带有方法参数的方法中，可以有效避免使用lambda表达式时，生成的匿名内部类。同时为了避免编译后的.class内容过大，inline修饰的方法不易过长，参考系统的apply、let等方法，基本都是一行方法。
> Expected performance impact from inlining is insignificant. Inlining works best for functions with parameters of functional types

在inline方法中，如果有多个block参数，其中部分，不希望变成inline，可以使用```noinline```来修饰，如下

```
fun test() {
        val m = {
            println("fff")
        }
        foo({
            println("abc")
        }, m)
}
    
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {
        inlined()
        notInlined()
}
```
```
public final void test() {
      Function0 m = (Function0)null.INSTANCE;
      int $i$f$foo = false;
      int var4 = false;
      String var5 = "abc";
      System.out.println(var5);
      m.invoke();
}
```
可以看到带noinline修饰的方法，不会被直接写到代码中。

crossinline使用，要明白crossinline使用，需要先明白local return和 non-local return
local return表示，return后从当前函数返回他的调用者处，如下
```
    private fun foo01() {
        foo02()
        println("foo01 end")
    }
    
    private fun foo02() {
        println("foo02")
        return
    }
```
上边方法foo02执行return后，foo01继续执行"foo01 end"打印。

non-local return表示，当前return调用后，不返回他的调用处，如下，
```
    private fun foo03() {
        foo04 {
            println("inlined method")
            return
        }
        println("foo03 end")
    }

    private inline fun foo04(inlined: () -> Unit) {
        inlined()
        println("foo04")
    }
```
上边方法foo03中，return后，将不会打印"foo03 end"和"foo04"，因为foo04是inline方法，反编译得到方法如下，
```
    private final void foo03() {
      int $i$f$foo04 = false;
      int var3 = false;
      String var4 = "inlined method";
      System.out.println(var4);
   }
```
明白了non-local return后，我们再来看下crossline，先来看下官方说明，
> Note that some inline functions may call the lambdas passed to them as parameters not directly from the function body, 
> but from another execution context, such as a local object or a nested function. In such cases, non-local control flow is 
> also not allowed in the lambdas. To indicate that the lambda parameter of the inline function cannot use non-local returns, 
> mark the lambda parameter with the crossinline modifier:


英语不好，理解了好久，大概意思就是，在inline方法中，传进来的block参数，有可能不是直接使用， 而是会在当前方法的嵌套方法中使用，为了标明这个block参数不能使用non-local return，
可以使用crossinline关键字。

如果还是觉得绕，可以简单理解为crossinline关键标记的参数，不能直接return，有强制提醒的作用。
    

### reified


### inner
用来修饰内部类，内部类（inner）与嵌套类（nested）区别如下，
1. 内部类需要使用inner class，而嵌套类使用class。
2. 内部类持有一个外部类的引用，所以内部类可以直接访问外部类成员属性、成员函数，而嵌套类则不行。












