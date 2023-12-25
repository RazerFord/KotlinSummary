### Корутины

- Классическая корутина - это кусок логики,предполагающий ожидание чего-либо

- Таймаута, асинхронного ввода, ответа на запрос, завершения другой корутины

- В момент ожидания она дает возможность поработать другим

```kotlin
fun main() = runBlocking {
    println("start")
    launch {
        // доходит до delay снимает с потока
        // ставит себя в очередь, когда таймаут 
        // закончился
        // поток начинает исполнять другую 
        // корутину ожидающую работу
        delay(1000L)
        println("World!")
    }
    println("Hello")
}
```

`runBlocking` - портал в мир корутин. Он создает *контекст*, запускает первую корутину, ожидает завершения. `launch` запускает вторую корутину в *контексте*. Вторая корутина засыпает "правильным"
способом. Во время сна она не занимает нить. Корутинный фреймворк освободившуюся нить
может отдать кому-то. В нашем случае - первой корутине. По истечении таймаута - корутина снова займет какую-то нить

```kotlin
fun main() = runBlocking {
    repeat(100) {
        launch {
            delay(5000L)
            println(Thread.currentThread().id) // у всех будет один id
            //Корутины запускаются в одном потоке, если не указано инного
        }
    }
}
```

### Декомпозиция

- Засыпающие функции помечаются ключевым словом `suspend`

- Функция, вызываюшая `suspend`-функции, обязана быть `suspend`-функцией

- Вызов `suspend`-функции вне корутинного
контекста запрещен

### Job

- `launch` возвращает объект класса `Job`

- над `Job` можно вызвать `join` - и дождаться завершения

- можно вызвать `cancel` - попытаться прервать корутину. И это случится, как только `Job` войдет в
`suspend` - режим

```kotlin
fun main() = runBlocking {
    val job = launch {
    }
    job.cancel()
    println("canceled? " + job.isCancelled) // canceled? true
}
```

```kotlin
fun main() = runBlocking {
    val job = launch {
        ...
    }
    job.join() // снять себя с исполнения и дать возможность поработать другим
}
```

```kotlin
fun main() = runBlocking {
    val job = launch {
        throw Exception()
    }
    try {
        println("canceled? " + job.isCancelled)
        delay(123)
    } catch (ignored: Exception) {
        println("Catching")
        println("canceled? " + job.isCancelled)
    }
}
/*
 * canceled? false
 * Catching
 * canceled? true
 * Бросится само исключение
 */
```

- Концептуально корутина и порожденные ею корутины считаются частью единого решения

- И по умолчанию - если случается проблема в одном компоненте, это проблема всего решения

![diagram state](/images/diagram_state.png)

Контекст корутины включает себя такой элемент как диспетчер корутины. Диспетчер корутины определяет какой поток или какие потоки будут использоваться для выполнения корутины

```kotlin
@OptIn(ExperimentalCoroutinesApi::class)
fun main() = runBlocking {
    launch(newSingleThreadContext("Custom thread")) {
        println(Thread.currentThread().name) // Custom thread
    }
    delay(1)
}
```

Пример с несколькими детьми

```kotlin
fun main() {
    runBlocking(Dispatchers.Default) {
        launch {
            launch { delay(1000) }
            launch { delay(2000) }
            println("1 ${coroutineContext.job.children.toList()}") // 1 [child1, child2]
        }
        println("2 ${coroutineContext.job.children.toList()}") // 2 [child1]
    }
}
```

Если вызывать `cancel()`, у `Job`, у которой есть `Job`-дети, то `Job`-дети и сам родитель отменятся

### Свои корутины?

Каждой корутине соответствует объект реализующий интерфейс `Continuation`. Через него можно будить корутину, или получить этот объект.

Общая идея такая, что должны сделать свой `suspend`-метод, в нем нужно вызвать `suspendCoroutine`, внутри которого должны протащить в `handler`, `continuation`, внутри которого дергаем resume или `resumeWithException`

### Диспетчеризация

Корутины кооперативны. `Suspention point` - место, где корутина может
уступить место другим. Может возобновиться на другой нити. На какой именно - решает контекст

