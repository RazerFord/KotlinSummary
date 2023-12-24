### Интерфейсы

Для методов интерфейса можно задавать реализацию по умолчанию

```kotlin
interface Runnable {
    fun run() {
        TODO()
    }
}
```

Для того, чтобы создать статику в интерфейсе нужно использовать компаньон или `object`

В интерфейсе могут встречаться свойства. Но их нельзя материализовывать, но можно определить `get`-метод. Будет как метод по умолчанию

```kotlin
interface Base {
    val url: String
        get() = "https://www.default.com"
}
class C: Base {
    override val url: String = "https://www.yandex.ru"
}
```

```kotlin
interface Runnable {
    val i : Int
}

class Test(override val i: Int): Runnable {

}
```

В объявлении метода могут быть параметры со значением по умолчанию. В месте определения параметр надо указать, но свое значение по умолчанию указать нельзя

```kotlin
interface Runnable {
    fun test(v: String = "hello world")
}

class Test: Runnable {
    override fun test(v: String) { // можно
        println(v)
    }
}

class Test2: Runnable {
    override fun test(v: String = "Text") { // так уже нельзя
        println(v)
    }
}
```

Как это реализовано? Форме без параметра соответствует синтетический статический интерфейсный метод. Ему передается параметром объект. Он вызывает над объектом его реализацию метода. И передает значение по умолчанию

Можно обратиться к конкретному методу по умолчанию. Указываем `super` и тип в угловых скобках. Помогает в разрешении неоднозначностей. Или в вызове "скрытого" метода

```kotlin
interface I1 {
    fun m() = println("I - 1")
}
interface I2 {
    fun m() = println("I - 2")
}
class C: I1, I2 {
    override fun m() {
        super<I1>.m()
        super<I2>.m()
    }
}
```

```kotlin
interface I1 {
    fun m(v: Int = 10) = println("I - 1: $v")
}
interface I2 : I1 {
    override fun m(v: Int) = println("I - 2: $v")
}
class C: I1, I2 {
    init {
        super<I1>.m(1)
        super<I2>.m(2)
        m()
        //super<I1>.m() - так нельзя. Так как под капотом будет вызов виртуальной функции и у нас есть `this`. Тогда если сделаем вызов,
        // то вызовется реализация из `I2`, потому что она перекрывает реализацию `I1`
    }
}
```

Байт код
```java
// $FF: synthetic method
public static void m$default(I1 var0, int var1, int var2, Object var3) {
   if (var3 != null) {
      throw new UnsupportedOperationException("Super calls with default arguments not supported in this target, function: m");
   } else {
      if ((var2 & 1) != 0) {
         var1 = 10;
      }

         var0.m(var1); // виртуальный вызов
      }
   }
}
```

```kotlin
interface I1 {
    fun m(x: Int = 10, y: Int = 12, z: Int =1515) = println("I - 1: x")
}
class C: I1 {
    init {
        super.m(1, 2, 3)
        // super.m() // так нельзя. Параметры по умолчанию для `super` не разрешены. Так как вызов через `super` определяется в `JVM` в `compile`
    }
}
```

### Умолчания всякие

По умолчанию классы, методы и `val` финальные. Ключевое слово `open` отключает `final`

Методы интерфейсов не финальные

`override` подразумевает `open`, но иногда хочется положить этому конец, тогда можно перед `override` указать `final`

### Модификаторы доступа

По умолчанию все - `public`. `public` доступность везде. 
`protected` - видимость в только в наследниках

Вне класса `protected` нет

`private`-элемент класса виден только в классе. 

*Внешний класс не видит `private`-элементы
своих внутренних классов*

`private`-элемент файла - только в файле

### Internal

Обозначает видимость в рамках единицы сборки. Например, собираем библиотеку. Класс нужен много где в библиотеке. Но он служебный

### Внутренние классы

По умолчанию - внутренние классы статические

