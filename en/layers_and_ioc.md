# Layers and Inversion of Control

![CleanArchitectureLayers](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/CleanArchitectureLayers.png)

- [Domain layer](#domain-layer)
- [Data layer](data-layer)
  - [Repository](#repository)
- [Presentation layer](#presentation-layer)
    - [Model](#model)
    - [View](#view)
    - [Presenter](#presenter)
    - [Binding View with Presenter](#binding-view-with-presenter)

As mentioned earlier, an app architecture based on Clean Architecture principles can be divided into three layers:

- presentation
- domain
- data

Above presents layers interaction scheme. Those black arrows represent dependencies of one layer on another, and blue arrows represent data flow. As you can see, data and presentation layers depend on domain layer, i. e. they use its classes. But **domain** layer doesn't know anything about outer layers and uses only its own classes and interfaces. Next, we will explain each of those layers in more detail, and how they interact.

As can be seen from the scheme, all three layers can exchange data. It is worth mentioning that direct interaction between the **presentation** and the **data** layers must not be allowed. Data flow should go from the **presentation** layer to the **data** layer through the **domain** (this could be, for example, passing string with search query or user registration data). The same can happen and vice versa (for example, when we return search results).

## Domain layer

![DomainLayer](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/DomainLayer.png)

**Business logic** is rules, that describe how a business works (for example, a user cannot make a purchase for a price that more than his account balance). Business logic does not depend on the implementation of the database or UI. Business logic changes only when business requirements change.

Robert Martin divides business logic into two types: a specific for a concrete application and common for all apps (if you want to share your code between platforms).

**Entity** - contains application independent business rules. 

**Interactor** - an object that implements business logic for a specific application.

But it's all in theory. In practice, only Interactors are used. At least I have not seen any application that uses Entity. By the way, many confuse Entity with DTO (Data Transfer Object). The fact is that the Entity of Clean Architecture is not exactly the Entity that we are used to seeing.

**Use Case** - is a series of operations to achieve a goal. Example of a use case for user registration:

> 1. The system checks user data
> 2. The system sends data to the server for registration
> 3. The system informs the user about the successful registration or error
>
> Exceptional situations:
>
> 1. The user entered incorrect data (the system shows an error)

Let's see how it looks like in practice. Robert Martin suggests to create the separate class for each use case, that has a single method to run it. An example of such a class:

```kotlin
class RegisterUserInteractor(
        private val userRepository: UserRepository,
        private val registerDataValidator: RegisterDataValidator,
        private val schedulersProvider: SchedulersProvider
) {

    fun execute(userData: User): Single<RegisterResult> {
        return registerDataValidator.validate(userData)
                .flatMap { userData -> userRepository.registerUser(userData) }
                .subscribeOn(schedulersProvider.io())
    }

}
```

However, practice shows that with this approach, you get a huge number of classes, with a small amount of code. A better approach would be to create single Interactor for one screen, the methods of which implement a certain use case. For example:

```kotlin
class ArticleDetailsInteractor(
        private val articlesRepository: ArticlesRepository,
        private val schedulersProvider: SchedulersProvider
) {

    fun getArticleDetails(articleId: Long): Single<Article> {
        return articlesRepository.getArticleDetails(articleId)
                .subscribeOn(schedulersProvider.io())
    }

    fun addArticleToFavorite(articleId: Long, isFavorite: Boolean): Completable {
        return articlesRepository.addArticleToFavorite(articleId, isFavorite)
                .subscribeOn(schedulersProvider.io())
    }

}
```

As you can see, sometimes Interactor methods cannot contain business logic at all, and Interactor methods act as a proxy between Repository and Presenter.

If you notice, the Interactor methods return not just the result, but the classes of RxJava 2 (depending on the type of operation we use different classes - Single, Completable, etc.). This gives several advantages:

1. You don't need to create listeners to get results.
2. It's easy to switch threads.
3. It's easy to handle errors.


To switch between threads, we use the` subscribeOn` method, as usual, but we get the Scheduler not through the static methods of the Schedulers class, but with the [SchedulersProvider](#schedulersprovider). It will help us in the future when we want to test our code.

### Data layer

![DataLayer](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/DataLayer.png)

This layer contains everything about storing and managing data. It could be database, SharedPreferences, network or file system, as well as caching logic.

As a "bridge" between data and domain layer, there is Repository interface (in the original Uncle Bob's scheme it's called Gateway). The interface itself is stored in the Domain layer, but its implementation is stored in the Data layer. In doing so, domain layer classes don't know where the data comes from - from the database, the network or from somewhere else. That's why all caching logic should be contained in the data layer.

#### Repository

**Repository** - is the interface with which Interactor works. It describes what data Interactor wants to obtain from external layers. There may be several repositories in the application, depending on the task. For example, if we develop a news application, the repository that works with articles can be called ArticleRepository, and a repository for working with comments will be called CommentRepository . An example of a repository that works with articles:

```kotlin
interface ArticleRepository {

    fun getArticle(articleId: String): Single<Article>
    
    fun getLastNews(): Single<List<Article>>

    fun getCategoryArticles(categoryId: String): Single<List<Article>>

    fun getRelatedPosts(articleId: String): Single<List<Article>>

}
```

### Presentation layer

![PresentationLayer](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/PresentationLayer.png)

Presentation layer contains all UI components, such as views, Activities, Fragments, etc. Also, it contains Presenters and Views (or ViewModels if you use MVVM). In this tutorial, we will use MVP (Model-View-Presenter) pattern, but you can choose other one (MVVM, MVI).

The library helps to solve many problems related with Activity lifecycle. Moxy has base classes, there are `MvpView` and `MvpPresenter`, that must be extended by all your Views and Presenters. To save you from coding boilerplate classes Moxy uses Annotation Processing. For the correct work of code generation, you must use the special annotations provided by Moxy. More information about the library you can find [here](https://medium.com/redmadrobot-mobile/android-without-lifecycle-mpvsv-approach-with-moxy-6a3ae33521e).

#### Model

Model contains the business-logic and code for managing data. And because we use Clean Architecture + MVP, as Model will act Data and Domain layers.

#### View

View is responsible for how the data will be shown to the user. In the case of Android, View must be implemented by an Activity or Fragment. Also, View informs the Presenter of user interaction, such as button click or text input. There is example of View:

```kotlin
interface ArticlesListView : MvpView {

    fun showLoadingProgress(show: Boolean)
    fun showArticles(articles: List<Article>)
    fun showArticlesLoadingErrorMessage()

}
```

So far we have described View interface, i. e. which View methods the Presenter can call. Note that our View is inherited from the **MvpView**, which is part of the Moxy library. That is a must for correct library work.

#### Presenter

According to the MVP concept, View can not directly interact with the Model, so the bridge between them is Presenter. Presenter reacts to user actions that View has notified to it (such as pressing a button, list item click or text input), and then decides what to do next. For example, it could be a data request from the model and display them in the View. Presenter example:

```kotlin
@InjectViewState
class ArticlesListPresenter @Inject constructor(
        private val articlesListInteractor: ArticlesListInteractor,
        private val schedulersProvider: SchedulersProvider
) : MvpPresenter<ArticlesListView>() {

    init {
        loadArticles()
    }

    private fun loadArticles() {
        viewState.showLoadingProgress(true)
        articlesListInteractor.getArticles()
                .observeOn(schedulersProvider.ui())
                .subscribe(
                        { articles ->
                            getViewState().showLoadingProgress(false)
                            getViewState().showArticles(articles)
                        },
                        { throwable -> getViewState().showLoadingError() }
                )
    }

    fun onArticleSelected(article: Article) {
        ...
    }

}
```

All the necessary classes we pass through the Presenter's constructor. It's actually called "constructor injection".

When creating the Presenter's object we must pass dependencies required by the constructor. If there are a lot of them, the creation of Presenter will be rather difficult. Because we don't want to do this manually, we will delegate this work to the Component.

```kotlin
@Presenter
@Component(dependencies = ApplicationComponent::class)
interface ArticlesListComponent {

    val getPresenter() : ArticlesListPresenter

}
```

It will supply the necessary dependencies, and we need only get the Presenter instance by calling the **getPresenter()** method. If you have a question, "How do you pass arguments to Presenter then?", then look in the FAQ - there is a detailed description of this issue.

Sometimes you can find the code in which the DI-container (Component) is passed to the constructor, after which all the necessary dependencies inject in the fields:

```kotlin
@Inject
lateinit var articlesListInteractor: ArticlesListInteractor

init {
    component.inject(this)
}
```

However, this approach is incorrect, because it complicates the testing of the class and creates a bunch of unnecessary code. If in the first case we could just pass mocked classes through the constructor, then now we need to create a DI-container and pass it. Also, this approach makes the class dependent on a particular DI-framework, which is also not good.

Also, note that before we display the results from Interactor, we switch the UI thread via `observeOn (schedulersProvider.ui())`, because we don't know in advance in what thread we receive the data

#### Binding View with Presenter

In the context of Android developing, the Activity (or Fragment) acts as a View, so after creating the View interface, we need to implement it in our Activity or Fragment:

```kotlin
class ArticlesListActivity : MvpAppCompatActivity(), ArticlesListView {

    @InjectPresenter
    lateinit var presenter: ArticlesListPresenter

    @ProvidePresenter
    fun provideArticlesListPresenter(): ArticlesListPresenter {
        val component = DaggerArticlesListPresenterComponent.builder()
                .applicationcomponent(MyApplication.getComponent())
                .build()
        return component.getPresenter()
    }

    override fun onCreate(savedInstanceState: Bundle) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_articles_list)

    }

    override fun showArticles(articles: List<Article>) {
       ...
    }

    override fun showLoadingError() {
       ...
    }

}
```

For Moxy to correct work, our Activity must extend **MvpAppCompatActivity** (or **MvpAppCompatFragment** for Fragments). To inject an `ArticlesListPresenter` instance into `presenter` field we use ```@InjectPresenter``` annotation. 

Since Presenter has arguments we must provide Presenter instance via ```provideArticlesListPresenter```, that we annotated with ```@ProvidePresenter```. Note that all annotated fields and methods must have public or package-private visibility.

### Packages organization

We recommend using a *feature based* package structure for your code. Here is an example of package structure for a news app:

```
com.mydomain
|
|----data
|     |---- database
|     |     |---- NewsDao
|     |---- filesystem
|     |     |---- ImageCacheManager
|     |---- network
|     |     |---- NewsApiService
|     |---- repositories
|     |     |---- ArticlesRepositoryImpl
|     |     |---- CategoriesRepositoryImpl
| 
|---- domain
|     |---- global
|     |     |---- models
|     |     |     |---- Article
|     |     |     |---- Category
|     |     |---- repositories
|     |     |     |---- ArticlesRepository
|     |     |     |---- CategoriesRepository
|     |---- articledetails
|     |     |---- ArticleDetailsInteractor
|     |---- articleslist
|     |     |---- ArticlesListInteractor
|
|---- presentation
|     |---- mvp
|     |     |---- global
|     |     |     |---- routing
|     |     |     |     |---- NewsRouter
|     |     |---- articledetails
|     |     |     |---- ArticleDetailsPresenter
|     |     |     |---- ArticleDetailsView
|     |     |---- articleslist
|     |     |     |---- ArticlesListPresenter
|     |     |     |---- ArticlesListView
|     |---- ui
|     |     |---- global
|     |     |     |---- views
|     |     |     |---- utils
|     |     |---- articledetails
|     |     |     |---- ArticleDetailsActivity
|     |     |---- articleslist
|     |     |     |---- ArticlesListActivity
|     
|---- di
|     |---- global
|     |     |---- modules
|     |     |     |---- ApiModule
|     |     |     |---- ApplicationModule
|     |     |---- scopes
|     |     |---- modifiers
|     |     |---- ApplicationComponent
|     |---- articledetails
|     |     |---- ArticleDetailsComponent
|     |     |---- ArticleDetailsModule
|     |---- articleslist
|     |     |---- ArticleListComponent
```

First of all, we divided the code into layers: data, domain, presentation. Also, we created a separate package for the DI code. The packages **domain**, **presentation** and **di** are divided by features, and the **data** package is divided by data source type (network, database, file system). This is because all features use the same classes (for example, **NewsApiService**) and it will be very difficult to divide them into features.

Packages named **global** contain common classes that are used in several features. For example, the **data/global** package stores models and repository interfaces.

The **presentation** layer is divided into two packages - **mvp** and **ui**. In the **mvp** are stored Presenter and View classes, as the name implies. In **ui** is stored the implementation of the View layer from MVP, i.e. Activity, Fragments, etc.

This structure has the following benefits:

- **Understandability.** Any developer can tell what functions there are in the application without looking in the code.
- **Easier to add new feature**. If you want to add a new feature to the application, for example, viewing the user profile, then you just need to add the **userprofile** package and work only with it, rather than going through entire package structure for creating the necessary classes.
- **Easier to edit a feature.** When you edit a feature, you need to keep at most two or three packages open and you see only those classes that relate to current feature. When you divide classes by the type, you have to keep open almost the whole tree of packages and you have to see the classes that you do not need now, related to other features.
- **Scalability and easier code navigation**. With the increasing number of app's functions, the number of classes also increases. When you divide classes by type, adding new classes makes navigation among them very uncomfortable since you need to search for the necessary class, among dozens of others, which affects the speed of development. Dividing by features solves this problem because you can combine related packages (for example, you can combine **login** and **registration** packages into the **authentication** package).

I'd like to say a few words about package naming: should package names be singular or plural? I hold the approach described [here](https://softwareengineering.stackexchange.com/a/75929):

1) Use singular for packages with heterogeneous classes. For example, a package, that contains classes like **Dog**, **Cat** and **Cow** will be called **animals**. Another example is different implementations of the same interface (**XmlResponseAdapter**, **JsonResponseAdapter**).
2) Use the plural for packages with homogeneous classes. For example, **order** package, that contains **OrderInfo**, **OrderInteractor**, **OrderValidation**, etc.