### Контекст

- Каждая корутина запускается в некотором контексте
- Контекст представлен интерфейсом `CoroutineContext`
- `CoroutineContext` - это своего рода словарь со своеобразным интерфейсом. У него есть `plus`, то есть можем складывать контексты, также есть `fold`

```kotlin
public interface CoroutineContext {
    ...
}
```

Контекст доступен через интерфейс `coroutineContext`. Можно посмотреть все ключи через `fold`

```kotlin
fun main() {
    runBlocking {
        coroutineContext.fold(Unit) { a, b ->
            println("key class" + b.key.javaClass)
            println("value class" + b[b.key]?.javaClass)
            println("key" + b.key)
            println("value" + b[b.key])
        }
    }
}
```

Можно получить `Job`

```kotlin
fun main() {
    runBlocking {
        // Получить текущую Job
        println(coroutineContext.job)
        // Так как Key является companion object
        println(coroutineContext[Job.Key])
        println(coroutineContext[Job])

        // Описывает способ исполнения coroutine
        println(
            coroutineContext[ContinuationInterceptor.Key]
        )
        println(
            coroutineContext[ContinuationInterceptor]
        )
    }
}
```

- Контекст можем включать несколько элементов

- В нашем случае - два: один описывает детали `Job`-а; другой - способ диспетчеризации

- В первом приближении - `job`-часть определяется типом корутины

- Часть дипетчеризации - тем, что было передано параметром в `launch` или аналог

- По умолчанию - передается `CoroutineContext.EMPTY`

- Что означает наследование способа диспетчеризации. То есть берем диспетчер предыдущего контекста и складываем с тем контекстом, что передается в `launch`

`Dispatchers.default` - количество потоков = количество ядер процессора

Пример с многими потоками 

```kotlin
fun current(): Thread = Thread.currentThread()
fun main() {
    println(current())
    runBlocking(
        Dispatchers.Default.limitedParallelism(3)
    ) {
        println(
            coroutineContext[ContinuationInterceptor]
        )
        repeat(100) { i ->
            launch {
                println("start $i: ${current()}")
                delay(3000)
                println("finish $i: ${current()}")
            }
            println("launched $i from ${current()}")
        }
    }
}
```

### Обработка ошибок в детях

- Можем сделать родителя супервизором
- Ему будут приходить извещения о проблемах детей
- Супервизор что-то с этим делает
- Может игнорировать, может перезапускать
- Супервизор - то что создает элементы параллелизма и обрабатывает ошибки
- Если родитель не супервизор, то падает все. Подобие акторных моделей
- 1 корутина = 1 `Continuation` (то что делать дальше). Диспетчер - частный случай контекста. У каждой корутины есть свой контекст, который может быть диспетчером. Не у каждого контекста может быть много корутин. Какой-то контекст может быть порожден на основе другого. `Scope` - набор взаимосвязанных корутин, которые связаны задачей, скоуп завершается, если завершились все корутины скоупа. По умолчанию `launch` создает задачи в родительском скоупе. Внутри скоупа действует принцип - один упал = все упали

```kotlin
fun main() {
    runBlocking(Dispatchers.Default) {
        launch {
            launch {
                val scope = CoroutineScope(
                    SupervisorJob()
                )
                scope.launch {
                    throw Exception()
                }
                delay(1000)
                println("success")// выполнится не смотря на выброс исключения
            }
        }
    }
}
```

Обработка исключений

```kotlin
fun main() {
    val action: suspend CoroutineScope.() -> Unit = {
        throw IOException()
    }
    runBlocking(Dispatchers.Default) {
        launch {
            launch {
                val scope = CoroutineScope(SupervisorJob())
                val job1 = scope.launch(block = action)
                job1.invokeOnCompletion {
                    it?.let {
                        // it : Throwable
                        scope.launch { action(scope) }
                    }
                }
            }
        }
    }
}
```

- `suspend`-функция сама по себе - не корутина
- `launch` порождает `Job`/корутину
- внутри могут вызываться `suspend`-функции
- их значения можем использовать при вызове других
- но `launch` ничего не возвращает
