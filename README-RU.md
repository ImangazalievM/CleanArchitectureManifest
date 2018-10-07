![CleanArchitectureManifest](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/CleanArchitectureManifest.png)

# Clean Architecture Manifest (v. 0.9.5)

Здесь вы найдете описание основных принципов и правил, которыми стоит руководствоваться при разработке Android-приложений с использованием чистой архитектуры. 

**Translations:**

[English](/README.md) | [Русский](/README-RU.md)

Если вы хотите перевести этот документ на свой язык, пожалуйста посетите [эту страницу](contributing.md).

## Содержание

- [Введение](#Введение)
- [Слои и инверсия зависимостей](#Слои-и-инверсия-зависимостей)
  - [Слой бизнес-логики (Domain)](#Слой-бизнес-логики-domain)
  - [Слой работы с данными (Data)](#Слой-работы-с-данными-data)
    - [Repository](#repository)
  - [Слой отображения (Presentation)](#Слой-отображения-presentation)
    - [Model](#model)
    - [View](#view)
    - [Presenter](#presenter)
    - [Связывание View с Presenter'ом](#Связывание-view-c-presenterом)
- [Разбиение классов по пакетам](#Разбиение-классов-по-пакетам)
- [Дополнительные сущности, используемые на практике](#Дополнительные-сущности-используемые-на-практике)
  - [Router](#router)
  - [Mapper](#mapper)
  - [ResourceManager](#resourcemanager)
  - [SchedulersProvider](#schedulersprovider)
- [Обработка ошибок](#Обработка-ошибок)
- [Тестирование](#Тестирование)
- [Перенос на Clean Architecture существующих проектов](#Перенос-на-clean-architecture-существующих-проектов)
- [FAQ по Clean Architecture](#faq-по-clean-architecture)

## Введение

Clean Achitecture — способ построения архитектуры приложения, [предложенный](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html) Робертом Мартином (который также известен как дядюшка Боб - Uncle Bob) в 2012 году. 

Clean Architecture включает в себя два основных принципа:

1. **Разделение на слои** 
2. **Инверсия зависимостей**

Давайте расшифруем каждый из них.

**Разделение на слои**

Суть принципа заключается в разделении всего кода приложения на слои. Всего мы имеем три слоя: 

- слой отображения
- слой бизнес логики
- слой работы с данными

Самым главным слоем является слой бизнес логики. Особенность данного слоя заключается в том, что он не зависит ни от каких внешних библиотек или фреймворков. Это достигается за счет *инверсии зависимостей*.

**Инверсия зависимостей**

Согласно данному принципу слой бизнес-логики не должен зависеть от внешних. То есть классы из внешних слоев не должны использоваться в классах бизнес-логики. Взаимодействие с внешними слоями происходит через интерфейсы, которые реализуют классы внешних слоев. 

Благодаря разделению ответственности между классами мы легко можем изменять код приложения, а также добавлять новый функционал, затрагивая при этом минимальное количество классов. Помимо этого мы получаем легко тестируемый код. Стоит заметить, что построение правильной архитектуры целиком и полностью зависит от самого разработчика и его опыта.

Преимущества чистой архитектуры:

- независимость от UI, БД и фреймворков
- позволяет быстрее добавлять новые функции
- более высокий процент покрытия кода тестами
- повышенная простота навигации по структуре пакетов

Недостатки чистой архитектуры:

- большое количество классов
- довольно высокий порог вхождения и, зачастую, неправильное понимание на первых порах

**Внимание:** перед прочтением данного документа, настоятельно рекомендую ознакомиться  со следующими темами (иначе, вы ничего не поймете):

- Dagger 2
- RxJava 2

Очень желательно, чтобы у вас был практический опыт их использования, так вы быстрее войдете в курс дела. Если вы уже знакомы с ними, то можете смело приступать к прочтению. Всё объяснение темы Clean Architecture будет строиться вокруг новостного приложения, которое мы, в теории, хотели бы создать.

**P. S.** Я разработал приложение, чтобы продемонстрировать использование чистой архитектуры на практике. Исходный код вы можете найти здесь - [Bubbble](https://github.com/ImangazalievM/Bubbble).

## Слои и инверсия зависимостей

![CleanArchitectureLayers](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/CleanArchitectureLayers.png)

Как уже говорилось ранее, архитектуру приложения, построенную по принципу Clean Architecture можно разделить на три слоя: 

- слой отображения (presentation)
- слой бизнес-логики (domain)
- слой работы с данными (data)

Выше представлена схема того, как эти слои взаимодействуют. Черными стрелками обозначены зависимости одних слоев от других, а синими - поток данных. Как видите, слои **data** и **presentation** зависят от **domain**, т. е. они используют его классы. Сам же слой **domain** ничего не знает о внешних слоях и использует только собственные классы и интерфейсы. Далее мы разберем более подробно каждый из этих слоев, и то, как они взаимодействуют между собой.

Как видно из схемы, все три слоя могут обмениваться данными. Следует отметить, что нельзя допускать прямого взаимодействия между слоями  **presentation** и **data**. Поток данных должен идти от слоя **presentation** к **domain**, а от него к слою **data** (это может быть, например, передача строки с поисковым запросом или регистрационные данные пользователя). То же самое может происходить и в обратном направлении (например, при передаче списка с результатами поиска). 

## Слой бизнес-логики (Domain)

![DomainLayer](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/DomainLayer.png)

**Бизнес-логика** - это правила, описывающие, как работает бизнес (например, пользователь не может совершить покупку на сумму больше, чем есть на его счёте). Бизнес-логика не зависит от реализации базы данных или интерфейса пользователя. Бизнес-логика меняется только тогда, когда меняются требования бизнеса, и не зависит от используемой СУБД или интерфейса пользователя. 

Роберт Мартин разделяет бизнес-логику на два вида: специфичную для конкретного приложения и общую для всех приложений (в том случае, если вы хотите сделать ваш код общим между приложениями под разные платформы). 

**Бизнес объект (Entity)** - хранят бизнес-логику общую для всех приложений. 

**Interactor** – объект, реализующий бизнес-логику специфичную для конкретного приложения.

Но это все в теории. На практике же используются только Interactor'ы. По крайней мере, мне не встречались приложения, использующие Entity. Кстати, многие путают Entity с DTO (Data Transfer Object). Дело в том, что Entity из Clean Architecture - это не совсем те Entity, которые мы привыкли видеть. [Данная](https://habrahabr.ru/company/mobileup/blog/335382/) статья проливает свет на этот вопрос, а также на многие другие.

**Сценарий использования (Use Case)** - набор операций для выполнения какой-либо задачи.  Пример сценария использования при регистрации пользователя:

> 1. Проверяем данные пользователя
> 2. Отправляем данные на сервер для регистрации
> 3. Сообщаем пользователю об успешной регистрации или ошибке
>
> Исключительные ситуации:
>
> 1. Пользователь ввел неверные данные (выдаем ошибку)

Давайте теперь посмотрим как это выглядит на практике. Роберт Мартин предлагает создавать для каждого сценария использования отдельный класс, который имеет один метод для его запуска. Пример такого класса:

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

<details><summary><b>Kotlin version</b></summary><p>

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

</p></details>

Однако практика показывает, что при таком подходе получается огромное количество классов, с малым количеством кода. Более правильным будет создание одного Interactor'а на один экран, методы которого реализуют определенный сценарий, например:

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

<details><summary><b>Kotlin version</b></summary><p>

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

</p></details>

Как видите иногда методы Interactor'а могут и вовсе не содержать бизнес-логики, а методы Interactor'а выступают в качестве прослойки между Repository и Presenter'ом.

Если вы заметили, методы Interactor'а возвращают не просто результат, а классы RxJava 2 (в зависимости от типа операции мы используем разные классы - Single, Completable и т. д.).  Это дает несколько преимуществ:

1. Не нужно создавать слушатели для получения результата.
2. Легко переключать потоки.
3. Легко обрабатывать ошибки.

Для переключения потоков мы используем, как обычно, метод subscribeOn, однако мы получаем Scheduler не через статические методы класса Schedulers, а при помощи [SchedulersProvider'а](#schedulersprovider). В будущем это поможет нам при тестировании.

### Слой работы с данными (Data)

![DataLayer](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/DataLayer.png)



В данном  слое содержится всё, что связано с хранением данных и управлением ими. Это может работа с базой данных, SharedPreferences, сетью или файловой системой, а также логика кеширования, если она имеется.

"Мостом" между слоями data и domain является интерфейс Repository (в оригинальной схеме дядюшки Боба он называется Gateway). Сам интерфейс находится в слое domain, а уже реализация располагается в слое data. При этом классы domain-слоя не знают откуда берутся данные - из БД, сети или откуда-то ещё. Именно поэтому вся логика кеширования должна содержаться в data-слое.

#### Repository

**Repository** - представляет из себя интерфейс, с которым работает Interactor. В нем описывается какие данные хочет получать Interactor от внешних слоев. В приложении может быть несколько репозиториев, в зависимости от задачи. Например, если мы делаем  новостное приложение, репозиторий работающий со статьями может называться ArticleRepository, а репозиторий для работы с комментариями CommentRepository. Пример репозитория, работающего со статьями:

```java
public interface ArticleRepository {

  Single<Article> getArticle(String articleId);

  Single<List<Article>> getLastNews();
  
  Single<List<Article>> getCategoryArticles(String categoryId);
  
  Single<List<Article>> getRelatedPosts(String articleId);
  
}
```

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
interface ArticleRepository {

    fun getArticle(articleId: String): Single<Article>
    
    fun getLastNews(): Single<List<Article>>

    fun getCategoryArticles(categoryId: String): Single<List<Article>>

    fun getRelatedPosts(articleId: String): Single<List<Article>>

}
```

</p></details>

### Слой отображения (Presentation)

![PresentationLayer](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/PresentationLayer.png)

Слой представления содержит все компоненты, которые связаны с UI, такие как View-элементы, Activity, Fragment'ы и т. д. Помимо этого здесь содержатся Presenter'ы и View (или ViewModel'и при использовании MVVM). В данном туториале для реализации слоя presentation будет использован шаблон MVP, но вы можете выбрать любой другой (MVVM, MVI).

Для более удобной связки View и Presenter мы будем использовать библиотеку [Moxy](https://github.com/Arello-Mobile/Moxy). Она помогает решить многие проблемы, связанные с жизненным циклом Activity или Fragment'а. Moxy имеет базовые классы, такие как ```MvpView``` и ```MvpPresenter```, от которых должны наследоваться наши View и Presenter. Для избежания написания большого количества кода по связыванию View и Presenter, Moxy использует кодогенерацию. Для правильной работы кодогенерации мы должны использовать специальные аннотации, которые предоставляет нам Moxy. Более подробную информацию о библиотеке можно найти [здесь](https://habrahabr.ru/post/276189/).

#### Model

MVP расшифровывается как Model-View-Presenter (модель-представление-презентер). Model содержит в себе бизнес-логику и код по работе с данными.  Т. к. мы используем связку Clean Architecture + MVP, то Model у нас является код находящийся в слоях Data (работа с данными) и Domain (бизнес-логика). Следовательно, в слое Presentation остаются лишь два компонента - View и Presenter.

#### View

View отвечает за то, каким образом данные будут показаны пользователю. В случае с Android в качестве View выступает Activity или Fragment. Также View сообщает о действиях пользователя Presenter'у, будь то нажатие на кнопку или ввод текста. Пример View:

```java
public interface ArticlesListView extends MvpView {

    void showLoadingProgress(boolean show);
    void showArticles(List<Article> articles);
    void showArticlesLoadingErrorMessage();

}
```

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
interface ArticlesListView : MvpView {

    fun showLoadingProgress(show: Boolean)
    fun showArticles(articles: List<Article>)
    fun showArticlesLoadingErrorMessage()

}
```

</p></details>

Пока мы описали лишь интерфейс View, т. е. какие команды Presenter может отдавать View. Обратите внимание, что наш интерфейс наследуется от интерфейса **MvpView**, входящего в библиотеку Moxy. Это является обязательным условием для корректной работы библиотеки.

#### Presenter

Согласно концепции MVP, View не может напрямую взаимодействовать с Model, поэтому связующим звеном между ними является Presenter. Presenter реагирует на действия пользователя, о которых ему сообщила View  (такие как нажатие на кнопку, пункт списка или ввод текста), после чего принимает решения о том, что делать дальше. Например, это может быть запрос данных у модели и отображение их во View. Пример Presenter'а:

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

<details><summary><b>Kotlin version</b></summary><p>

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

</p></details>

Все необходимые классы для работы Presenter'а (как и всех остальных классов) мы передаем через конструктор. Этот способ так и называется - внедрение через конструктор.

При создании объекта Presenter'а мы должны передать ему запрашиваемые конструктором зависимости. Если их будет много, то создание Presenter'а будет довольно сложным делом. Чтобы не делать этого вручную, мы доверим это дело Component'у. 

```java
@Presenter
@Component(dependencies = ApplicationComponent.class)
public interface ArticlesListComponent {

    ArticlesListPresenter getPresenter();

}
```

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
@Presenter
@Component(dependencies = ApplicationComponent::class)
interface ArticlesListComponent {

    val getPresenter() : ArticlesListPresenter

}
```

</p></details>

Он подставит нужные зависимости, а нам нужно будет лишь получить инстанс Presenter'а вызвав  метод **getPresenter()**. Если у вас возник вопрос "А как в таком случае передавать  аргументы в Presenter?", то загляните в FAQ - там подробно описан этот вопрос.

Иногда можно встретить такое, что в конструктор передается DI-контейнер (Component), после чего все необходимые зависимости внедряются в поля:

```java
@Inject
ArticlesListInteractor articlesListInteractor;

public VisitsPresenter(ArticlesListPresenterComponent component) {
	component.inject(this);
}

```

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
@Inject
lateinit var articlesListInteractor: ArticlesListInteractor

init {
    component.inject(this)
}
```

</p></details>

Однако, данный способ является неправильным, т. к. усложняет тестирование класса и создает кучу ненужного кода. Если в первом случае мы сразу могли передать mock'и классов через конструктор, то теперь нам нужно создать DI-контейнер и передавать его. Также данный способ делает класс зависимым от конкретного DI-фреймворка, что тоже не есть хорошо.

Также обратите внимание на то, что перед тем как отобразить результаты, полученные от Interactor'а, мы переключаем поток на UI при помощи `observeOn(schedulersProvider.ui())`. Это сделано потому, что мы не знаем заранее в каком потоке нам придут данные.

#### Связывание View с Presenter'ом

В контексте разработки под Android роль View на себя берет Activity (или Fragment), поэтому после создания интерфейса View, мы должны реализовать его в нашей Activity или Fragment'е:

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

<details><summary><b>Kotlin version</b></summary><p>

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

</p></details>

Хочу заметить, что для правильной работы библиотеки Moxy, наша Activity должна обязательно наследоваться от класса **MvpAppCompatActivity** (или **MvpAppCompatFragment** в случае, если вы используете фрагменты). С помощью аннотации ```@InjectPresenter``` мы сообщаем Annotation Processor'у в какую переменную нужно "положить" Presenter. 

Так как конструктор нашего Presenter'а не пустой, а принимает на вход определенные параметры, нам нужно предоставить библиотеке объект Presenter'а. Мы делаем это при помощи метода ```provideArticlesListPresenter```, который мы пометили аннотацией ```@ProvidePresenter```. Как и во всех других случаях использования кодогенерации, переменные и методы, помеченные аннотациями, должны быть видны на уровне пакета, т. е. у них не должно быть модификаторов видимости (private, public, protected).

### Разбиение классов по пакетам

Ниже представлен пример разбиения пакетов по фичам новостного приложения: 

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

В пакетах с именем **global** хранятся общие классы, которые используются в нескольких фичах. Например, в пакете **data/global** хранятся модели и интерфейсы репозиториев. 

Слой **presentation**  разбит на два пакета - **mvp** и **ui**. В **mvp** хранятся, как понятно из названия, классы Presenter'ов и View. В **ui** хранятся реализация слоя View из MVP, т. е. Activity, Fragment'ы и т. д. 

Разбиение классов по фичам имеет ряд преимуществ:

- **Очевидность.** Даже не знакомый с проектом разработчик, при первом взгляде на структуру пакетов сможет примерно понять что делает приложение, не заглядывая в сам код. 
- **Добавление нового функционала**.  Если вы решили добавить новую функцию в приложение, например, просмотр профиля пользователя, то вам лишь нужно добавить пакет **userprofile** и работать только с ним, а не "гулять" по всей структуре пакетов, создавая нужные классы.
- **Удобство редактирования.** При редактировании какой либо фичи, нужно держать открытыми максимум два-три пакета и вы видите только те классы, которые относятся к конкретной фиче. При разбиении по типу класса, раскрытой приходится держать практически всё дерево пакетов и вы видите классы, которые вам сейчас не нужны, относящиеся к другим фичам.
- **Удобство масштабирования**. При увеличении количества функций приложения, увеличивается и количество классов. При разбиении классов по типу, добавление новых классов делает навигацию по ним очень не удобным, т.к. приходится искать нужный класс, среди десятков других, что сказывается на скорости и удосбстве разработки. Разбиение по фичам решает эту проблему, т.к. вы можете объединить связанные между собой пакеты с фичами (например, можно объединить пакеты **login** и **registration** в пакет **authentication**).

Также хочется сказать пару слов об именовании пакетов: в каком числе их нужно называть - множественном или единственном? Я придерживаюсь подхода, описанного [здесь](https://softwareengineering.stackexchange.com/a/75929):

1) Если пакет содержит однородные классы, то имя пакета ставится во множественном числе. Например, пакет с классами **Dog**, **Cat** и **Cow** будет называться **animals**. Другой пример - различные реализации какого-либо интерфейса (**XmlResponseAdapter**, **JsonResponseAdapter**).
2) Если пакет содержит разнородные классы, реализующую определенную функцию, то имя пакета ставится в единственном числе. Пример - пакет **order**, содержащий классы **OrderInfo**,  **OrderInteractor**, **OrderValidation** и т. д.

### Дополнительные сущности, используемые на практике

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

<details><summary><b>Kotlin version</b></summary><p>

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

</p></details>

Т. к. слой **domain** ничего не знает о классах других слоев, то маппинг моделей должен выполняться во внешних слоях, т. е. репозиторием (при конвертации **data** > **domain** или **domain** > **data**) или презентером (при конвертации **domain** > **presentation** и наоборот) .

#### ResourceManager

В некоторых случаях может потребоваться получить строку или число из ресурсов приложения в Presenter'е или слое **domain** . Однако, мы знаем, что они не должны напрямую взаимодействовать с фреймворком Android. Чтобы решить эту проблему мы можем создать специальную сущность ResourceManager, для доступа к внешним ресурсам. Для этого мы создаем интерфейс:

```java
public interface ResourceManager {

    String getString(int resourceId);

    int getInteger(int resourceId);

}
```

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
interface ResourceManager {

    fun getString(resourceId: Int): String

    fun getInteger(resourceId: Int): Int

}
```

</p></details>

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

<details><summary><b>Kotlin version</b></summary><p>

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

</p></details>

Далее мы должны связать интерфейс и реализацию нашего ResourceManager'а в ApplicationModule:

```java
@Singleton
@Provides
protected ResourceManager provideResourceManager(AndroidResourceManager resourceManager) {
    return resourceManager
}
```

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
@Singleton
@Provides
fun  provideResourceManager(resourceManager: AndroidResourceManager) : ResourceManager = resourceManager
```

</p></details>

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

<details><summary><b>Kotlin version</b></summary><p>

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

</p></details>

Наверное, у внимательных читателей возник вопрос: почему мы используем класс **R** в Presenter'е? Ведь он также относится к Android? На самом деле, это не совсем так. Класс **R** вообще не использует никакие классы, и представляет из себя набор идентификаторов ресурсов. Поэтому, нет ничего плохого, чтобы использовать его в Presenter'е.

#### SchedulersProvider

Перед началом тестирования нам нужно сделать все операции синхронными. Для этого мы должны заменить все Scheduler'ы на **TestScheduler**, поэтому мы не устанавливаем Scheduler'ы напрямую через класс **Schedulers**, а используем **SchedulersProvider**:

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

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
open class SchedulersProvider @Inject constructor() {

    open fun ui(): Scheduler = AndroidSchedulers.mainThread()
    open fun computation(): Scheduler = Schedulers.computation()
    open fun io(): Scheduler = Schedulers.io()
    open fun newThread(): Scheduler = Schedulers.newThread()
    open fun trampoline(): Scheduler = Schedulers.trampoline()

}
```

</p></details>

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

<details><summary><b>Kotlin version</b></summary><p>

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

</p></details>

Далее, при самом тестировании, нам нужно будет лишь использовать TestSchedulersProvider вместо SchedulersProvider. Более подробно о тестировании кода с RxJava можно почитать [здесь](https://github.com/Froussios/Intro-To-RxJava/blob/master/Part%204%20-%20Concurrency/2.%20Testing%20Rx.md).

## Обработка ошибок

[раздел на доработке]

## Тестирование

Одним из самых главных преимуществ чистой архитектуры является то, что мы можем покрыть тестами намного больший функционал приложения за счет разбиения кода на мелкие классы, каждый из которых выполняет строго определенную задачу. Благодаря принципу инверсии зависимостей, используемому в чистой архитектуре мы можем с легкостью подменять реализацию тех или иных классов на фейковые, которые реализуют нужное нам поведение.

Прежде чем начать писать тесты, мы должны ответить себе на два вопроса:

- что мы хотим тестировать?
- как мы будем это тестировать?

Что мы хотим тестировать:

- Мы хотим проверить нашу бизнес-логику независимо от какого-либо фреймворка или библиотеки.
- Мы хотим протестировать нашу интеграцию с API.
- Мы хотим протестировать нашу интеграцию с нашей системой персистентности.
- Все, что содержит условия.

Что мы НЕ должны тестировать:

- Сторонние библиотеки (мы предполагаем, что они работают правильно, потому что уже протестированы разработчиками)
- Тривиальный код (например, геттеры и сеттеры)

Теперь, давайте разберём то, как мы будем тестировать каждый из слоев.

### Тестирование слоя представления

Данный слой включает в себя 2 типа тестов:  Unit-тесты и UI-тесты.

- Unit-тесты используются для тестирования Presenter'ов.
- UI-тесты используются для тестирования Activity (проверяется корректность отображения элементов и т. д.).

Существуют различные соглашения по именованию тестовых методов. Например, в этой [статье](https://dzone.com/articles/7-popular-unit-test-naming) описаны некоторые из них. В примерах, которые я буду приводить далее, я не буду придерживаться какого-либо соглашения. Самое главное понять из названия, что тестирует наш метод и что мы хотим получить в результате. 

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

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
class ArticlesListPresenterTest {

    @Test
    fun shouldLoadArticlesOnViewAttached() {
        //preparing
        val interactor = Mockito.mock(ArticlesListInteractor::class.java)
        val schedulers = TestSchedulersProvider()
        val presenter = ArticlesListPresenter(interactor, schedulers)
        val view = Mockito.mock(ArticlesListView::class.java)

        val articlesList = ArrayList<Article>
        `when`(interactor.getArticlesList()).thenReturn(Single.just(articlesList))

        //testing
        presenter.attachView(view)

        //asserting
        verify(view, times(1)).showLoadingProgress(true)
        verify(view, times(1)).showLoadingProgress(false)
        verify(view, times(1)).showArticles(articlesList)
    }

}
```

</p></details>

Как видите, мы разделили код теста на три части:

- Подготовка к тестированию. Здесь мы инициализируем объекты для тестирования, подготавливаем тестовые данные, а также предопределяем поведение моков.
- Само тестирование. 
- Проверка результатов тестирования. Здесь мы проверяем, что у View были вызваны нужные методы и переданы аргументы.

### Тестирование бизнес-логики

В данном слое тестируюится классы Interactor'ов и Entity. Необходимо проверить, действительно ли бизнес-логика реализует требуемое поведение .

[раздел на доработке]

### Тестирование слоя работы с данными

[раздел на доработке]

## Начало разработки приложения с использованием Clean Architecture

Если вы начиначете разработку нового мобильного приложения, то лучше начать с создания пользовательского интерфейса, т. к. именно UI определяет 

[раздел на доработке]

## Перенос на Clean Architecture существующих проектов

[раздел на доработке]

#### Шаг 1: 

#### Шаг 2: 

#### Шаг 3:

## FAQ по Clean Architecture

#### Стоит ли переписывать весь проект при переносе проекта на Clean Architecture?

Наверное, нет однозначного ответа на этот вопрос. Если проект большой и переход на Clean Architecture может занять длительный промежуток времени, то лучше переписывать код постепенно, используя подход, который мы описали выше. Если же проект простой и состоит из 2-3 экранов, а сроки не поджимают, то вы можете попробовать переписать проект с нуля.

В конце хочется привести поучительную историю про Netscape, который переписывали с нуля больше, чем три года - [Things You Should Never Do, Part I](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/)

#### Обязательно ли создавать отдельные сущности для каждого из слоев (Domain, Data, Presentaion)?

Согласно принципам Clean Architecture, слой Domain ничего не должен знать о внешних слоях (Data и Presentation), но внешние слои без проблем могут использовать классы из слоя Domain. Следовательно, можно не создавать отдельные сущности для каждого из слоев, а использовать только те, что лежат в слое Domain. Однако, если их формат не совпадает с тем, что используется во внешних слоях, то нужно создать отдельную сущность. Если вам нужно добавить к моделям аннотации от библиотек (например, от [Gson](https://github.com/google/gson) или [Room](https://developer.android.com/topic/libraries/architecture/room.html)), то вы можете добавить их прямо в модели Domain-слоя, несмотря на то, что это внешние библиотеки слоя Data, т. к. создание отдельных моделей приведёт к ненужному дублированию кода.

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

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
interface LoginPresenter {

	fun onLoginButtonPressed(email: String, password: String)
  
}

class LoginPresenterImpl : LoginPresenter {
	...
}
```

</p></details>

Нет, интерфейсы для презентера и интерактора создавать не нужно. Это создает дополнительные сложности при разработке, при этом пользы от данного подхода практически нет. Вот лишь некоторые проблемы, которые порождает создание лишних интерфейсов:

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

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
@Module
class ArticleDetailsModule(val articleId: Long) {

    @Provides
    @Presenter
    fun provideArticleId() = articleId

}
```

</p></details>

Далее нам нужно добавить наш модуль в Component:

```java
@Component(dependencies = ApplicationComponent.class, modules = ArticleDetailsModule.class)
public interface ArticleDetailsComponent {
```

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
@Component(dependencies = [ApplicationComponent::class], modules = [ArticleDetailsModule::class])
interface ArticleDetailsComponent {
```

</p></details>

При создании Component'а мы должны передать наш модуль с идентификатором:

```java
long articleId = ...

ArticleDetailsComponent component = DaggerArticleDetailsComponent.builder()
    .applicationComponent(MyApplication.getComponent())
    .articleDetailsModule(new ArticleDetailsModule(articleId))
    .build();
```

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
val component = DaggerArticleDetailsComponent.builder()
    .applicationComponent(MyApplication.getComponent())
    .articleDetailsModule(ArticleDetailsModule(articleId))
    .build();
```

</p></details>

Теперь мы можем получить наш идентификатор через конструктор:

```java
@Inject
public UserFollowersPresenter(ArticleDetailsInteractor interactor, long articleId) {
    this.interactor = interactor;
    this.articleId = articleId;
}
```

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
class UserFollowersPresenter @Inject constructor(
        private val interactor: ArticleDetailsInteractor,
        private val articleId: Long
)
```

</p></details>

Теперь представим, что помимо идентфикатора статьи, мы хотим передать ID пользователя, который так же имеет тип long. Если мы попытаемся создать ещё один provide-метод в нашем модуле, Dagger выдаст ошибку, о том, что типы совпадают и он не знает какой из них является идентфикатором статьи, а какой идентфикатором пользователя. 

Чтобы исправить это, нам необходимо создать Qualifier-аннотации,  которые будут указывать Dagger'у "who is who":

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

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
@Qualifier
@Retention(AnnotationRetention.RUNTIME)
annotation class ArticleId

@Qualifier
@Retention(AnnotationRetention.RUNTIME)
annotation class UserId
```

</p></details>

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

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
@ArticleId
@Provides
@Presenter
fun provideArticleId() = articleId

@UserId
@Provides
@Presenter
fun provideUserId() = userId
```

</p></details>

Также нужно пометить аннотациями аргументы конструктора:

```java
@Inject
public UserFollowersPresenter(ArticleDetailsInteractor interactor, @ArticleId long articleId, @UserId long userId) 
```

<details><summary><b>Kotlin version</b></summary><p>

```kotlin
class UserFollowersPresenter @Inject constructor(
        private val interactor: ArticleDetailsInteractor,
        @ArticleId private val articleId: Long,
        @UserId private val userId: Long
)
```

</p></details>

Готово. Теперь Dagger сможет верно расставить аргументы в конструктор.