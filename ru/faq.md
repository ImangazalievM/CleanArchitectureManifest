# FAQ по Clean Architecture

### Стоит ли переписывать весь проект при переносе проекта на Clean Architecture?

Наверное, нет однозначного ответа на этот вопрос. Если проект большой и переход на Clean Architecture может занять длительный промежуток времени, то лучше переписывать код постепенно, используя подход, который мы описали выше. Если же проект простой и состоит из 2-3 экранов, а сроки не поджимают, то вы можете попробовать переписать проект с нуля.

В конце хочется привести поучительную историю про Netscape, который переписывали с нуля больше, чем три года - [Things You Should Never Do, Part I](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/)

### Обязательно ли создавать отдельные сущности для каждого из слоев (Domain, Data, Presentaion)?

Согласно принципам Clean Architecture, слой Domain ничего не должен знать о внешних слоях (Data и Presentation), но внешние слои без проблем могут использовать классы из слоя Domain. Следовательно, можно не создавать отдельные сущности для каждого из слоев, а использовать только те, что лежат в слое Domain. Однако, если их формат не совпадает с тем, что используется во внешних слоях, то нужно создать отдельную сущность. Если вам нужно добавить к моделям аннотации от библиотек (например, от [Gson](https://github.com/google/gson) или [Room](https://developer.android.com/topic/libraries/architecture/room.html)), то вы можете добавить их прямо в модели Domain-слоя, несмотря на то, что это внешние библиотеки слоя Data, т. к. создание отдельных моделей приведёт к ненужному дублированию кода.

### Нужно ли создавать интерфейсы для классов Presenter и Interactor для улучшения тестируемости кода?

Пример Presenter'а с интерфейсом:

```kotlin
interface LoginPresenter {

	fun onLoginButtonPressed(email: String, password: String)
  
}

class LoginPresenterImpl : LoginPresenter {
	...
}
```

Нет, интерфейсы для презентера и интерактора создавать не нужно. Это создает дополнительные сложности при разработке, при этом пользы от данного подхода практически нет. Вот лишь некоторые проблемы, которые порождает создание лишних интерфейсов:

- Если мы хотим добавить новый метод или изменить существующий, нам нужно изменить интерфейс. Помимо этого мы также должны изменить реализацию метода. Это занимает довольно времени, даже при использовании такой продвинутой IDE как Android Studio.
- Использование дополнительных интерфейсов усложняет навигацию по коду. Если вы хотите перейти к реализации метода Presenter'а из Activity (т. е. реализации View), то вы переходите к интерфейсу Presenter'а.
- Интерфейс никак не улучшает тестируемость кода. Вы с легкостью можете заменить класс Presenter'а на mock, используя любую библиотеку для mock'ирования.

Более подробно можете почитать об этом в следующих статьях:

- [Interfaces for presenters in MVP are a waste of time!](http://blog.karumi.com/interfaces-for-presenters-in-mvp-are-a-waste-of-time)
- [Франкен-код или франкен-дизайн](https://plus.google.com/u/0/+SergeyTeplyakov/posts/gRaqrqaiGbe)
- [Управление зависимостями](http://sergeyteplyakov.blogspot.ru/2012/11/blog-post.html)

### Как передать аргументы в Presenter, если его инстанс создает DI-контейнер?

Часто при создании презентера возникает необходимость передать дополнительные аргументы. Например, мы хотим передать идентфикатор статьи, чтобы получить её содержимое от сервера. Чтобы сделать это, нам необходимо создать отдельный модуль для Presenter'а и передать аргументы туда:

```kotlin
@Module
class ArticleDetailsModule(val articleId: Long) {

    @Provides
    @Presenter
    fun provideArticleId() = articleId

}
```

Далее нам нужно добавить наш модуль в Component:

```kotlin
@Component(dependencies = [ApplicationComponent::class], modules = [ArticleDetailsModule::class])
interface ArticleDetailsComponent {
```

При создании Component'а мы должны передать наш модуль с идентификатором:

```kotlin
val component = DaggerArticleDetailsComponent.builder()
    .applicationComponent(MyApplication.getComponent())
    .articleDetailsModule(ArticleDetailsModule(articleId))
    .build();
```

Теперь мы можем получить наш идентификатор через конструктор:

```kotlin
class UserFollowersPresenter @Inject constructor(
        private val interactor: ArticleDetailsInteractor,
        private val articleId: Long
)
```

Теперь представим, что помимо идентфикатора статьи, мы хотим передать ID пользователя, который так же имеет тип long. Если мы попытаемся создать ещё один provide-метод в нашем модуле, Dagger выдаст ошибку, о том, что типы совпадают и он не знает какой из них является идентфикатором статьи, а какой идентфикатором пользователя. 

Чтобы исправить это, нам необходимо создать Qualifier-аннотации,  которые будут указывать Dagger'у "who is who":

```kotlin
@Qualifier
@Retention(AnnotationRetention.RUNTIME)
annotation class ArticleId

@Qualifier
@Retention(AnnotationRetention.RUNTIME)
annotation class UserId
```

Добавляем аннотации к нашим provide-методам:

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

Также нужно пометить аннотациями аргументы конструктора:

```kotlin
class UserFollowersPresenter @Inject constructor(
        private val interactor: ArticleDetailsInteractor,
        @ArticleId private val articleId: Long,
        @UserId private val userId: Long
)
```

Готово. Теперь Dagger сможет верно расставить аргументы в конструктор.