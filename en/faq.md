# Clean Architecture FAQ

### Should I rewrite all project when I migrate to Clean Architecture

There is no definite answer to this question. If your project is big enough and migration to Clean Architecture take a lot of time, the best bet is to rewrite the project gradually, using the approach that we have described above. But you can try to rewrite the project from scratch if your project is small (contains 2-3 screens) and you have enough time.

Certainly a cautionary tale about developers of Netscape, who decided to rewrite the code from scratch - [Things You Should Never Do, Part I](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i).

### Is it necessary to create separate models for each layer (Domain, Data, Presentation)?

According to the principles of Clean Architecture, Domain layer should know nothing about outer layers (Data and Presentation), but outer layer can use classes from Domain layer. Consequently, yoг don't have to create separate models for each layer. However, if models for each layer are different, you must to create different classes. If you need to add annotations from libraries (for example, from Gson or Room) to models (without changing model structure), then you can add them directly to Domain-layer models, even though these are external libraries of Data layer, because creating separate models will be unnecessary duplication of code.

### Do we need to create interfaces for Presenters and Interactors to improve code testability?

This is an example of Presenter with interface:
```kotlin
interface LoginPresenter {

	fun onLoginButtonPressed(email: String, password: String)
  
}

class LoginPresenterImpl : LoginPresenter {
	...
}
```

No, you don't need to create interface for Presenters or Interactors, because it creates additional problems and has no advantages. There are some of the problems created by using redundant interfaces:

- If we want to add new method or change existing one, we need to change the interface. Also, we need to change implementation of the interface. It takes quite some time, even using such a powerful IDE as Android Studio.
- Using of additional interfaces makes code navigation more difficult. For example, if you want to open an implementation of a Presenter method from Activity, then you go to a method definition in the interface.
- The interface doesn't improve code testability. You can easily replace Presenter's implementation to it's mock, using any mocking library.

You can read more about this topic in the following articles:

- [Interfaces for presenters in MVP are a waste of time!](http://blog.karumi.com/interfaces-for-presenters-in-mvp-are-a-waste-of-time)

### How to pass arguments into Presenter, when it's instance created via DI-container?

Often when creating a presenter, we want to pass arguments to constructor For example, we want to pass an article ID to get its content from the server. To do this, we need to create a separate module for Presenter and pass the arguments there:

```kotlin
@Module
class ArticleDetailsModule(val articleId: Long) {

    @Provides
    @Presenter
    fun provideArticleId() = articleId

}
```

Next we need to add this module to the Component:
```kotlin
@Component(dependencies = [ApplicationComponent::class], modules = [ArticleDetailsModule::class])
interface ArticleDetailsComponent {
```

When creating the Component we provide our module with arguments:
```kotlin
val component = DaggerArticleDetailsComponent.builder()
    .applicationComponent(MyApplication.getComponent())
    .articleDetailsModule(ArticleDetailsModule(articleId))
    .build();
```

Now we can get our identifier through constructor:
```kotlin
class UserFollowersPresenter @Inject constructor(
        private val interactor: ArticleDetailsInteractor,
        private val articleId: Long
)
```

Let's suppose that in addition an article ID, we want to pass an user ID, that also has type "long". If we try to create another provide-method in our module, Dagger will give an error message, because it doesn't know which method provides an user ID or an article ID.

To fix this, we need to create **Qualifier** annotations that will tell Dagger "who is who":

```kotlin
@Qualifier
@Retention(AnnotationRetention.RUNTIME)
annotation class ArticleId

@Qualifier
@Retention(AnnotationRetention.RUNTIME)
annotation class UserId
```

Add annotations to our provide-methods:

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

Also, we need to mark the constructor arguments with annotations:

```kotlin
class UserFollowersPresenter @Inject constructor(
        private val interactor: ArticleDetailsInteractor,
        @ArticleId private val articleId: Long,
        @UserId private val userId: Long
)
```

Done. Now Dagger can pass the arguments correctly.