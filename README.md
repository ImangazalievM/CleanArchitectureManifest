# CleanArchitectureManifest (v 0.9.0)



Здесь вы найдете описание основных принципов и правил, которыми стоит руководствоваться при разработке Android-приложений с использованием чистой архитектуры. 

![CleanArchitectureManifest](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/CleanArchitectureManifest.png)

## Содержание

- [Введение](Введение)
- [Слои и инверсия зависимостей](#Слои-и-инверсия-зависимостей)
  - [Слой отображения (presentation)](#Слой-отображения-presentation)
    - [Model](#model)
    - [View](#view)
    - [Presenter](#presenter)
    - [Связывание View с Presenter'ом](#Связывание-view-с-presenterом)
  - [Слой бизнес-логики (domain)](#Слой-бизнес-логики-domain)
  - [Слой работы с данными (data)](#Слой-работы-с-данными-data)
    - [Repository](#repository)
    - [DataStore](#datastore)
  - [Взаимодействие между слоями](#Взаимодействие-между-слоями)
 - [Дополнительные сущности, используемые на практике](#Дополнительные-сущности-используемые-на-практике)
    - [Router](#router)
    - [Mapper](#mapper)
- [Обработка ошибок](#Обработка-ошибок)
- [Тестирование](#Тестирование)
- [Перенос на Clean Architecture существующих проектов](#Перенос-на-clean-architecture-существующих-проектов)
- [FAQ по Clean Architecture](#faq-по-clean-architecture)

## Введение

Clean Achitecture — принцип построения архитектуры приложения, [предложенный](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html) Робертом Мартином в 2012 году. 

Clean Architecture включает в себя два основных принципа:

1. **Разделение на слои** 
2. **Инверсия зависимостей**

Давайте расшифруем каждый из них.

**Разделение на слои**

Суть принципа заключается в разделении всего кода приложения на слои. Всего мы имеем три слоя: 

- слой бизнес логики
- слой отображения
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
- MVP

Очень желательно, чтобы у вас был практический опыт их использования, так вы быстрее войдете в курс дела. Если вы уже знакомы с ними, то можете смело приступать к прочтению.

**P. S.** Я разработал приложение, чтобы продемонстрировать использование чистой архитектуры на практике. Исходный код вы можете найти здесь - [Bubbble](https://github.com/ImangazalievM/Bubbble).

## Слои и инверсия зависимостей

![CleanArchitectureLayers](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/CleanArchitectureLayers.png)

Как уже говорилось ранее, архитектуру приложения, построенную по принципу Clean Architecture можно разделить на три слоя: 

* слой отображения (presentation)
* слой бизнес-логики (domain)
* слой работы с данными (data)

Выше представлена схема того, как эти слои взаимодействуют. Черными стрелками обозначены зависимость одних слоев от других, а красными - поток данных. Как видите слои **data** и **presentation** зависят от **domain**, т. е. они используют его классы. Сам же слой **domain** ни от чего не зависит и использует только собственные классы и интерфейсы. Далее мы разберем более подробно каждый из этих слоев, а также то, как они взаимодействуют между собой.

## Слой бизнес-логики (Domain)

![DomainLayer](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/DomainLayer.png)

Слой бизнес-логики является "главным" из всех трех слоев. Он включает в себя два типа сущностей **Entity** и **Interactor** (другое название - Use Case). 

**Entity (бизнес-сущность)** - представляет из себя класс. Это ядро вашего приложения. Бизнес-сущности отражают функционал приложения, который можно описать, просто взглянув на них. Например, если вы разрабатваете приложение для интернет-магазина, то для него бизнес-моделями могут быть классы Product, Category и Order. Entity наименее подвержены изменениям, когда что-либо меняется в приложении. Пример Entity:

```java
public class Product {

  private long id;
  private String name;
  private int price;

  public Product(String title, String previewText, String articleUrl) {
    this.title = title;
    this.previewText = previewText;
    this.articleUrl = articleUrl;
  }

  public long getId() {
    return id;
  }

  public String getName() {
    return name;
  }

  public String getPrice() {
    return price;
  }
  
}
```

**Бизнес-логика** - это правила, описывающие, как работает бизнес (например, пользователь не может совершить покупку на сумму больше, чем есть на его счёте). Бизнес-логика не зависит от реализации базы данных или интерфейса пользователя. Бизнес-логика меняется только тогда, когда меняются требования бизнеса, и не зависит от используемой СУБД или интерфейса пользователя. Это достигается за счет использования интерфейсов вместо конкретной реализации.

Классы, в которых заключена бизнес-логика называются Interactor'ами (иногда их называют UseCase'ами). Например, перед тем как отправить заказ на обработку, мы хотим проверить, хватает ли средств на внутреннем аккаунте пользователя для осуществления заказа. Также нам нужно учесть комиссию платежной системы в 18%, которая взимается с пользователя.

```java
public class OrderInteractor {

    private UsersRepository userRepository;
    private CartRepository cartRepository;
    private OrderRepository orderRepository;

    public OrderInteractor(UsersRepository userRepository, 
                           CartRepository cartRepository, 
                           OrderRepository orderRepository) {
        this.userRepository = userRepository;
        this.cartRepository = cartRepository;
        this.orderRepository = orderRepository;
    }
    
    ...
  
    public void sendOrder(DeliveryInfo deliveryInfo, OrderResultListener resultlistener) {
        User user = userRepository.getUserInfo();
    
        int cartTotalSum = cartRepository.getTotalSum();
        int oderTotalSum = cartTotalSum * 1.18;
        
        if (user.getBalance() >= oderTotalSum) {
            Order order = new Order(user, cartRepository.getProducts(), deliveryInfo)
            orderRepository.sendOrder(order);
            orderResultlistener.onOrderSendSuccess();
        } else {
            orderResultlistener.onError(new InsufficientFundsException());
        }
    }

}
```

Иногда Interactor может и вовсе не содержать бизнес-логики, а методы Interactor'а выступают в качестве прослойки между Repository и Presenter'ом:

```java
public class CartInteractor {

    private CartRepository cartRepository;
    
    public OrderInteractor(CartRepository cartRepository) {
        this.cartRepository = cartRepository;
    }
    
    public List<Products> getCartProducts() {
        return cartRepository.getProducts();
    }

}
```

Использование слушателей для получения результатов от Interactora может быть не очень удобным при асинхронных операциях и при обработке ошибок. Чтобы 

```java
public class OrderInteractor {

    private UsersRepository userRepository;
    private CartRepository cartRepository;
    private OrderRepository orderRepository;

    public OrderInteractor(UsersRepository userRepository, 
                           CartRepository cartRepository, 
                           OrderRepository orderRepository) {
        this.userRepository = userRepository;
        this.cartRepository = cartRepository;
        this.orderRepository = orderRepository;
    }
    
    public void sendOrder(DeliveryInfo deliveryInfo, OrderResultListener resultlistener) {
        User user = userRepository.getUserInfo();
    
        int cartTotalSum = cartRepository.getTotalSum();
        int oderTotalSum = cartTotalSum * 1.18;
        
        if (user.getBalance() >= oderTotalSum) {
            Order order = new Order(user, cartRepository.getProducts(), deliveryInfo)
            orderRepository.sendOrder(order);
            orderResultlistener.onOrderSendSuccess();
        } else {
            orderResultlistener.onError(new InsufficientFundsException());
        }
    }
```

### Слой работы с данными (Data)

![DataLayer](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/DataLayer.png)

В данном слое содержатся классы для работы то базой данных, сетью или файловой системой и т. д., а также логика кеширования, если она имеется.

"Мостом" между слоями data и domain является интерфейс Repository (в оригинальной схеме дядюшки Боба он называется Gateway). Сам интерфейс находится в слое domain, а уже реализация располагается в слое data. При этом классы domain-слоя не знают откуда берутся данные - из БД, сети или откуда-то ещё. Именно поэтому вся логика кеширования должна содержаться в data-слое.

#### Repository

**Repository** - представляет из себя интерфейс, с которым работает Interactor. В нем описывается какие данные хочет получать Interactor от внешних слоев. В приложении может быть несколько репозиториев, в зависимости от задачи. Например, если мы делаем  новостное приложение, репозиторий работающий со статьями может называться ArticleRepository, а репозиторий для работы с комментариями CommentRepository. Пример репозитория, работающего со статьями:

```java
public interface ArticleRepository {

  Single<ArticleEntity> getArticle(String articleId);

  Single<List<ArticleEntity>> getLastNews();
  
  Single<List<ArticleEntity>> getCategoryArticles(String categoryId);
  
  Single<List<ArticleEntity>> getRelatedPosts(String articleId);
  
}
```

Как вы могли заметить, репозиторий возвращает не просто статьи, а класс из RxJava 2 -  [Single](http://reactivex.io/documentation/single.html). Благодаря этому мы легко можем сделать операцию извлечения данных синхронной/асинхронной, просто поменяв [Scheduler](http://reactivex.io/documentation/scheduler.html). Помимо класса Single могут быть использованы другие классы RxJava 2, такие как Completable, Maybe, Observable, Flowablr. Использование каждого из них обуславливается решаемой задачей. Подробнее о классах RxJava 2 можно почитать здесь - [What's different in 2.0](https://github.com/ReactiveX/RxJava/wiki/What%27s-different-in-2.0#observable-and-flowable);

#### DataSource

(БД, файлы, сеть, кеш в оперативной памяти)

### Слой отображения (Presentation)

![PresentationLayer](https://raw.githubusercontent.com/ImangazalievM/CleanArchitectureManifest/master/images/PresentationLayer.png)

Слой представления содержит все компоненты, которые связаны с UI, такие как View-элементы, Activity, Fragment'ы и т. д. Для

Для рализацииВ данном туториале в качестве примера будет использован шаблон MVP, но вы можете выбрать любой другой (MVVM, MVI).

MVP расшифровывается как Model-View-Presenter (модель-представление-презентер). В случае с Android в качестве View выступает Activity или Fragment. Model содержит в себе бизнес-логику и код по работе с данными. Согласно концепции MVP, View не может напрямую взаимодействовать с Model, поэтому связующим звеном между ними является Presenter. Т. к. мы используем связку Clean Architecture + MVP, то Model у нас является код находящийся в слоях Data (работа с данными) и Domain (бизнес-логика). Следовательно в слое Presentation остаются лишь два компонента - View и Presenter.

Для более удобной связки View и Presenter мы будем использовать библиотеку [Moxy](https://github.com/Arello-Mobile/Moxy). Она помогает решить многие проблемы, связанные с жизненным циклом Activity или Fragment'а. Moxy имеет базовые классы, такие как ```MvpView``` и ```MvpPresenter``` от которых должны наследоваться наши View и Presenter. Для избежания написания большого количества кода по связыванию View и Presenter, Moxy использует кодогенерацию. Для правильной работы кодогенерации мы должны использовать специальные аннотации, которые предоставляет нам Moxy. Более подробную информацию о библиотеке можно найти [здесь](https://habrahabr.ru/post/276189/).

#### Model

Во многих статьях про MVP, Model описывается как слой отвечающий за бизнес-логику, а также за работу с данными, будь то база данных, файлы или запросы к серверу. В связке Clean Architecture + MVP, фактически остаются только слои View и Presenter. Роль Model берут на себя классы из слоев  **Domain** и **Data** в Clean Architecture.

#### View

View отвечает за то, каким образом данные будут показаны пользователю. Также View сообщает о действиях пользователя Presenter'у, будь то нажатие на кнопку или ввод текста. Пример View:

```java
public interface ArticlesListView extends MvpView {

    void showVisits(List<ArticleEntity> articles);
    void showArticlesLoadingErrorMessage();

}
```

Пока мы описали лишь интерфейс View, т. к. какие команды Presenter может отдавать View. Обратите внимание, что наш интерфейс наследуется от интерфейса **MvpView**, входящего в библиотеку Moxy. Это является обязательным условием для корректной работы библиотеки.

#### Presenter

Presenter является связующим звеном между View и Model. Presenter реагирует на действия пользователя, о которых ему сообщила View  (такие как нажатие на кнопку, пункт списка или ввод текста), после чего принимает решения о том, что делать дальше. Например, это может быть запрос данных у модели и отображение их во View. Пример Presenter'а:

```java
@InjectViewState
public class ArticlesListPresenter extends MvpPresenter<ArticlesListView> {

    private ArticlesListInteractor articlesListInteractor;

    @Inject
    public VisitsPresenter(ArticlesListInteractor articlesListInteractor) {
        this.articlesListInteractor = articlesListInteractor;
      
        loadArticles();
    }
  
    private void loadArticles() {
        articlesListInteractor.getArticles()
                .subscribe(articles -> getViewState().showArticles(articles),
                        throwable -> getViewState().showLoadingError());
    }
  
    public void onArticleSelected(Article article) {
        ...
    }  

}
```

Все необходимые классы для работы Presenter'а (как и всех остальных классов) мы передаем через конструктор. Этот способ так и называется - внедрение через конструктор.

При создании объекта Presenter'а мы должны передать ему запрашиваемые конструктором зависимости. Если их будет много, то создание Presenter'а будет довольно сложным делом. Чтобы не делать этого вручную, мы доверим это дело Component'у. 

```java
@Presenter
@Component(dependencies = ApplicationComponent.class)
public interface ArticlesListComponent {

    ArticlesListPresenter getPresenter();

}
```

Он подставит нужные зависимости, а нам нужно будет лишь получить инстанс Presenter'а вызвав  метод **getPresenter()**. Если у вас возник вопрос "А как в таком случае передавать  аргументы в Presenter?", то загляните в FAQ - там подробно описан этот вопрос.

Иногда можно встретить такое, что в конструктор передается DI-контейнер (Component), после чего все необходимые зависимости внедряются в поля:

```java
@Inject
ArticlesListInteractor articlesListInteractor;

public VisitsPresenter(ArticlesListPresenterComponent component) {
	component.inject(this);
}

```
Однако, данный способ является неправильным, т. к. усложняет тестирование класса и создает кучу ненужного кода. Если в первом случае мы сразу могли передать mock'и классов через конструктор, то теперь нам нужно создать DI-контейнер и передавать его. Также данный способ делает класс зависимым от конкретного DI-фреймворка, что тоже не есть хорошо.

#### Связывание View с Presenter'ом

[раздел на доработке]

В контексте разработки под Android роль View на себя берет Activity (или Fragment), поэтому после создания интерфейса View, мы должны реализовать его нашей Activity или Fragment'е:

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
    
  public void showArticles(List<ArticleEntity> articles) {
    ...
  }

  public void showLoadingError() {
    ...
  }

}
```

Хочу заметить, что для правильной работы библиотеки Moxy, наша Activity должна обязательно наследоваться от класса MvpAppCompatActivity (или MvpAppCompatFragment в случае, если вы используете фрагменты). С помощью аннотации ```@InjectPresenter``` мы сообщаем Annotation Processor'у в какую переменную нужно "положить" Presenter.

Так как конструктор нашего Presenter'а не пустой, а принимает на вход ```ArticlesListPresenterComponent```, то мы должны сами создать объект Presenter'а и предоставить его нашему генератору кода. Мы делаем это при помощи метода ```provideArticlesListPresenter```, который мы пометили аннотацией ```@ProvidePresenter```. Хочется отметить, что как и во всех других случаях использования кодогенерации, переменные и методы, помеченные аннотациями, должны быть видны на уровне пакета, т. е. у них не должно быть модификаторов видимости (private, public, protected).

### Разбиение классов по пакетам



Ниже представлен пример разбиения пакетов по фичам чат-приложения: 

```
com.mydomain
|
|----data
|     |---- database
|     |---- filesystem
|     |---- network
|     |     |---- ChatApiService
|     |---- repositories
|     |     |---- ChatRepositoryImpl
| 
|---- domain
|     |---- global
|     |     |---- models
|     |     |     |---- Chat
|     |     |     |---- Message
|     |     |     |---- User
|     |     |---- repositories
|     |     |     |---- ChatRepository
|     |---- chatdetails
|     |     |---- ChatDetailsInteractor
|     |---- chatslist
|     |     |---- ChatListInteractor
|
|---- presentation
|     |---- mvp
|     |     |---- global
|     |     |     |---- routing
|     |     |---- chatsdetails
|     |     |     |---- ChatDetailsPresenter
|     |     |     |---- ChatDetailsView
|     |     |---- chatslist
|     |     |     |---- ChatListPresenter
|     |     |     |---- ChatListView
|     |---- ui
|     |     |---- global
|     |     |     |---- views
|     |     |     |---- utils
|     |     |---- chatdetails
|     |     |     |---- ChatDetilsActivity
|     |     |---- chatslist
|     |     |     |---- ChatListActivity
|     
|---- di
|     |---- global
|     |     |---- modules
|     |     |---- scopes
|     |     |---- modifiers
|     |     |---- ApplicationComponent
|     |---- chatdetails
|     |     |---- ChatDetailsComponent
|     |     |---- ChatDetailsModule
|     |---- chatslist
|     |     |---- ChatListComponent
```
Прежде чем делить код по фичам, мы разделили его на слои. Данный подход позволяет сразу определить к какому слою относится тот или иной класс. Если вы заметили, классы слоя **data** разбиты немного не так, как в слоях **domain**, **presentation** и **di**. Здесь вместо фич приложения мы выделили типы источников данных - сеть, база данных, файловая система. Это связано с тем, что все фичи используют практически одни и те же классы (например, **ChatApiService**) и их не имеет смысла разбивать по фичам.

В пакетах с именем **global** хранятся общие классы, которые используются в нескольких фичах. Например, в пакете **data/global** хрянятся модели и интерфейсы репозиториев. 

Слой **presentation**  разбит на два пакета - **mvp** и **ui**. В **mvp** хранятся, как понятно из названия, классы Presenter'ов и View. В **ui** хранятся реализация слоя View из MVP, т. е. Activity, Fragment'ы и т. д. 

Разбиение классов по фичам имеет ряд преимуществ:

- **Очевидность:** Даже не знакомый с проектом разработчик, при первом взгляде на структуру пакетов сможет примерно понять что делает приложение, не заглядывая в сам код. 
- **Добавление нового функционала**.  Если вы решили добавить новую функцию в приложение, например, просмотр профиля пользователя, то вам лишь нужно добавить пакет **userprofle** и работать только с ним, а не "гулять" по всей структуре пакетов, создавая нужные классы.
- **Удобство редактирования.** При редактировании какой либо фичи, нужно держать открытыми максимум два-три пакета и вы видите только те классы, которые относятся к конкретной фиче. При разбиении по типу класса, раскрытой приходится держать практически все дерево пакетов и вы видите классы, которые вам сейчас не нужны, относящиеся к другим фичам.
- **Удобство масштабирования**. При увеличении количества функций приложения, увеличивается и количество классов. При разбиении классов по типу, добавление новых классов делает навигацию по ним очень не удобным, т.к. приходится искать нужный класс, среди десятков других, что сказывается на скорости и удосбстве разработки. Разбиение по фичам решает эту проблему, т.к. вы можете объединить связанные между собой пакеты с фичами (напрмер, можно объединить пакеты **login** и **registration** в пакет **authentication**).

Также хочется сказать пару слов об именовании пакетов: в каком числе их нужно называть - множественном или единственном? Я придерживаюсь подхода, описанного [здесь](https://softwareengineering.stackexchange.com/a/75929):

1) Если пакет содержит однородные классы, то имя пакета ставится во множественном числе. Например, пакет с классами **Dog**, **Cat** и **Cow** будет называться **animals**. Другой пример - различные реализации какого-либо интерфейса (**XmlResponseAdapter**, **JsonResponseAdapter**).
2) Если пакет содержит разнородные классы, реализующую определенную функцию, то имя пакета ставится в единственном числе. Пример - пакет **order**, содержащий классы **OrderInfo**,  **OrderInteractor**, **OrderValidation** и т. д.

### Взаимодействие между слоями (перенести наверх, засунуть в какой-нибудь раздел)

Для связки всех трех слоев используется Dependency Injection. В нашем примере мы используем библиотеку Dagger 2.

### Дополнительные сущности, используемые на практике



#### Router

Т. к. Presenter содержит в себе логику реагирования на действия пользователя, то он также знает о том, на какой экран нужно перейти. Однако сам Presenter не может осуществлять переход на новый экран, т. к. для этого нам требуется Context. Поэтому за открытие нового экрана должна отвечать View. Для осуществления перехода на следующий экран мы должны вызвать метод View, например, **openProfileScreen()**, а уже в реализации самого метода осуществлять переход. Помимо данного подхода некоторые разработчики используют для навигации так называемый Router.

**Router** - класс, для осуществления переходов между экранами (активити или фрагментами).

Для реализации Router'а вы можете использовать библиотеку [Alligator](https://github.com/aartikov/Alligator)

#### Mapper

Mapper - специальный класс, для конвертирования моделей из одного типа в другой, например, из модели БД в модель бизнес-логики (Entity). Обычно они имеют название типа XxxMapper, и имеют единственный метод с названием map (иногда встречаются названия convert/transform), например:

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
    public VisitsPresenter(Context context) {
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
    public ShotsPresenter(...,  AndroidResourceManager resourceManager) {
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

## Обработка ошибок

[раздел на доработке]

## Тестирование

Одним из самых главных преимуществ является то, что мы можем покрыть тестами намного больший функционал приложения, за счет разбиения кода на мелкие классы, каждый из которых выполняет строго определенную задачу. Благодаря принципу инверсии зависимостей, используемому в чистой архитектуре мы можем с легкостью подменять реализацию тех или иных классов на фейковые, которые реализуют нужное нам поведение.


Прежде чем начать писать тесты, мы должны ответить себе на два вопроса:

- что мы хотим тестировать?
- как мы будем это тестировать?

Что мы хотим тестировать?

- Мы хотим проверить нашу бизнес-логику независимо от какого-либо фреймворка или библиотеки.
- Мы хотим протестировать нашу интеграцию с API.
- Мы хотим протестировать нашу интеграцию с нашей системой персистентности.
- Мы хотим протестировать некоторые общие компоненты пользовательского интерфейса.
- Мы хотим проверить критерии приемлемости, написанные с точки зрения пользователя, в сценарии полностью черного ящика.

### Тестирование слоя представления

Данный слой включает в себя 2 типа тестов: UI-тесты и unit-тесты.

- UI-тесты используются для тестирования Activity (проверяется корректность отображения элементов и т. д.). Для реализации этих тестов я рекомендую использовать фреймворк Espresso.
- unit-тесты используются для тестирования Presenter'ов.

### Тестирование бизнес-логики

В данном слое тестируюится классы Interactorov. Необходимо проверить, действительно ли бизнес-логика реализует предопределенные требования к продукту.

[раздел на доработке]

## Тестирование слой работы с данными



## Начало разработки приложения с использованием Clean Architecture

Если вы начиначете разработку нового мобильного приложения, то лучше начать с создания пользовательского интерфейса, т. к. именно UI определяет 

[раздел на доработке]

## Перенос на Clean Architecture существующих проектов

Текст-введение

#### Шаг 1: Разбиение Activity на View и Presenter



#### Шаг 2: Отеделяем Model от Presenter
### Шаг 3: Разделяем Model на DataStore'ы (database, network)
#### Шаг 4: 
#### Шаг 5: Выносим бизнес-логику в UseCase'ы и Interactor'ы

[раздел на доработке]

## FAQ по Clean Architecture

#### Стоит ли переписывать весь проект при переносе проекта на Clean Architecture?

Наверное, нет однозначного ответа на этот вопрос. Если проект большой и переход на Clean Architecture может длительный промежуток времени, то лучше переписывать код постепенно, используя подход, который мы описали выше. Если же проект простой и состоит из 2-3 экранов, а сроки не поджимают, то вы можете попробовать переписать проект с нуля.

В конце хочется привести поучительную историю про Netscape, который переписывали с нуля больше, чем три года - [Things You Should Never Do, Part I](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/)

#### Обязательно ли создавать отдельные сущности для каждого из слоев (Domain, Data, Presentaion) или же можно использовать Entity во всех трех слоях?

Согласно принципам Clean Architecture, слой Domain ничего не должен знать о внешних слоях (Data и Presentation), но внешние слои без проблем могут использовать классы из слоя Domain. Следовательно, можно не создавать отдельные сущности для каждого из слоев, а использовать Entity. Однако, если формат Entity не совпадает с тем, что используется во внешних слоях, то нужно создать отдельную сущность. Также не следует использовать в Entity аннотации, которые требуются библиотекам, типа [Gson](https://github.com/google/gson) или [Room](https://developer.android.com/topic/libraries/architecture/room.html). В этом случае нужно создать отдельную сущность в слое Data.

[раздел на доработке]

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

- Если мы хотим добавить новый метод или изменить существующий, нам нужно изменить интерфейс. Помимо это мы также должны изменить реализацию метода. Это занимает довольно времени, даже при использовании такой продвинутой IDE как Android Studio.
- Использование дополнительных интерфейсов усложняет навигацию по коду. Если вы хотите перейти к реализации метода Presenter'а из Activity (т. е. реализации View), то вы переходите к интерфейсу Presenter'а.
- Интерфейс никак не улучшает тестируемость кода. Вы с легкостью можете заменить класс Presenter'а на mock, используя любую библиотеку для mock'ирования.

Более подробно можете почитать об этом в следующих статьях:

- [Interfaces for presenters in MVP are a waste of time!](http://blog.karumi.com/interfaces-for-presenters-in-mvp-are-a-waste-of-time)
- [Франкен-код или франкен-дизайн](https://plus.google.com/u/0/+SergeyTeplyakov/posts/gRaqrqaiGbe)
- [Управление зависимостями](http://sergeyteplyakov.blogspot.ru/2012/11/blog-post.html)

#### Как передать аргументы в Presenter, если его инстанс создает DI-контейнер?

Часто при создании презентера возникает необходимость передать дополнительные аргументы. Например, мы хотим передать идентфикатор статьи, чтобы получить её содержимое от сервера. Чтобы сделать это, нам необходимо создать отдельный модуль для Presenter'а и передать аргументы туда:

``` java
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