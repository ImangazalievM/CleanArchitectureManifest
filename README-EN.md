# Clean Architecture Manifest (v. 0.9.5)

Here you will find description of the main principles and rules, that are worth following in developing Android apps using Clean Architecture approach.

![CleanArchitectureManifest](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/CleanArchitectureManifest.png)

## Table of contents

- [Introduction](#introduction)
- [Layers and Inversion of Control](#layers-and-inversion-of-control)
  - [Domain layer](#domain-layer)
  - [Data layer](#data-layer)
    - [Repository](#repository)
  - [Presentation layer](#presentation-layer)
    - [Model](#model)
    - [View](#view)
    - [Presenter](#presenter)
    - [Binding View with Presenter](#binding-view-with-presenter)
- [Additional entities used in practicе](#additional-entities-used-in-practice)
  - [Router](#router)
  - [Mapper](#mapper)
  - [ResourceManager](#resourcemanager)
  - [SchedulersProvider](#schedulersprovider)
- [Errors handling](#errors-handling)
- [Testing](#testing)
- [Start a new application development using Clean Architecture](#start-a-new-application-development-using-clean-architecture)
- [Migration project to Clean Architecture](#migration-project-to-clean-architecture)
- [Clean Architecture FAQ](#clean-architecture-faq)

## Introduction

Clean Architecture is an approach suggested by Robert Martin in 2012.

Clean Architecture includes two main principles:

1. Separation into layers
2. Inversion of Control

Let's look at each of them.

**1. Separating into layers**

The sense of this principle is in the separation whole app code into layers. In general we have three layers:

1. Presentation layer

2. Domain layer (business logic) 

3. Data layer

**2.Inversion of Control**

According to this principle, domain layer must not depends on outer ones. That is, classes from outer layers must not be used in the domain layer. Interaction with outer layers is implemented through interfaces. Declaration of the interfaces contains in domain layer and their implementation contains in outer ones.

Thanks to separation of concerns between classes we can easily change an application code and add new functional with modifying minimal number of classes. In addition we get testable code. Please note that building the right app architecture depends entirely on a developer experience.

Advantages of Clean Architecture:

- independent of UI, DB or frameworks
- allows you to add new features faster
- a higher percentage of test coverage
- easy packages structure navigation

Disadvantages:

- large number of classes 
- hard for beginners to understand 

**Attention:** this article assumes knowledge on the following topics: 

- Dagger 2
- RxJava 2

**P. S.** I developed an application to demonstrate the use of Clean Architecture in practice. You can find the source code here - [Bubbble](https://github.com/ImangazalievM/Bubbble).

## Layers and Inversion of Control

![CleanArchitectureLayers](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/CleanArchitectureLayers.png)

As mentioned earlier, an app architecture based on Clean Architecture principles, can be divided into three layers:

- presentation
- domain
- data

Above presents layers interaction scheme. Those black arrows represent dependencies of one layers on another, and blue arrows represent data flow. As you can see, data and presentation layers depend on domain layer, i. e. they use it's classes. But **domain** layer doesn't know anything about outer layers and uses only its own classes and interfaces. Next, we will explain each of those layers in more detail, and how they interact.

As can be seen from the scheme, all three layers can exchange data. It is worth mentioning that direct interaction between **presentation** и **data** layers must not be allowed. Data flow should go from **presentation** layer to **data** layer through **domain** (this could be, for example, passing string with search query or user registration data). The same can happen and vice versa (for example, when we return search results).

## Domain layer

![DomainLayer](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/DomainLayer.png)

**Business logic** is rules, that describe how a business works (for example, a user cannot make a purchase for a price that more than his account balance). Business logic does not depend on the implementation of the database or UI. Business logic changes only when business requirements change.

Robert Martin divides business logic into two types: a specific for a concrete application and common for all apps (if you want to share your code between platforms).

**Entity** - contains application independent business rules. 

**Interactor** - an object that implements business logic for a specific application.

But it's all in theory. In practice, only Interactors are used. At least I have not seen any applications that uses Entity. By the way, many confuse Entity with DTO (Data Transfer Object). The fact is that the Entity of Clean Architecture is not exactly the Entity that we are used to seeing.

**Use Case** - is a series of operations to achieve a goal. Example of a use case for user registration:

> 1. The system checks user data
> 2. The system sends data to the server for registration
> 3. The system informs the user about the successful registration or error
>
> Exceptional situations:
>
> 1. The user entered incorrect data (the system shows an error)

Let's see how it looks like in practice. Robert Martin suggests to create separate class for each use case, that  has single method to run it. An example of such a class:

```java
public class RegisterUserInteractor {
  
    private UserRepository userRepository;
    private RegisterDataValidator registerDataValidator;
    private SchedulersProvider schedulersProvider;

    public RegisterUserInteractor(UserRepository userRepository, 
                                  RegisterDataValidator registerDataValidator,
                                  SchedulersProvider schedulersProvider) {
        this.userRepository = userRepository;
        this.registerDataValidator = registerDataValidator;
        this.schedulersProvider = schedulersProvider;
    }
  
    public Single<RegisterResult> execute(User userData) {
        return registerDataValidator.validate(userData)
                .flatMap(userData -> userRepository.registerUser(userData))
                .subscribeOn(schedulersProvider.io());
    }
  
}
```

However, practice shows that with this approach, you get a huge number of classes, with a small amount of code. A better approach would be to create single Interactor for one screen, the methods of which implement a certain use case. For example:

```java
public class ArticleDetailsInteractor {
  
    private ArticlesRepository articlesRepository;
    private SchedulersProvider schedulersProvider;

    public ArticleDetailsInteractor(ArticlesRepository articlesRepository,
                                    SchedulersProvider schedulersProvider) {
        this.articlesRepository = articlesRepository;
        this.schedulersProvider = schedulersProvider;
    }
  
    public Single<Article> getArticleDetails(long articleId) {
        return articlesRepository.getArticleDetails(articleId)
                          .subscribeOn(schedulersProvider.io());
    }
  
    public Completable addArticleToFavorite(long articleId, boolean isFavorite) {
        return articlesRepository.addArticleToFavorite(articleId, isFavorite)
                          .subscribeOn(schedulersProvider.io());
    }
  
}
```

As you can see, sometimes Interactor methods can not contain business logic at all, and Interactor methods act as a proxy between Repository and Presenter.

If you notice, the Interactor methods return not just the result, but the classes of RxJava 2 (depending on the type of operation we use different classes - Single, Completable, etc.). This gives several advantages:

1. You don't need to create listeners to get results.
2. It's easy to switch threads.
3. It's easy to handle errors.


To switch between threads, we use the` subscribeOn` method, as usual, but we get the Scheduler not through the static methods of the Schedulers class, but with the [SchedulersProvider](#schedulersprovider). It will help us in the future, when we want to test our code.

### Data layer

![DataLayer](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/DataLayer.png)

This layer contains everything about storing and managing data. It could be database, SharedPreferences, network or file system, as well as caching logic.

As a "bridge" between data and domain layer, there is Repository interface (in the original Uncle Bob's scheme it's called Gateway). The interface itself is stored in the Domain layer, but it's implementation is stored in the Data layer. In doing so, domain layer classes don't know where the data comes from - from the database, the network or from somewhere else. That's why all caching logic should be contained in the data layer.

#### Repository

**Repository** - is the interface with which Interactor works. It describes what data Interactor wants to obtain from external layers. There may be several repositories in the application, depending on the task. For example, if we develop a news application, the repository that works with articles can be called ArticleRepository, and a repository for working with comments will be called CommentRepository . An example of a repository that works with articles:

```java
public interface ArticleRepository {

  Single<Article> getArticle(String articleId);

  Single<List<Article>> getLastNews();
  
  Single<List<Article>> getCategoryArticles(String categoryId);
  
  Single<List<Article>> getRelatedPosts(String articleId);
  
}
```

### Presentation layer

![PresentationLayer](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/PresentationLayer.png)

Presentation layer contains all UI components, such as views, Activities, Fragments, etc. Also, it contains Presenters and Views (or ViewModels if you use MVVM). In this tutorial we will use MVP (Model-View-Presenter) pattern, but you can choose other one (MVVM, MVI).

The library helps to solve many problems related with Activity lifecycle. Moxy has base classes, there are `MvpView` and `MvpPresenter`, that must be extended by all your Views and Presenters. To save you from coding boilerplate classes Moxy uses Annotation Processing. For the correct work of code generation, you must use the special annotations provided by Moxy. More information about the library you can find [here](https://medium.com/redmadrobot-mobile/android-without-lifecycle-mpvsv-approach-with-moxy-6a3ae33521e).

#### Model

Model contains the business-logic and code for managing data. And because we use Clean Architecture + MVP, as Model will act Data and Domain layers.

#### View

View is responsible for how the data will be shown to the user. In the case of Android, View must be implemented by an Activity or Fragment. Also, View informs the Presenter of user interaction, such as button click or text input. There is example of View:

```java
public interface ArticlesListView extends MvpView {

    void showLoadingProgress(boolean show);
    void showVisits(List<Article> articles);
    void showArticlesLoadingErrorMessage();

}
```

So far we have described View interface, i. e. which View methods the Presenter can call. Note that our View is inherited from the **MvpView**, which is part of the Moxy library. That is a must for correct library work.

#### Presenter

According to the MVP concept, View can not directly interact with the Model, so the bridge between them is Presenter. Presenter reacts to user actions that View has notified to it (such as pressing a button, list item click or text input), and then decides what to do next. For example, it could be a data request from the model and display them in the View. Presenter example:

```java
@InjectViewState
public class ArticlesListPresenter extends MvpPresenter<ArticlesListView> {

    private ArticlesListInteractor articlesListInteractor;
    private SchedulersProvider schedulersProvider;

    @Inject
    public ArticlesListPresenter(ArticlesListInteractor articlesListInteractor,
                           SchedulersProvider schedulersProvider) {
        this.articlesListInteractor = articlesListInteractor;
        this.schedulersProvider = schedulersProvider;
      
        loadArticles();
    }
  
    private void loadArticles() {
      	getViewState().showLoadingProgress(true);
        articlesListInteractor.getArticles()
            .observeOn(schedulersProvider.ui())
            .subscribe(articles -> {
                getViewState().showLoadingProgress(false);
                getViewState().showArticles(articles);
            },
            throwable -> getViewState().showLoadingError());
    }
  
    public void onArticleSelected(Article article) {
        ...
    }  

}
```

All the necessary classes we pass through the Presenter's constructor. It's actually called "constructor injection".

When creating the Presenter's object we must pass dependencies required by the constructor. If there are a lot of them, the creation of Presenter will be rather difficult. Because we don't want to do this manually, we will delegate this work to the Component.

```java
@Presenter
@Component(dependencies = ApplicationComponent.class)
public interface ArticlesListComponent {

    ArticlesListPresenter getPresenter();

}
```

It will supply the necessary dependencies, and we need only get the Presenter instance by calling the **getPresenter()** method. If you have a question, "How do you pass arguments to Presenter then?", then look in the FAQ - there is a detailed description of this issue.

Sometimes you can find the code in which the DI-container (Component) is passed to the constructor, after which all the necessary dependencies inject in the fields:

```java
@Inject
ArticlesListInteractor articlesListInteractor;

public VisitsPresenter(ArticlesListPresenterComponent component) {
	component.inject(this);
}
```

However, this approach is incorrect, because it complicates the testing of the class and creates a bunch of unnecessary code. If in the first case we could just pass mocked classes through the constructor, then now we need to create a DI-container and pass it. Also, this approach makes the class dependent on a particular DI-framework, which is also not good.

Also, note that before we display the results from Interactor, we switch the UI thread via `observeOn (schedulersProvider.ui())`, because we don't know in advance in what thread we receive the data

#### Binding View with Presenter

In the context of Android developing, the Activity (or Fragment) acts as a View, so after creating the View interface, we need to implement it in our Activity or Fragment:

```java
public class ArticlesListActivity extends MvpAppCompatActivity implements ArticlesListView {

  @InjectPresenter
  ArticlesListPresenter presenter;
  
  @ProvidePresenter
  ArticlesListPresenter provideArticlesListPresenter() {
      ArticlesListPresenterComponent component = DaggerArticlesListPresenterComponent.builder()
                .applicationComponent(MyApplication.getComponent())
                .build();
    return component.getPresenter;
  }

  @Override
  protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_articles_list);

  }
    
  public void showArticles(List<Article> articles) {
    ...
  }

  public void showLoadingError() {
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

First of all, we divided the code into layers: data, domain, presentation. Also we created a separate package for the DI code. The packages **domain**, **presentation** and **di** are divided by features, and the **data** package is divided by data source type (network, database, file system). This is because all features use the same classes (for example, **NewsApiService**) and it will be very difficult to divide them into features.

Packages named **global** contain common classes that are used in several features. For example, the **data/global** package stores models and repository interfaces.

The **presentation** layer is divided into two packages - **mvp** and **ui**. In the **mvp** are stored Presenter and View classes, as the name implies. In **ui** is stored the implementation of the View layer from MVP, i.e. Activity, Fragments, etc.

This structure has the following benefits:

- **Understandability.** Any developer can tell what functions there are in the application without looking in the code.
- **Easier to add new feature**. If you want to add a new feature to the application, for example, viewing the user profile, then you just need to add the **userprofile** package and work only with it, rather than going through entire package structure for creating the necessary classes.
- **Easier to edit a feature.** When you edit a feature, you need to keep at most two or three packages open and you see only those classes that relate to current feature. When you divide classes by the type, you have to keep open almost the whole tree of packages and you have to see the classes that you do not need now, related to other features.
- **Scalability and easier code navigation**. With the increasing number of app's functions, the number of classes also increases. When you divide classes by type, adding new classes makes navigation among them very uncomfortable, since you need to search for the necessary class, among dozens of others, which affects the speed of development. Dividing by features solves this problem, because you can combine related packages (for example, you can combine **login** and **registration** packages into the **authentication** package).

I'd like to say a few words about package naming: should package names be singular or plural? I hold the approach described [here](https://softwareengineering.stackexchange.com/a/75929):

1) Use singular for packages with heterogeneous classes. For example, a package, that contains classes like **Dog**, **Cat** and **Cow** will be called **animals**. Another example is different implementations of the same interface (**XmlResponseAdapter**, **JsonResponseAdapter**).
2) Use the plural for packages with homogeneous classes. For example, **order** package, that contains **OrderInfo**, **OrderInteractor**, **OrderValidation**, etc.

### Additional entities used in practice

#### Router

Since Presenter decides what happens when you interact with the view, it also know what screen will be opened. But we can't open Activity from Presenter directly. There are two ways of solving this problem - call View method like **openProfileScreen()** and open new screen from Activity or use Router

**Router** is a special class for navigating between screens (Activities or Fragments).

We recommend to use [Alligator](https://github.com/aartikov/Alligator) library for implementing navigation.

#### Mapper

**Mapper** is special class for converting models between layers, for example, from DB model to Domain model. Usually they calles like XxxMapper and have single method with name map (or convert/transform), for example:

```java
public class ArticleDbModelMapper {

  public Article map(ArticleDbModel model) {
    return new Article(model.getName(), model.getLastname, model.getAge());
  }
  
  public List<Article> map(Collection<ArticleDbModel> models) {
        final List<Article> result = new ArrayList<>(models.size());
        for (ArticleDbModel model : models) {
            result.add(map(model));
        }
        return result;
    }

}
```

Since domain layer doesn't know anything about classes from other layers, the mapping of models must be performed in the outer layers, i. e. in Repository (when mapping **data > domain** or **domain> data**) or in Presenter (when mapping **domain> presentation** and vice versa) .

#### ResourceManager

I some cases we need to get a string or a number from resources and use it in Presenter or domain layer. But we know that we can't use Context class. To resolve this problem we must use special class that called ResourceManager. Let's create an interface for this one:

```java
public interface ResourceManager {

    String getString(int resourceId);

    int getInteger(int resourceId);

}
```

This interface must be contained in domain layer. Then we create the interface implementation in presentation layer:

```java
public class AndroidResourceManager implements ResourceManager {
  
    private Context context;

    @Inject
    public AndroidResourceManager(Context context) {
        this.context = context;
    }
  
    @Override	
    public String getString(int resourceId)  {
        return context.getResources().getString(resourceId);
    }

    @Override	
    public int getInteger(int resourceId) {
        return context.getResources().getInteger(resourceId);
    }

}
```

After that we must bind the interface with it's implementation in ApplicationModule:

```java
@Singleton
@Provides
protected ResourceManager provideResourceManager(AndroidResourceManager resourceManager) {
    return resourceManager
}
```

Now we can use ResourceManager in our Presenters or Interactors:

```java
@InjectViewState
public class ArticlesListPresenter extends MvpPresenter<ArticlesListView> {

    ...
    private ResourceManager resourceManager;
    
    @Inject
    public ArticlesListPresenter(..., AndroidResourceManager resourceManager) {
        ...
        this.resourceManager = resourceManager;
    }
    
    private void onLoadError(Throwable throwable) {
         ...
         getViewState().showMessage(resourceManager.getString(R.string.articles_load_error));
    }
    
}
```

Perhaps you have a question: "Why we can use **R** class in the Presenter?" Because it uses Android Framework. Actually, that's not entirely true. **R** class not use any class at all. So there is nothing bad with using **R** class in Presenter.

#### SchedulersProvider

To test our code we need make all operations synchronous. For that, we must replace all Schefulers to **TestScheduler**. For this reason, we set Schedulers through **SchedulersProvider**, but not directly.

```java
public class SchedulersProvider {

    @Inject
    public SchedulersProvider() {
    }

    public Scheduler ui() {
        return AndroidSchedulers.mainThread();
    }

    public Scheduler computation() {
        return Schedulers.computation();
    }

    public Scheduler io() {
        return Schedulers.io();
    }

    public Scheduler newThread() {
        return Schedulers.newThread();
    }

    public Scheduler trampoline() {
        return Schedulers.trampoline();
    }

}
```

It allows easily replace Schedules by extending **SchedulersProvider** and overriding it's methods:

```java
public class TestSchedulersProvider extends SchedulersProvider {

    private final TestScheduler testScheduler = new TestScheduler();

    @Override
    public Scheduler ui() {
        return testScheduler;
    }

    @Override
    public Scheduler computation() {
        return testScheduler;
    }

    @Override
    public Scheduler io() {
        return testScheduler;
    }

    @Override
    public Scheduler newThread() {
        return testScheduler;
    }

    @Override
    public Scheduler trampoline() {
        return testScheduler;
    }

    public TestScheduler testScheduler() {
        return testScheduler;
    }

}
```

Then, when testing we just need use TestSchedulersProvider instead of SchedulersProvider. More info about testing code with RxJava you can find [here](https://github.com/Froussios/Intro-To-RxJava/blob/master/Part%204%20-%20Concurrency/2.%20Testing%20Rx.md).

## Errors handling

[this chapter is in preparation]

## Testing

One key advantage of Clean Architecture is that we can cover with tests much more functionality of the application through the code into small classes, each of which performs a strictly defined task. Thanks to the Inversion of Control principle, that used in Clean Architecture, we can easily replace the implementation classes with fake ones that implement the behavior we want.

Before we start writing tests, we must answer ourselves to two questions:

- What do we want to test?
- How will we test this?

What we want to test:

- We want to test our business logic regardless of any framework or library.
- We want to test our integration with the API.
- We want to test integration with our persistence system.
- Everything that contains conditions.

What we should NOT test:

- Third-party libraries (we assume that they work correctly, because they have already been tested by the developers)
- Trivial code (for example, getters and setters)

Let's take a closer look at how we will test each of layers.

### Presentation layer testing

This layer include 2 types of tests: Unit-tests и UI-tests.

- Unit-tests are used for testing Presenters.
- UI-tests are used for testing Activities (to check if UI works correctly)

There are different naming conventions for unit tests. For example, this](https://dzone.com/articles/7-popular-unit-test-naming) article describes some of them. In my examples of tests I will not adhere to any agreement. The most important thing is to understand what the tests are doing and what we want to get as a result.

Let us take the example of test for **ArticlesListPresenter**:

```java
public class ArticlesListPresenterTest {

    @Test
    public void shouldLoadArticlesOnViewAttached() {
        //preparing
        ArticlesListInteractor interactor = Mockito.mock(ArticlesListInteractor.class);
        TestSchedulersProvider schedulers = new TestSchedulersProvider();
        ArticlesListPresenter presenter = new ArticlesListPresenter(interactor, schedulers);
        ArticlesListView view = Mockito.mock(ArticlesListView.class);
      
        ArrayList<Article> articlesList = ArrayList<Article>;
        when(interactor.getArticlesList()).thenReturn(Single.just(articlesList));
      
      	//testing
        presenter.attachView(view)

        //asserting
        verify(view, times(1)).showLoadingProgress(true);
        verify(view, times(1)).showLoadingProgress(false);
        verify(view, times(1)).showArticles(articlesList);
    }
  
}
```

As you can see, we divided the test code into three parts:

- Preparation for testing. Here we initialize the objects for testing, prepare the test data, and also define the behavior of the mocks.
- Testing itself
- Checking the test results. Here we check that View methods have been called with necessary arguments .

### Domain layer testing

[this chapter is in preparation]

### Data layer testing

[this chapter is in preparation]

## Start a new application development using Clean Architecture

[this chapter is in preparation]

## Migration project to the Clean Architecture

[this chapter is in preparation]

## Clean Architecture FAQ

#### Should I rewrite all project when I migrate to Clean Architecture

There is no definite answer on this question. If your project is big enough and migration to Clean Architecture take a lot of time, the best bet is to rewrite the project gradually, using approach that we have described above. But you can try to rewrite the project from scratch if your project is small (contains 2-3 screens) and you have enough time.

Certainly a cautionary tale about developers of Netscape, who decided to rewrite the code from scratch - [Things You Should Never Do, Part I](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i).

#### Is it necessary to create separate models for each layer (Domain, Data, Presentaion)?

According to the principles of Clean Architecture, Domain layer should know nothing about outer layers (Data and Presentation), but outer layer can use classes from Domain layer. Consequently, yoг don't have to create separate models for each layer. However, if models for each layer are different, you must to create different classes. If you need to add annotations from libraries (for example, from Gson or Room) to models (without changing model structure), then you can add them directly to Domain-layer models, even though these are external libraries of Data layer, because creating separate models will be unnecessary duplication of code.

#### Do we need to create interfaces for Presenters and Interactors to improve code testability?

This is an example of Presenter with interface:

```java
public interface LoginPresenter {

  void onLoginButtonPressed(String email, String password);
}

public class LoginPresenterImpl implements LoginPresenter {
  ...
}
```

No, you don't need to create interface for Presenters or Interactors, because it creates additional problems and has no advantages. There are some of the problems created by using redundant interfaces:

- If we want to add new method or change existing one, we need to change the interface. Also we need to change implementation of the interface. It takes quite some time, even using such a powerful IDE as Android Studio.
- Using of additional interfaces makes code navigation more difficult. For example, if you want to open an implementation of a Presenter method from Activity, then you go to a method definition in the interface.
- The interface doesn't improve code testability. You can easily replace Presenter's implementation to it's mock, using any mocking library.

You can read more about this topic in the following articles:

- [Interfaces for presenters in MVP are a waste of time!](http://blog.karumi.com/interfaces-for-presenters-in-mvp-are-a-waste-of-time)

#### How to pass arguments into Presenter, when it's instance created via DI-container?

Often when creating a presenter, we want to pass arguments to constructor For example, we want to pass an article ID to get its content from the server. To do this, we need to create a separate module for Presenter and pass the arguments there:

```java
@Module
public class ArticleDetailsModule {

    private final long articleId;

    public ArticleDetailsModule(long articleId) {
        this.articleId = articleId;
    }

    @Provides
    @Presenter
    long provideArticleId() {
        return articleId;
    }

}
```

Next we need to add this module to the Component:

```java
@Component(dependencies = ApplicationComponent.class, modules = ArticleDetailsModule.class)
public interface ArticleDetailsComponent {
```

When creating the Component we provide our module with arguments:

```java
long articleId = ...

ArticleDetailsComponent component = DaggerArticleDetailsComponent.builder()
    .applicationComponent(MyApplication.getComponent())
    .articleDetailsModule(new ArticleDetailsModule(articleId))
    .build();
```

Now we can get our identifier through constructor:

```java
@Inject
public UserFollowersPresenter(ArticleDetailsInteractor interactor, long articleId) {
    this.interactor = interactor;
    this.articleId = articleId;
}
```

Let's suppose that in addition an article ID, we want to pass an user ID, that also has type "long". If we try to create another provide-method in our module, Dagger will give an error message, because it doesn't know which method provides an user ID or an article ID.

To fix this, we need to create **Qualifier** annotations that will tell Dagger "who is who":

```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface ArticleId {

}

@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface UserId {

}
```

Add annotations to our provide-methods:

```java
@ArticleId
@Provides
@Presenter
long provideArticleId() {
    return articleId;
}

@UserId
@Provides
@Presenter
long provideUserId() {
    return userId;
}
```

Also, we need to mark the constructor arguments with annotations:

```java
@Inject
public UserFollowersPresenter(ArticleDetailsInteractor interactor, @ArticleId long articleId, @UserId long userId) 
```

Done. Now Dagger can pass the arguments correctly.
