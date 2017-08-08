# CleanArchitectureManifest (v 0.7.1)

Здесь вы найдете описание основных принципов и правил, которыми стоит руководствоваться при разработке Android-приложений с использованием чистой архитектуры.

<p align="center">
    <img src="https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/TheCleanArchitecture.png">
</p>

# Содержание

- Введение
  - [Что такое Clean Architecture?](#Что-такое-clean-architecture)
  - [Какие проблемы решает?](#Какие-проблемы-решает)
- [Слои и сущности](#Слои-и-сущности)
  - [Слой отображения (presentation)](#Слой-отображения-presentation)
    - [View](#view)
    - [Presenter](#presenter)
    - [Связывание View с Presenter'ом](#Связывание-view-с-presenterом)
    - [Model](#model)
  - [Слой бизнес-логики (domain)](#Слой-бизнес-логики-domain)
    - [Entities](#entities)
    - [UseCase](#usecase)
    - [Interactor](#interactor)
  - [Слой работы с данными (data)](#Слой-работы-с-данными-data)
    - [Repository](#repository)
    - [DataStore](#datastore)
  - [Взаимодействие между слоями](#Взаимодействие-между-слоями)
 - [Дополнительные сущности, используемые на практике](#Дополнительные-сущности-используемые-на-практике)
    - [Router](#router)
    - [Mapper](#mapper)
- [Тестирование](#Тестирование)
- [Обработка ошибок](#Обработка-ошибок)
- [Перенос на Clean Architecture существующих проектов](#Перенос-на-clean-architecture-существующих-проектов)
- [FAQ по Clean Architecture](#faq-по-clean-architecture)

# Что такое Clean Architecture? 

Clean Achitecture — принцип разработки приложений, [предложенный](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html) Робертом Мартином в 2012 году. После него данный подход был [адаптирован](https://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/) под реалии Android-разработки Fernando Cejas'ом. Суть Clean Architecture заключается в разделении логики приложения на несколько слоев, каждый из которых выполняет строго определенную задачу.

# Какие проблемы решает?

Благодаря разделению ответственности между классами в  Clean Architecture, код приложения становится независимым от базы данных или от внешнего вида приложения. Мы легко можем изменять код приложения, а также добавлять новый функционал, затрагивая при этом минимальное количество классов. Помимо этого мы получаем легко [тестируемый код](http://misko.hevery.com/attachments/Guide-Writing%20Testable%20Code.pdf). Стоит заметить, что построение правильной архитектуры целиком и полностью зависит от самого разработчика и его опыта.

Преимущества чистой архитектуры:

- более высокий процент покрытия кода
- повышенная простота навигации по структуре пакета
- проект легче поддерживать, поскольку классы более сфокусированы
- позволяет быстрее добавлять новые функции
- Чистая архитектура - это четкая дисциплина и в целом хорошая практика

Недостатки чистой архитектуры:

- большое количество классов (хотя именно за счет этого классы становятся переиспользуемыми)
- доавольно высокий порого вхождения и зачастую неправильное использование на первых порах использования


# Слои и сущности

<p align="center">
    <img src="https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/CleanArchitectureLayers.png">
</p>

Архитектуру Android-приложения, построенную по принципу Clean Architecture можно разделить на три слоя: слой отображения (presentation), слой бизнес-логики (domain), слой работы с данными (data). 

## Слой бизнес-логики (domain)

<p align="center">
    <img src="https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/DomainLayer.png">
</p>

Главным слоем в Clean Arcitecture является слой бизнес-логики. Именно он определяет поведение приложения. Данный слой включает в себя три сущности: Entity, UseCase, Interactor. В отличие от слоев data и presentation, domain-слой не использует классы из фреймворка Android, ограничиваясь классами из "чистой" Java.

### Entities 

**Entities (бизнес-модели)** - представляет из себя POJO-класс. Это ядро вашего приложения. Они отражают функционал приложения, и вы должны описать то, что делает приложение просто взглянув на них. Например, если выделаете новостное приложение, то для него бизнес-моделями будут классы CategoryEntity и ArticleEntity. Они наименее подвержены изменениям, когда что-либо в приложении меняется. Пример Entity:

```java
public class ArticleEntity {

  private long id;
  private String title;
  private String  previewText;
  private String articleUrl;

  public ArticleEntity(String title, String previewText, String articleUrl) {
    this.title = title;
    this.previewText = previewText;
    this.articleUrl = articleUrl;
  }

  public String getTitle() {
    return title;
  }

  public String getPreviewText() {
    return previewText;
  }

  public String getArticleUrl() {
      return articleUrl;
  }
  
}
```

### UseCase

**UseCase** - классы, содержащие в себе бизнес-логику, и выполняющее одно конкретное действие. Признаком хорошего UseCase'а является то, что его можно описать одним приложение, например, "Переводит деньги с одного счета на другой". Вы даже можете давать названия классам типа TransferMoneyUseCase. Ниже приведен пример UseCase'а, получающего список последних статей и сортирующего их по рейтингу:

```java
articleRepository.getLastNews()
.toObservable
.toSortedList(new SortArticleByRatingComparator())
.flatMap(Observable::from)
.toSortedList()
```

### Interactor

**Interactor** - набор UseCase'ов, связанных между собой. Обычно для каждого экрана приложения создается свой Interactor. Например, если мы делаем новостное приложение, то Interactor для экрана с содержанием статьи будет выглядеть примерно так:

```java
public class ArticleDetailInteractor {

    private GetArticleUseCase getArticleUseCase;
    private GetRelatedArticlesUseCase getRelatedArticlesUseCase;

    public ArticleDetailInteractor(GetArticleUseCase getArticleUseCase,
                              GetRelatedArticlesUseCase relatedArticlesUseCase) {
        this.getArticleUseCase = getArticleUseCase;
        this.relatedArticlesUseCase = relatedArticlesUseCase;
    }

    public Single<ArticleEntity> getArticle(String articleId) {
        return getArticleUseCase.getSingle(articleId);
    }

    public Single<List<ArticleEntity>> getRelatedArticles(String articleId) {
        return getRelatedArticlesUseCase.getSingle(articleId);
    }

}
```

## Слой отображения (presentation)

<p align="center">
    <img src="https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/PresentationLayer.png">
</p>

**Внимание:** В данном туториале в качестве примера будет использован шаблон MVP, но вы можете выбрать любой другой (MVVM, MVI).

MVP расшифровывается как Model-View-Presenter (модель-представление-презентер). В случае с Android в качестве View выступает Activity или Fragment. Model содержит в себе бизнес-логику и код по работе с данными. Согласно концепции MVP, View не может напрямую взаимодействовать с Model, поэтому связующим звеном между ними является Presenter. Т. к. мы используем связку CleanArchitecture + MVP, то Model у нас является код находящийся в слоях Data (работа с данными) и Domain (бизнес-логика). Следовательно в слое Presentation остаются лишь два компонента - View и Presenter.

Для более удобной связки View и Presenter мы будем использовать библиотеку [Moxy](https://github.com/Arello-Mobile/Moxy). Она помогает решить многие проблемы, связанные с жизненным циклом Activity или Fragment'а. Moxy имеет базовые классы, такие как ```MvpView``` и ```MvpPresenter``` от которых должны наследоваться наши View и Presenter. Для избежания написания большого количества кода по связыванию View и Presenter, Moxy использует кодогенерацию. Для правильной работы кодогенерации мы должны использовать специальные аннотации, которые предоставляет нам Moxy. Более подробную информацию о библиотеке можно найти [здесь](https://habrahabr.ru/post/276189/).

### View

View отвечает за то, каким образом данные будут показаны пользователю. Также View сообщает о действиях пользователя Presenter'у, будь то нажатие на кнопку или ввод текста. Пример View:
```
public interface ArticlesListView extends MvpView {

    void showVisits(List<ArticleEntity> articles);
    void showErrorMessage(String text);

}
```
Пока мы описали лишь интерфейс View, т. к. какие команды Presenter может отдавать View. Обратите внимание, что наш интерфейс наследуется от интерфейса MvpView, входящего в библиотеку Moxy. Это является обязательным условием для корректной работы библиотеки.

**Внимание:** Не путайте базовый класс для UI-элементов android.view.View с понятием View в контексте MVP.

### Presenter

Является связующим звеном между Interactor'ом и View. В обязанности Presenter'а принятие решений о том, что и когда отображать. Помимо этого Presenter реагирует на действия пользователя, о которых ему сообщила View. Пример Presenter'а:

```
@InjectViewState
public class ArticlesListPresenter extends MvpPresenter<ArticlesListView> {

	@Inject
    ArticlesListInteractor articlesListInteractor;

    public VisitsPresenter(ArticlesListPresenterComponent articlesListPresenterComponent) {

        applicationComponent.inject(this);
    }
    
    public void showArticles() {
        articlesListInteractor.getArticles()
                .subscribe(articles -> getViewState().showArticles(articles),
                        throwable -> handleError(throwable));
    }

    private void handleError(Throwable throwable) {
			getViewState().showErrorMessage(throwable.getMessage());
    }

}
```

В конструкторе Presenter'а мы получаем ApplicationComponent, который предоставляет нужные нам зависимости.

### Связывание View с Presenter'ом

В контексте разработки под Android роль View на себя берет Activity (или Fragment), поэтому после создания интерфейса View, мы должны реализовать его нашей Activity или Fragment'е:

```
public class ArticlesListActivity extends MvpAppCompatActivity implements ArticlesListView {

	@InjectPresenter
	ArticlesListPresenter articlesPresenter;
	
	@ProvidePresenter
	ArticlesListPresenter provideArticlesListPresenter() {
    ArticlesListPresenterComponent articlesListPresenterComponent = DaggerArticlesListPresenterComponent.builder()
                .applicationComponent(MyApplication.component())
                .build();
		return new ArticlesListPresenter(articlesListPresenterComponent);
	}

	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_articles_list);

        articlesPresenter.showArticles();
    }
    
	public void showArticles(List<ArticleEntity> articles) {
		...
	}

	public void showErrorMessage(String text) {
		...
	}

}
```

Хочу заметить, что для правильной работы библиотеки Moxy, наша Activity должна обязательно наследоваться от класса MvpAppCompatActivity (или MvpAppCompatFragment в случае, если вы используете фрагменты). С помощью аннотации ```@InjectPresenter``` мы сообщаем Annotation Processor'у в какую переменную нужно "положить" Presenter.

Так как конструктор нашего Presenter'а не пустой, а принимает на вход ```ArticlesListPresenterComponent```, то мы должны сами создать объект Presenter'а и предоставить его нашему генератору кода. Мы делаем это при помощи метода ```provideArticlesListPresenter```, который мы пометили аннотацией ```@ProvidePresenter```. Хочется отметить, что как и во всех других случаях использования кодогенерации, переменные и методы, помеченные аннотациями, должны быть видны на уровне пакета, т. е. у них не должно быть модификаторов видимости (private, public, protected).

### Model

Во многих статьях про MVP, Model описывается как слой отвечающий за работу с данными, будь то база данных, файлы или запросы к серверу. В связке Clean Architecture + MVP, фактически остаются только слои View и Presenter. Роль Model берет на себя слой **data** в Clean Architecture.

## Слой работы с данными (data)

<p align="center">
    <img src="https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/DataLayer.png">
</p>

Мостом между слоями data и domain является интерфейс Repository. Сам интерфейс находится в слое domain, а уже реализация располагается в слое data.

### Repository

**Repository** - представляет из себя интерфейс, с которым работает UseCase. В нем описывается какие данные хочет получать UseCase от внешних слоев. В приложении может быть несколько репозиториев, в зависимости от задачи. Например, если мы делаем  новостное приложение, репозиторий работающий со статьями может называться ArticleRepository, а репозиторий для работы с комментариями CommentRepository. Пример репозитория, работающего со статьями:

```java
public interface ArticleRepository {

  Single<ArticleEntity> getArticle(String articleId);

  Single<List<ArticleEntity>> getLastNews();
  
  Single<List<ArticleEntity>> getCategoryArticles(String categoryId);
  
  Single<List<ArticleEntity>> getRelatedPosts(String articleId);
  
}
```

Как вы могли заметить, репозиторий возвращает не просто статьи, а класс из RxJava 2 -  [Single](http://reactivex.io/documentation/single.html). Благодаря этому мы легко можем сделать операцию извлечения данных синхронной/асинхронной, просто поменяв [Scheduler](http://reactivex.io/documentation/scheduler.html). Помимо класса Single могут быть использованы другие классы RxJava 2, такие как Completable, Maybe, Observable, Flowablr. Использование каждого из них обуславливается решаемой задачей. Подробнее о классах RxJava 2 можно почитать здесь - [What's different in 2.0](https://github.com/ReactiveX/RxJava/wiki/What%27s-different-in-2.0#observable-and-flowable);

### DataSource

(БД, файлы, сеть, кеш в оперативной памяти)

## Разбиение классов по пакетам

Как известно есть два стособа разбиения классов: по фичам и по типу классов.

Пример разбиения по фичам:


Пример разбиения по типу классов:

```java
app(module)
|
|----data
|    |
|    |----repository
|    |----cache
|    |----database
|    |----api
|    |----exception
|    |----shared
|
|----domain
|    |
|    |----model
|    |----interactor
|    |----executor
|    |----api
|    |----abstraction
|    |----shared
|
|----presentation
|    |
|    |----presenter
|    |----view
|    |----navigation
|    |----shared
|
|----di
|    |
|    |----modules
|    |----scopes
|    |----modifiers
```

[раздел на доработке]

## Взаимодействие между слоями

Для связки всех трех слоев используется Dependency Injection. В нашем примере мы используем библиотеку Dagger 2.

## Дополнительные сущности, используемые на практике



### Router

Т. к. Presenter содержит в себе логику реагирования на действия пользователя, то он также знает о том, на какой экран нужно перейти. Однако сам Presenter не может осуществлять переход на новый экран, т. к. для этого нам требуется Context. Поэтому за открытие нового экрана должна отвечать View. Для осуществления перехода на следующий экран мы должны вызвать метод View, например, openProfileScreen(), а уже в реализации самого метода осуществлять переход. Помимо данного подхода некоторые разработчики используют для навигации так называемый Router.

***Router*** - класс, для осуществления переходов между экранами (активити или фрагментами).

Для реализации Router'а мы рекомендуем использовать библиотеку [Alligator](https://github.com/aartikov/Alligator)

### Mapper

Mapper - специальный класс, для конвертирования моделей из одного типа в другой, например, из модели БД (DTO) в модель бизнес-логики (Entity). Обычно они имеют название типа XxxMapper/XxxConverter, и имеют единственный метод с названием map/convert/transform, например:

```java
public class ArticleModelToEntityMapper {

	public ArticleEntity map(ArticleModel model) {
		return new EntityProfile(model.getName(), model.getLastname, model.getAge());
	}
	
	public List<ArticleEntity> map(Collection<ArticleModel> models) {
        final List<ToArticleEntity> result = new ArrayList<>(models.size());
        for (ArticleModel model : models) {
            result.add(map(model));
        }
        return result;
    }

}
```

# Обработка ошибок

[раздел на доработке]

# Тестирование

Одним из самых главных преимуществ является то, что мы можем покрыть тестами намного больший функционал приложения, за счет разбиения кода на мелкие классы, каждый из которых выполняет строго определенную задачу. Однако, тестировать нужно не все классы, а лишь те, в которых содержится бизнес-логика в слое Domain, а именно UseCase'ы и Interactor'ы. Также тестами желательно покрыть некоторые классы DataStore из слоя Data, и Presenter'ы из слоя Presentation.

[раздел на доработке]

# Начало разработки приложения с использованием Clean Architecture

Если вы начиначете разработку нового мобильного приложения, то лучше начать с создания пользовательского интерфейса, т. к. именно UI определяет 

[раздел на доработке]

# Перенос на Clean Architecture существующих проектов

##  Этап 1: Разбиение Activity на View и Presenter
##  Этап 2: Выделяем работу с дынными в отдельные классы
##  Этап 3: Выделяем UseCase'ы и Interactor

[раздел на доработке]

# FAQ по Clean Architecture

- **Стоит ли переписывать весь проект при переносе проекта на Clean Architecture?**

Наверное, нет однозначного ответа на этот вопрос. Если проект большой и переход на Clean Architecture может длительный промежуток времени, то лучше переписывать код постепенно, используя подход, который мы описали выше. Если же проект простой и состоит из 2-3 экранов, а сроки не поджимают, то вы можете попробовать переписать проект с нуля.

В конце хочется привести поучительную историю про Netscape, который переписывали с нуля больше, чем три года - [Things You Should Never Do, Part I](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/)

- **Нужно ли создавать интерфейсы для классов Presenter и Interactor для улучшения тестируемости кода?**

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

- Если мы хотим добавить новый метод или изменить существующий, нам нужно изменить интерфейс. Помимо это мы также должны изменить реализацию метода. Это занимает довольно времени, даже при использовании такой продвинутой IDE как Android Studio.
- Использование дополнительных интерфейсов усложняет навигацию по коду. Если вы хотите перейти к реализации метода Presenter'а из Activity (т. е. реализации View), то вы переходите к интерфейсу Presenter'а.
- Интерфейс никак не улучшает тестируемость кода. Вы с легкостью можете заменить класс Presenter'а на mock, используя любую библиотеку для mock'ирования.

Более подробно можете почитать об этом в следующих статьях:

- [Interfaces for presenters in MVP are a waste of time!](http://blog.karumi.com/interfaces-for-presenters-in-mvp-are-a-waste-of-time)
- [Франкен-код или франкен-дизайн](https://plus.google.com/u/0/+SergeyTeplyakov/posts/gRaqrqaiGbe)
- [Управление зависимостями](http://sergeyteplyakov.blogspot.ru/2012/11/blog-post.html)

- **Нужно ли создавать отдельный класс Component для каждого Presenter'а?**

[раздел на доработке]

Все зависит от ситуации и сценария использования. 


Объект класса ```ApplicationComponent``` является общим для всего приложения, однако в случае, когда вам нужны специфические для данного Presenter'а классы, вы можете создать компонент специально для этого Presenter'а, например, ArticlesListPresenterComponent.

- **Есть ли какие-нибудь правила по именованию методов View и Presenter'а?**

Мы рекомендуем придерживаться следующего соглашения:

- методы презентера вызвываются из View только при каком-либо событии (```onSomethingClicked```, ```onSomethingEdited```, ```onViewLoaded```). Т. е. View сообщает о событии, но не говорит Presenter'у что делать (например, ```doSomething```)
- методы View, вызываются Presenter'ом для показа чего-либо (```showSomeError```, ```setSomeButtonEnabled```)

В итоге мы получаем пассивную View, которая отображает то, что ей приказыват Presenter, а Presenter, в свою очередь, управляет логикой отображения.

Также хочется отметить, что ни у View, ни у Presenter'а не может быть методов, которые возвращают что-либо (getSomething). Если у вас возникает такая потребность, то, скорее всего, вы неправильно организовали работу View и Presenter'а.