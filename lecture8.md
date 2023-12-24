### Generics

Требуют обязательного указывания `generic-type`

```kotlin
class Test<T>

fun main() {
    val test: Test<Int> = Test()
//    val test1 = Test() нельзя без указания типа
}
```

### Generic-methods

Могут иметь свои типы-параметры. И типы-параметры объемлющего класса. При совпадении имен внутреннее
приоритетнее внешнего. В функциях могут определяться локальные классы с параметрами-типами. Или в локальных классах использоваться параметры-типы функции

```kotlin
fun <T> f(s: T): Callable<Int> {
    class C<E>(private val v: T) : Callable<Int> {
        override fun call(): Int {
            return v.hashCode()
        }
    }
    return C<String>(s)
}
```

```kotlin
class Test<T> {
    fun <T> test(v: T) {
        println(v)
    }
}

fun main() {
    val o = Test<String>()
    o.test<Int>(42)
}
```

### Generic в расширениях

Можно использовать как в составе параметра, так и в составе расширяемого типа

```kotlin
fun <T> T.printIf(f: (T) -> Boolean) {
    if (f(this)) {
        println(this)
    }
}
```

### Особенности

Конкретное расширение имеет приоритет над обобщенным. Если возникает неоднозначность - ошибка компиляции.

```kotlin
fun <T> T.test() {
    println("T help")
}

fun <E : Exception> E.test() {
    println("Exception help")
}

fun String.test() {
    println("String help")
}

fun main() {
    1.test()
    "asd".test()
    Exception().test()
}
```

```kotlin
fun <T> T.m(other: String) {
    println("2")
}

fun <T> Int.m(other: T) {
    println("3")
}

fun main() {
    // 1.m("hello") ошибка компилятор не сможет выбрать перегрузку
}
```

```kotlin
fun <T> T.m(other: String) {
    println("2")
}

fun <T> Int.m(other: T) {
    println("3")
}

fun Int.m(other: String) {
    println("4")
}

fun main() {
    1.m("hello") // выведется 4
}
```

### Реализация

Как обычное расширение - только тип параметр меняется на `java.lang.Object`. Как для `Any`.
`Generic`-расширение может сочетаться с
одноименным расширением конкретного типа. Но если этим типом будет `Any` - получим конфликт и ошибку компиляции

### Nullable

`Nullable` в сигнатурах методов реализуется
добавлением аннотации. И добавкой проверки на `null` для необнуляемых параметров. Это означает невозможность иметь два метода с одним именем, отличающихся только обнуляемостью параметра

```kotlin
fun test(s: String) {}
// fun test(s: String?) {} нельзя - ошибка компиляции
```

`Nullable`-тип может быть подставлен в тип-параметр. Знак вопрос можно добавить к типу-параметру. Синтаксически можно получить "двойной `nullable`". Если используется что-то типа `T?`. А в `T` подставляется `String?`

```kotlin
fun <T> m1(a: T?) {
    println(a)
}
fun <T> m2(a: T) {
    println(a)
}
fun main() {
    m1("sad")
    m1(null)
    m2("sad")
    m2(null)
}
```

### Nothing

`Nothing` - отдельный тип, который символизирует вычисление, которое не завершится. Например, из-за цикла или исключения

Результат функции, которая не завершится штатно, можно присвоить обычной типизированной переменной. Или объявить функцию как что-то возвращающую - и при этом бросить исключение

Нельзя объявить функцию как возвращающую `Nothing`. И попытаться вернуть `15`, `"hello"`

```kotlin
fun f1(): Nothing {
    throw RuntimeException()
}

fun f2(): Nothing {
    while (true) {}
}

fun main() {
    val q1: Int = f1()
    println(q1)
    val q2 = f2()
    //println(q2) не откомпилируется, так как нет toString()
}
```

```kotlin
sealed interface Option<T>
data object None: Option<Nothing>
data class Some<Q>(val value: Q): Option<Q>

fun main() {
    val strOpt = Some("hello")
    val intOpt = Some(112233)
// val strOpt2: Option<String> = None
// val intOpt2: Option<Int> = None
}
```

### Ковариантность

Хотим, чтобы `Option<A>` был подтипом `Option<B>`, если `A` - подтип `B`

```kotlin
sealed interface Option<out T>
data object None : Option<Nothing>
data class Some<Q>(val value: Q) : Option<Q>

fun main() {
    val strOpt = Some("hello")
    val intOpt = Some(112233)
    val strOpt2: Option<String> = None
    val intOpt2: Option<Int> = None
}
```

### Вариантность

- Есть понятия "ковариантная позиция" и "контравариантная позиция"

- Ковариантая - чтение значения в широком смысле

- Контравариантная - запись в широком смысле

- Если тип-параметр всегда используется в ковариантных позициях - можно сделать ковариантным

- Если только в контравариантных - сделать контравариантным

- Если и так, и так - инвариантным