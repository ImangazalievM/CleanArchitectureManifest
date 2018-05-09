# CleanArchitectureManifest (v 0.9.4)

Here you will find description of main principles and rules, that are worth following in developing Android apps using Clean Architecture approach.

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

According to this principle, domain layer must not depends on outer ones. That is, classes from outer layers must not be used in the domain layer. Interaction with outer layers is implemented through interfaces. Declaration of the interfaces contains in domain layer and implementation contains in outer ones.

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

Above presents layers interaction scheme. Those black arrows represent dependencies of some layers on others, and blue arrows represent data flow. As you can see, data and presentation layers depend on domain layer, i. e. they use his classes. But **domain** layer doesn't know anything about outer layers and uses only its own classes and interfaces. Next, we will explain each of those layers in more detail, and how they interact with each other.

As can be seen from the scheme, all three layers can exchange data. It is worth mentioning that direct interaction between **presentation** и **data** layers must not be allowed. Data flow should go from **presentation** layer to **data** layer through **domain** (this could be, for example, passing string with search query or user registration data). The same can happen and vice versa (for example, when we return search results).

## Domain layer

![DomainLayer](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/DomainLayer.png)

**Business logic** is rules, that describe how a business works (for example, user can't make a purchase for more than there is on his account). Business logic does not depend on the implementation of the database or UI. Business logic changes only when business requirements change.

Robert Martin divides business logic into two types: a specific for a concrete application and common for all apps (if you want to share your code between platforms).

**Entity** - contain application independent business rules. 

**Interactor** - an object that implements business logic for a specific application.

But it's all in theory. In practice, only Interactors are used. At least I have not seen any applications that use Entity. By the way, many confuse Entity with DTO (Data Transfer Object). The fact is that the Entity of Clean Architecture is not exactly the Entity that we are used to seeing.

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

1. You do not need to create listeners to get results.
2. It's easy to switch threads.
3. It's easy to handle  errors.


To switch the between threads, we use the` subscribeOn` method, as usual, but we get the Scheduler not through the static methods of the Schedulers class, but with the [SchedulersProvider'а](#schedulersprovider). It will help us in the future, when we want to test our code.

### Data layer

![DataLayer](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/DataLayer.png)

This layer contains everything about storing and managing data. It could be database, SharedPreferences, network or file system, as well as caching logic.

As a "bridge" between data and domain layer, there is Repository interface (in the original Uncle Bob's scheme it's called Gateway). The interface itself is stored in the Domain layer, but his implementation is stored in the Data layer. In doing so, domain layer classes don't know where the data comes from - from the database, the network or from somewhere else. That's why all caching logic should be contained in the data layer.

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

According to the MVP concept, View can not directly interact with the Model, so the bridge between them is Presenter. Presenter reacts to user actions that View has notified to it (such as pressing a button, list item or text input), and then decides what to do next. For example, it could be a data request from the model and display them in the View. Presenter example:

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

Since Presenter has arguments we must to provide Presenter instance via ```provideArticlesListPresenter```, that we annotated with ```@ProvidePresenter```. Note that all annotated fields and methods must have public or package-private visibility.

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

Прежде чем делить код по фичам, мы разделили его на слои. Данный подход позволяет сразу определить к какому слою относится тот или иной класс. Если вы заметили, классы слоя **data** разбиты немного не так, как в слоях **domain**, **presentation** и **di**. Здесь вместо фич приложения мы выделили типы источников данных - сеть, база данных, файловая система. Это связано с тем, что все фичи используют практически одни и те же классы (например, **NewsApiService**) и их не имеет смысла разбивать по фичам.

В пакетах с именем **global** хранятся общие классы, которые используются в нескольких фичах. Например, в пакете **data/global** хрянятся модели и интерфейсы репозиториев. 

Слой **presentation**  разбит на два пакета - **mvp** и **ui**. В **mvp** хранятся, как понятно из названия, классы Presenter'ов и View. В **ui** хранятся реализация слоя View из MVP, т. е. Activity, Fragment'ы и т. д. 

Разбиение классов по фичам имеет ряд преимуществ:

- **Очевидность:** Даже не знакомый с проектом разработчик, при первом взгляде на структуру пакетов сможет примерно понять что делает приложение, не заглядывая в сам код. 
- **Добавление нового функционала**.  Если вы решили добавить новую функцию в приложение, например, просмотр профиля пользователя, то вам лишь нужно добавить пакет **userprofle** и работать только с ним, а не "гулять" по всей структуре пакетов, создавая нужные классы.
- **Удобство редактирования.** При редактировании какой либо фичи, нужно держать открытыми максимум два-три пакета и вы видите только те классы, которые относятся к конкретной фиче. При разбиении по типу класса, раскрытой приходится держать практически всё дерево пакетов и вы видите классы, которые вам сейчас не нужны, относящиеся к другим фичам.
- **Удобство масштабирования**. При увеличении количества функций приложения, увеличивается и количество классов. При разбиении классов по типу, добавление новых классов делает навигацию по ним очень не удобным, т.к. приходится искать нужный класс, среди десятков других, что сказывается на скорости и удосбстве разработки. Разбиение по фичам решает эту проблему, т.к. вы можете объединить связанные между собой пакеты с фичами (напрмер, можно объединить пакеты **login** и **registration** в пакет **authentication**).

Также хочется сказать пару слов об именовании пакетов: в каком числе их нужно называть - множественном или единственном? Я придерживаюсь подхода, описанного [здесь](https://softwareengineering.stackexchange.com/a/75929):

1) Если пакет содержит однородные классы, то имя пакета ставится во множественном числе. Например, пакет с классами **Dog**, **Cat** и **Cow** будет называться **animals**. Другой пример - различные реализации какого-либо интерфейса (**XmlResponseAdapter**, **JsonResponseAdapter**).
2) Если пакет содержит разнородные классы, реализующую определенную функцию, то имя пакета ставится в единственном числе. Пример - пакет **order**, содержащий классы **OrderInfo**,  **OrderInteractor**, **OrderValidation** и т. д.

### Additional entities used in practice

#### Router

Т. к. Presenter содержит в себе логику реагирования на действия пользователя, то он также знает о том, на какой экран нужно перейти. Однако сам Presenter не может осуществлять переход на новый экран, т. к. для этого нам требуется Context. Поэтому за открытие нового экрана должна отвечать View. Для осуществления перехода на следующий экран мы должны вызвать метод View, например, **openProfileScreen()**, а уже в реализации самого метода осуществлять переход. Помимо данного подхода некоторые разработчики используют для навигации так называемый Router.

**Router** - класс, для осуществления переходов между экранами (активити или фрагментами).

Для реализации Router'а вы можете использовать библиотеку [Alligator](https://github.com/aartikov/Alligator).

#### Mapper

**Mapper** - специальный класс, для конвертирования моделей из одного типа в другой, например, из модели БД в модель бизнес-логики. Обычно они имеют название типа XxxMapper, и имеют единственный метод с названием map (иногда встречаются названия convert/transform), например:

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

Т. к. слой **domain** ничего не знает о классах других слоев, то маппинг моделей должен выполняться во внешних слоях, т. е. репозиторием (при конвертации **data** > **domain** или **domain** > **data**) или презентером (при конвертации **domain** > **presentation** и наоборот) .

#### ResourceManager

В некоторых случаях может потребоваться получить строку или число из ресурсов приложения в Presenter'е или слое **domain** . Однако, мы знаем, что они не должны напрямую взаимодействовать с фреймворком Android. Чтобы решить эту проблему мы можем создать специальную сущность ResourceManager, для доступа у внешним ресурсам. Для этого мы создаем интерфейс:

```java
public interface ResourceManager {

    String getString(int resourceId);

    int getInteger(int resourceId);

}
```

Сам интерфейс должен располагаться в слое **domain**. После этого в слое **presentation** мы создаем реализацию нашего интерфейса:

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

Далее мы должны связать интерфейс и реализацию нашего ResourceManager'а в ApplicationModule:

```java
@Singleton
@Provides
protected ResourceManager provideResourceManager(AndroidResourceManager resourceManager) {
    return resourceManager
}
```

Теперь мы можем использовать ResourceManager в Presenter'е или Interactor'ах:

```java
@InjectViewState
public class ArticlesListPresenter extends MvpPresenter<ArticlesListView> {

    ...
    private ResourceManager resourceManager;
    
    @Inject
    public ArticlesListPresenter(...,  AndroidResourceManager resourceManager) {
        ...
        this.resourceManager = resourceManager;
    }
    
    private void onLoadError(Throwable throwable) {
         ...
         getViewState().showMessage(resourceManager.getString(R.string.articles_load_error));
    }
    
}
```

Наверное, у внимательных читателей возник вопрос: почему мы используем класс **R** в Presenter'е? Ведь он также относится к Android? На самом деле, это не совсем так. Класс **R** вообще не использует никакие классы, и представляет из себя набор идентификаторов ресурсов. Поэтому, нет ничего плохого, чтобы использовать его в Presenter'е.

#### SchedulersProvider

Перед началом тестирования нам нужно сделать все операции синхронными. Для этого мы должны заменить все Scheduler'ы на **TestScheduler**, поэтому мы не устанавливаем Scheduler'ы напрямую через класс **Schedulers**, используем **SchedulersProvider**:

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

Благодаря этому мы можем легко заменить Scheduler'ы на нужные нам, всего лишь создав наследника класса SchedulersProvider'а и переопределив методы:

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

Далее, при самом тестировании, нам нужно будет лишь использовать TestSchedulersProvider вместо SchedulersProvider. Более подробно о тестировании кода с RxJava можно почитать [здесь](https://github.com/Froussios/Intro-To-RxJava/blob/master/Part%204%20-%20Concurrency/2.%20Testing%20Rx.md).

## Errors handling

[this chapter is in preparation]

## Testing

Одним из самых главных преимуществ является то, что мы можем покрыть тестами намного больший функционал приложения, за счет разбиения кода на мелкие классы, каждый из которых выполняет строго определенную задачу. Благодаря принципу инверсии зависимостей, используемому в чистой архитектуре мы можем с легкостью подменять реализацию тех или иных классов на фейковые, которые реализуют нужное нам поведение.

Прежде чем начать писать тесты, мы должны ответить себе на два вопроса:

- что мы хотим тестировать?
- как мы будем это тестировать?

Что мы хотим тестировать:

- Мы хотим проверить нашу бизнес-логику независимо от какого-либо фреймворка или библиотеки.
- Мы хотим протестировать нашу интеграцию с API.
- Мы хотим протестировать нашу интеграцию с нашей системой персистентности.
- Мы хотим протестировать некоторые общие компоненты пользовательского интерфейса.

Что мы НЕ должны тестировать:

- Сторонние библиотеки (мы предполагаем, что они работают правильно, потому что уже протестированы разработчиками)
- Тривиальный код (например, геттеры и сеттеры)

Теперь, давайте разберём то, как мы будем тестировать каждый из слоев.

### Presentation layer testing

Данный слой включает в себя 2 типа тестов:  Unit-тесты и UI-тесты.

- Unit-тесты используются для тестирования Presenter'ов.
- UI-тесты используются для тестирования Activity (проверяется корректность отображения элементов и т. д.).

Существуют различные соглашения по именованию тестовых методов. Например, в этой [статье](https://dzone.com/articles/7-popular-unit-test-naming) описаны некоторые из них. В примерах, которые я буду приводить далее, я не буду придерживаться какого-либо соглашения. В общем, нет большой разницы, как их называть. Самое главное понять из названия, что тестирует наш метод и что мы хотим получить в результате. 

Давайте рассмотрим пример теста для **ArticlesListPresenter**:

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

Как видите, мы разделили код теста на три части:

- Подготовка к тестированию. Здесь мы инициализируем объекты для тестирования, подготавливаем тестовые данные, а также предопределяем поведение моков.
- Само тестирование. 
- Проверка результатов тестирования. Здесь мы проверяем, что у View были вызваны нужные методы и переданы аргументы.

### Domain layer testing

В данном слое тестируюится классы Interactor'ов и Entity. Необходимо проверить, действительно ли бизнес-логика реализует требуемое поведение .

[this chapter is in preparation]

### Data layer testing

[this chapter is in preparation]

## Start a new application development using Clean Architecture

Если вы начиначете разработку нового мобильного приложения, то лучше начать с создания пользовательского интерфейса, т. к. именно UI определяет 

[this chapter is in preparation]

## Migration project to Clean Architecture

[this chapter is in preparation]

#### Step 1: 

#### Step 2: 

#### Step 3:

## Clean Architecture FAQ

#### Стоит ли переписывать весь проект при переносе проекта на Clean Architecture?

Наверное, нет однозначного ответа на этот вопрос. Если проект большой и переход на Clean Architecture может длительный промежуток времени, то лучше переписывать код постепенно, используя подход, который мы описали выше. Если же проект простой и состоит из 2-3 экранов, а сроки не поджимают, то вы можете попробовать переписать проект с нуля.

В конце хочется привести поучительную историю про Netscape, который переписывали с нуля больше, чем три года - [Things You Should Never Do, Part I](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/)

#### Обязательно ли создавать отдельные сущности для каждого из слоев (Domain, Data, Presentaion)?

Согласно принципам Clean Architecture, слой Domain ничего не должен знать о внешних слоях (Data и Presentation), но внешние слои без проблем могут использовать классы из слоя Domain. Следовательно, можно не создавать отдельные сущности для каждого из слоев, а использовать только те, что лежат в слое Domain. Однако, если их формат не совпадает с тем, что используется во внешних слоях, то нужно создать отдельную сущность. Также не следует использовать в моделях слоя Domain аннотации, которые требуются библиотекам, типа [Gson](https://github.com/google/gson) или [Room](https://developer.android.com/topic/libraries/architecture/room.html). В этом случае нужно создать отдельную сущность в слое Data.

[this chapter is in preparation]

#### Нужно ли создавать интерфейсы для классов Presenter и Interactor для улучшения тестируемости кода?

Пример Presenter'а с интерфейсом:

```java
public interface LoginPresenter {

  void onLoginButtonPressed(String email, String password);
}

public class LoginPresenterImpl implements LoginPresenter {  
  ...
}
```

Нет, интерфейсы для презентера и интерактора создавать не нужно. Это создает дополнительные сложности при разработке, при этом пользы от данного подхода практически нет. Вот лишь некоторые проблемы, которые порождает создаение лишних интерфейсов:

- Если мы хотим добавить новый метод или изменить существующий, нам нужно изменить интерфейс. Помимо этого мы также должны изменить реализацию метода. Это занимает довольно времени, даже при использовании такой продвинутой IDE как Android Studio.
- Использование дополнительных интерфейсов усложняет навигацию по коду. Если вы хотите перейти к реализации метода Presenter'а из Activity (т. е. реализации View), то вы переходите к интерфейсу Presenter'а.
- Интерфейс никак не улучшает тестируемость кода. Вы с легкостью можете заменить класс Presenter'а на mock, используя любую библиотеку для mock'ирования.

Более подробно можете почитать об этом в следующих статьях:

- [Interfaces for presenters in MVP are a waste of time!](http://blog.karumi.com/interfaces-for-presenters-in-mvp-are-a-waste-of-time)
- [Франкен-код или франкен-дизайн](https://plus.google.com/u/0/+SergeyTeplyakov/posts/gRaqrqaiGbe)
- [Управление зависимостями](http://sergeyteplyakov.blogspot.ru/2012/11/blog-post.html)

#### Как передать аргументы в Presenter, если его инстанс создает DI-контейнер?

Часто при создании презентера возникает необходимость передать дополнительные аргументы. Например, мы хотим передать идентфикатор статьи, чтобы получить её содержимое от сервера. Чтобы сделать это, нам необходимо создать отдельный модуль для Presenter'а и передать аргументы туда:

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

Далее нам нужно добавить наш модуль в Component:

```java
@Component(dependencies = ApplicationComponent.class, modules = ArticleDetailsModule.class)
public interface ArticleDetailsComponent {
```

При создании Component'а мы должны передать наш модуль с идентификатором:

```java
long articleId = ...

ArticleDetailsComponent component = DaggerArticleDetailsComponent.builder()
    .applicationComponent(MyApplication.getComponent())
    .articleDetailsModule(new ArticleDetailsModule(articleId))
    .build();
```

Теперь мы можем получить наш идентификатор через конструктор:

```java
@Inject
public UserFollowersPresenter(ArticleDetailsInteractor interactor, long articleId) {
    this.interactor = interactor;
    this.articleId = articleId;
}
```

Теперь представим, что помимо идентфикатора статьи, мы хотим передать ID пользователя, который так же имеет тип long. Если мы попытаемся создать ещё один provide-метод в нашем модуле, Dagger выдаст ошибку, о том, что типы совпадают и он не знает какой из них является идентфикатором статьи, а какой идентфикатором пользователя. 

Чтобы исправить это, нам необходимо создать  Qualifier-аннотации,  которые будут указывать Dagger'у "who is who":

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

Добавляем аннотации к нашим provide-методам:

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

Также нужно пометить аннотациями аргументы конструктора:

```java
@Inject
public UserFollowersPresenter(ArticleDetailsInteractor interactor, @ArticleId long articleId, @UserId long userId) 
```

Готово. Теперь Dagger сможет верно расставить аргументы в конструктор.