```kotlin
class C {
    private val x = 1

    inner class C1 { // внутренний класс
        private val xn = x
    }

    class C2(c: C) { // вложенный статический класс
        private val xn = c.x
    }
    
    fun c1(c: C1) {
//        println(c.xn) нельзя, так как внешний класс не видит `private` внутреннего класса
    }

    fun c2(c: C1) {
//        println(c.xn) нельзя, так как внешний класс не видит `private` внутреннего класса
    }
}
```

```kotlin
class Outer {
    class Static
    inner class Inner {
        init {
            println(this)       // 54bedef2
            println(this@Inner) // 54bedef2
            println(this@Outer) // 5ca123f1
        }
    }
}
```

### sealed-классы

Хотим убедить компилятор в том, что других
подклассов нет

Прямые наследники изолированных классов и интерфейсов должны быть объявлены в том же пакете. Наследники `sealed`-класса  не могут быть локальными или анонимными объектами

Если прямой наследник изолированного класса не помечен как изолированный, он может быть расширен любыми способами, разрешенными его модификаторами

```kotlin
sealed interface Error // имеет реализации только в том же пакете и модуле

sealed class IOError(): Error // расширяется только в том же пакете и модуле
open class CustomError(): Error // может быть расширен везде, где виден
```

Нужно, чтобы был не анонимный `package`

Было

```kotlin
fun eval(e: Expr): Int = when (e) {
    is Value -> e.v
    is Sum -> eval(e.e1) + eval(e.e2)
    else -> throw IllegalArgumentException("why ?")
}
```

Стало

```kotlin
fun eval(e: Expr): Int = when (e) {
    is Value -> e.v
    is Sum -> eval(e.e1) + eval(e.e2) // благодаря sealed
}
```

### Конструктор супер класса

Связь задается при объявлении класса. Страдает `single-responsibility`. Меньше вариантов стыковки. Вторичные связываются с суперклассом через первичного

```kotlin
open class Base(val x: Int)

class T(val v: Int) : Base(v) {
    constructor() : this(10)
}
```

### Исключения

- Бросить - `throw`
- Поймать - `try/catch/finally`
- Нет понятия `checked exception`
- `try` - выражение

### `use`

`try-with-resources` в явном виде нет. Есть метод `use`, как расширение `Closeable`

```kotlin
public inline fun <T : Closeable?, R> T.use(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    var exception: Throwable? = null
    try {
        return block(this)
    } catch (e: Throwable) {
        exception = e
        throw e
    } finally {
        when {
            apiVersionIsAtLeast(1, 1, 0) -> this.closeFinally(exception)
            this == null -> {}
            exception == null -> close()
            else ->
                try {
                    close()
                } catch (closeException: Throwable) {
                    // cause.addSuppressed(closeException) // ignored here
                }
        }
    }
}
```

```kotlin
Socket.use {
    it.port // it === Socket
}
```

### `with`

Хотим получить строку через `StringBuilder`. Можно в лоб завести переменную-билдер. Что-то с ней поделать. Вернуть результат. `Kotlin` позволяет сделать красивее

Было

```kotlin
fun alphabet(): Stri    LINENUMBER 17 L10
    ARETURN
ng {
    val result = StringBuilder()
    for (letter in 'A'..'Z') {
        result.append(letter)
    }
    result.append("\nNow I know the alphabet!")
    return result.toString()
}
```
Стало

```kotlin
fun alphabet() = with(StringBuilder()) {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I know the alphabet!")
    this.toString()
}
```

Реализация `with`

```kotlin
public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return receiver.block()
}
```
`block: T.() -> R` - анонимная функция, которая является расширением `T`

### `apply`

Близкий аналог `with`. Метод, а не функция.Возвращает `this`

Реализация

```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
    return this
}
```

```kotlin
class Point {
    var x: Int = 0
    var y: Int = 0
}

fun main() {
    val p = Point().apply {
        x = 2
        y = 3
    }
}
```
