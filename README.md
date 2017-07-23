# CleanArchitectureManifest

Здесь вы найдете описание основных принципов и правил, которыми стоит руководствоваться при разработке Android-приложений с использованием чистой архитектуры.

<p align="center">
    <img src="https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/TheCleanArchitecture.png">
</p>

# Содержание

- Введение
  - [Что такое Clean Architecture?](#Что-такое-Clean-Architecture?)
  - [Какие проблемы решает?](#quick-start)
- [Слои и сущности](#creating-new-document)
  - [Слой отображения (presentation)]()
    - View
    - Presenter
    - Связывание View с Presenter'ом
    - Model
  - [Слой бизнес-логики (domain)]()
  - [Слой работы с данными (data)]()
  - [Взаимодействие между слоями]()
 - [Дополнительные сущности, используемые на практике]()
- Обработка ошибок
- Тестирование
- FAQ по Clean Architecture

# Что такое Clean Architecture? 

Clean Achitecture — принцип разработки приложений, [предложенный](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html) Робертом Мартином в 2012 году. После него данный подход был [адаптирован](https://fernandocejas.com/2014/09/03/architecting-android-the-clean-way/) под реалии Android-разработки Fernando Cejas'ом. Суть Clean Architecture заключается в разделении логики приложения на несколько слоев, каждый из которых выполняет строго определенную задачу.

# Какие проблемы решает?

Благодаря разделению ответственности между классами в  Clean Architecture, код приложения становится независимым от базы данных или от внешнего вида приложения. Мы легко можем изменять код приложения, а также добавлять новый функционал, затрагивая при этом минимальное количество классов. Помимо этого мы получаем легко [тестируемый код](http://misko.hevery.com/attachments/Guide-Writing%20Testable%20Code.pdf). Стоит заметить, что построение правильной архитектуры целиком и полностью зависит от самого разработчика и его опыта.

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

### Entitities 

**Entitities (бизнес-модели)** - представляет из себя POJO-класс. Это ядро вашего приложения. Они отражают функционал приложения, и вы должны описать то, что делает приложение просто взглянув на них. Например, если выделаете новостное приложение, то для него бизнес-моделями будут классы CategoryEntity и ArticleEntity. Они наименее подвержены изменениям, когда что-либо в приложении меняется. Пример Entity:

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

Как вы могли заметить, репозиторий возвращает не просто статьи, а класс из RxJava 2 -  [Single](http://reactivex.io/documentation/single.html). Благодаря этому мы легко можем сделать операцию извлечения данных синхронной/асинхронной, просто поменяв [Scheduler](http://reactivex.io/documentation/scheduler.html):

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

В данном туториале в качестве примера будет использован шаблон MVP, но вы можете выбрать любой другой (MVVM, MVI). 

Для более удобной связки View и Presenter мы будем использовать библиотеку [Moxy](https://github.com/Arello-Mobile/Moxy). Она помогает решить многие проблемы, связанные с жизненным циклом Activity или Fragment'а. Moxy имеет базовые классы, такие как ```MvpView``` и ```MvpPresenter``` от которых должны наследоваться наши View и Presenter. Для избежания написания большого количества кода по связыванию View и Presenter, Moxy использует кодогенерацию. Для правильной работы кодогенерации мы должны использовать специальные аннотации, которые предоставляет нам Moxy. Более подробную информацию о библиотеке можно найти [здесь](https://habrahabr.ru/post/276189/).

<p align="center">
    <img src="https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/PresentationLayer.png">
</p>

### View

View отвечает за то, каким образом данные будут показаны пользователю. Также View сообщает о действиях пользователя Presenter'у, будь то нажатие на кнопку или ввод текста. Пример View:
```
public interface ArticlesListView extends MvpView {

    void showVisits(List<ArticleEntity> articles);
    void showErrorMessage(String text);

}
```

**Внимание:** Не путайте базовый класс для UI-элементов android.view.View с понятием View в контексте MVP.

### Presenter

Является связующим звеном между Interactor'ом и View. В обязанности Presenter'а принятие решений о том, что и когда отображать. Помимо этого Presenter реагирует на действия пользователя, о которых ему сообщила View. Пример Presenter'а:

```
@InjectViewState
public class ArticlesListPresenter extends MvpPresenter<ArticlesListView> {

	@Inject
    ArticlesListInteractor articlesListInteractor;

    public VisitsPresenter(ApplicationComponent applicationComponent) {

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
		return new ArticlesListPresenter(MyApplication.component());
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

Так как конструктор нашего Presenter'а не пустой, а принимает на вход ```ApplicationComponent```, то мы должны сами создать объект Presenter'а и предоставить его нашему генератору кода. Мы делаем это при помощи метода ```provideArticlesListPresenter```, который мы пометили аннотацией ```@ProvidePresenter```. Хочется отметить, что как и во всех других случаях использования кодогенерации, переменные и методы, помеченные аннотациями, должны быть видны на уровне пакета, т. е. у них не должно быть модификаторов видимости (private, public, protected).

Объект класса ```ApplicationComponent``` является общим для всего приложения, однако в случае, когда вам нужны специфические для данного Presenter'а классы, вы можете создать компонент специально для этого Presenter'а, например, ArticlesListPresenterComponent.

### Model

Во многих статьях про MVP, Model описывается как слой отвечающий за работу с данными, будь то база данных, файлы или запросы к серверу. В связке Clean Architecture + MVP, фактически остаются только слои View и Presenter. Роль Model берет на себя слой **data** в Clean Architecture.

## Слой работы с данными (data)

<p align="center">
    <img src="https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/DataLayer.png">
</p>

Мостом между слоями data и domain является интерфейс Repository. Сам интерфейс находится в слое domain, а уже реализация располагается в слое data.

### Repository

**Repository** - представляет из себя интерфейс, с которым работает UseCase. В нем описывается какие данные хочет получать UseCase от внешних слоев. В приложении может быть несколько репозиториев, в зависимости от задачи. Например, если мы делаем  новостное приложение, репозиторий работающий со статьями может называться ArticleRepository. а репозиторий для работы с комментариями CommentRepository. Пример репозитория, работающего со статьями:

```java
public interface ArticleRepository {

  Single<ArticleEntity> getArticle(String articleId);

  Single<List<ArticleEntity>> getLastNews();
  
  Single<List<ArticleEntity>> getCategoryArticles(String categoryId);
  
  Single<List<ArticleEntity>> getRelatedPosts(String articleId);
  
}
```

### DataSource

(БД, файлы, сеть, кеш в оперативной памяти)

## Разбиение на пакеты


## Взаимодействие между слоями

Для связки всех трех слоев используется Dependency Injection. В нашем примере мы используем библиотеку Dagger 2.

## Дополнительные сущности, используемые на практике

**Router** - класс, для осуществления переходов между экранами (активити или фрагментами).

**Mapper** - специальный класс, для конвертирования моделей из одного типа в другой, например, из модели БД (DTO) в модель бизнес-логики (Entity). Обычно они имеют название типа XxxMapper/XxxConverter, и имеют единственный метод с названием map/convert/transform, например:

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



#  Тестирование



#  Перенос на Clean Architecture существующих проектов

# FAQ по Clean Architecture

- **С какого слоя лучше всего начать разработку приложения, если оно разрабатывается с нуля?**
Ответ
- **Нужно ли создавать интерфейсы для классов Presenter и Interactor?**
Ответ
- **Как перенести уже существующее приложение на Clean Architecture?**
Ответ
