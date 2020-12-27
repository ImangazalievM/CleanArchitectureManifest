# Additional entities used in practice

- [Router](#router)
- [Mapper](#mapper)
- [ResourceManager](#resourcemanager)
- [SchedulersProvider](#schedulersprovider)

### Router

Since Presenter decides what happens when you interact with the view, it also know what screen will be opened. But we can't open Activity from Presenter directly. There are two ways of solving this problem - call View method like **openProfileScreen()** and open new screen from Activity or use Router

**Router** is a special class for navigating between screens (Activities or Fragments).

We recommend to use [Alligator](https://github.com/aartikov/Alligator) library for implementing navigation.

### Mapper

**Mapper** is a special class for converting models between layers, for example, from DB model to Domain model. Usually, they called like XxxMapper and have a single method with name map (or convert/transform), for example:

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

Since domain layer doesn't know anything about classes from other layers, the mapping of models must be performed in the outer layers, i. e. in Repository (when mapping **data > domain** or **domain> data**) or in Presenter (when mapping **domain> presentation** and vice versa) .

### ResourceManager

In some cases, we need to get a string or a number from resources and use it in Presenter or domain layer. But we know that we can't use Context class. To resolve this problem we must use a special class that called ResourceManager. Let's create an interface for this one:

```kotlin
interface ResourceManager {

    fun getString(resourceId: Int): String

    fun getInteger(resourceId: Int): Int

}
```

This interface must be contained in the domain layer. Then we create the interface implementation in presentation layer:

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

After that we must bind the interface with its implementation in ApplicationModule:

```kotlin
@Singleton
@Provides
fun  provideResourceManager(resourceManager: AndroidResourceManager) : ResourceManager = resourceManager
```

Now we can use ResourceManager in our Presenters or Interactors:

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

Perhaps you have a question: "Why we can use **R** class in the Presenter?" Because it uses Android Framework. Actually, that's not entirely true. **R** class does not use any class at all. So there is nothing bad with using **R** class in Presenter.

#### SchedulersProvider

To test our code we need make all operations synchronous. For that, we must replace all Schedulers to **TestScheduler**. For this reason, we set Schedulers through **SchedulersProvider**, but not directly.

```kotlin
open class SchedulersProvider @Inject constructor() {

    open fun ui(): Scheduler = AndroidSchedulers.mainThread()
    open fun computation(): Scheduler = Schedulers.computation()
    open fun io(): Scheduler = Schedulers.io()
    open fun newThread(): Scheduler = Schedulers.newThread()
    open fun trampoline(): Scheduler = Schedulers.trampoline()

}
```

It allows easily replace Schedules by extending **SchedulersProvider** and overriding it's methods:

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

Then, when testing we just need use TestSchedulersProvider instead of SchedulersProvider. More info about testing code with RxJava you can find [here](https://github.com/Froussios/Intro-To-RxJava/blob/master/Part%204%20-%20Concurrency/2.%20Testing%20Rx.md).