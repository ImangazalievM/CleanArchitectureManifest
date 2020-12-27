# Testing

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

- Third-party libraries (we assume that they work correctly because they have already been tested by the developers)
- Trivial code (for example, getters and setters)

Let's take a closer look at how we will test each of layers.

## Presentation layer testing

This layer includes 2 types of tests: Unit-tests and UI-tests.

- Unit-tests are used for testing Presenters.
- UI-tests are used for testing Activities (to check if UI works correctly)

There are different naming conventions for unit tests. For example, this](https://dzone.com/articles/7-popular-unit-test-naming) article describes some of them. In my examples of tests, I will not adhere to an agreement. The most important thing is to understand what the tests are doing and what we want to get as a result.

Let us take the example of a test for **ArticlesListPresenter**:

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

As you can see, we divided the test code into three parts:

- Preparation for testing. Here we initialize the objects for testing, prepare the test data, and also define the behavior of the mocks.
- Testing itself
- Checking the test results. Here we check the View methods have been called with necessary arguments.

## Domain layer testing

[this chapter is in preparation]

## Data layer testing

[this chapter is in preparation]