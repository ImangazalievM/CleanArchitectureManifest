

# Дополнительные сущности, используемые на практике

- [Router](#router)
- [Mapper](#mapper)
- [ResourceManager](#resourcemanager)
- [SchedulersProvider](#schedulersprovider)

## Дополнительные сущности, используемые на практике

### Router

Т. к. Presenter содержит в себе логику реагирования на действия пользователя, то он также знает о том, на какой экран нужно перейти. Однако сам Presenter не может осуществлять переход на новый экран, т. к. для этого нам требуется Context. Поэтому за открытие нового экрана должна отвечать View. Для осуществления перехода на следующий экран мы должны вызвать метод View, например, **openProfileScreen()**, а уже в реализации самого метода осуществлять переход. Помимо данного подхода некоторые разработчики используют для навигации так называемый Router.

**Router** - класс, для осуществления переходов между экранами (активити или фрагментами).

Для реализации Router'а вы можете использовать библиотеку [Alligator](https://github.com/aartikov/Alligator).

### Mapper

**Mapper** - специальный класс, для конвертирования моделей из одного типа в другой, например, из модели БД в модель бизнес-логики. Обычно они имеют название типа XxxMapper, и имеют единственный метод с названием map (иногда встречаются названия convert/transform), например:

```kotlin
class ArticleDbModelMapper {

	fun map(model: ArticleDbModel) =
         Article(
                 name = model.name,
                 lastname = model.lastname,
                 age = model.age
         )
    
    fun map(models: Collection<ArticleDbModel>) = models.map { map(it) }

}
```

Т. к. слой **domain** ничего не знает о классах других слоев, то маппинг моделей должен выполняться во внешних слоях, т. е. репозиторием (при конвертации **data** > **domain** или **domain** > **data**) или презентером (при конвертации **domain** > **presentation** и наоборот) .

### ResourceManager

В некоторых случаях может потребоваться получить строку или число из ресурсов приложения в Presenter'е или слое **domain** . Однако, мы знаем, что они не должны напрямую взаимодействовать с фреймворком Android. Чтобы решить эту проблему мы можем создать специальную сущность ResourceManager, для доступа к внешним ресурсам. Для этого мы создаем интерфейс:

```kotlin
interface ResourceManager {

    fun getString(resourceId: Int): String

    fun getInteger(resourceId: Int): Int

}
```

Сам интерфейс должен располагаться в слое **domain**. После этого в слое **presentation** мы создаем реализацию нашего интерфейса:

```kotlin
class AndroidResourceManager @Inject constructor(
        private val context: Context
) : ResourceManager {

    override fun getString(resourceId: Int): String {
        return context.resources.getString(resourceId)
    }

    override fun getInteger(resourceId: Int): Int {
        return context.resources.getInteger(resourceId)
    }

}
```

Далее мы должны связать интерфейс и реализацию нашего ResourceManager'а в ApplicationModule:

```kotlin
@Singleton
@Provides
fun  provideResourceManager(resourceManager: AndroidResourceManager) : ResourceManager = resourceManager
```

Теперь мы можем использовать ResourceManager в Presenter'е или Interactor'ах:

```kotlin
@InjectViewState
class ArticlesListPresenter @Inject constructor(
	private val resourceManager: AndroidResourceManager
) : MvpPresenter<ArticlesListView>() {
    
    private fun onLoadError(throwable: Throwable) {
        ...
        
		viewState.showMessage(resourceManager.getString(R.string.articles_load_error))
    }

}
```

Наверное, у внимательных читателей возник вопрос: почему мы используем класс **R** в Presenter'е? Ведь он также относится к Android? На самом деле, это не совсем так. Класс **R** вообще не использует никакие классы, и представляет из себя набор идентификаторов ресурсов. Поэтому, нет ничего плохого, чтобы использовать его в Presenter'е.

### SchedulersProvider

Перед началом тестирования нам нужно сделать все операции синхронными. Для этого мы должны заменить все Scheduler'ы на **TestScheduler**, поэтому мы не устанавливаем Scheduler'ы напрямую через класс **Schedulers**, а используем **SchedulersProvider**:

```kotlin
open class SchedulersProvider @Inject constructor() {

    open fun ui(): Scheduler = AndroidSchedulers.mainThread()
    open fun computation(): Scheduler = Schedulers.computation()
    open fun io(): Scheduler = Schedulers.io()
    open fun newThread(): Scheduler = Schedulers.newThread()
    open fun trampoline(): Scheduler = Schedulers.trampoline()

}
```

Благодаря этому мы можем легко заменить Scheduler'ы на нужные нам, всего лишь создав наследника класса SchedulersProvider'а и переопределив методы:

```kotlin
class TestSchedulerProvider : SchedulersProvider() {

    val testScheduler = TestScheduler()

    override fun ui(): Scheduler = testScheduler
    override fun computation(): Scheduler = testScheduler
    override fun io(): Scheduler = testScheduler
    override fun newThread(): Scheduler = testScheduler
    override fun trampoline(): Scheduler = testScheduler

}
```

Далее, при самом тестировании, нам нужно будет лишь использовать TestSchedulersProvider вместо SchedulersProvider. Более подробно о тестировании кода с RxJava можно почитать [здесь](https://github.com/Froussios/Intro-To-RxJava/blob/master/Part%204%20-%20Concurrency/2.%20Testing%20Rx.md).