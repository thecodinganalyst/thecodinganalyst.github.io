---
title: "How to test NgRx"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Angular
  - NgRx
  - Effects
---

![NgRx Testing](/assets/images/2022/06/ngrx-testing.png)

Ngrx is a framework for angular that helps to separate the backend logic from the frontend, making the code easier cleaner. To know more about how it works and how to use it, do check out my previous article - [NgRx explained](https://thecodinganalyst.github.io/knowledgebase/ngrx-explained/). Testing with NgRx is pretty straightforward, you'll just need to use the mocks provided.

Firstly, as the `store` is injected in the constructors, and since unit tests are supposed to be isolated, therefore we'll need to add the `provideMockStore()` into the `providers` array when we configure the testing module. Then we can create a reference to our `store`, declaring as type `MockStore<AppState>`, and get it's value injected from the `TestBed`. 

```
let store: MockStore<AppState>;

beforeEach(() => {
  TestBed.configureTestingModule({
    declarations: [AppComponent, ItemComponent],
    imports: [ReactiveFormsModule],
    providers: [provideMockStore()],
    schemas: [CUSTOM_ELEMENTS_SCHEMA]
  });
  fixture = TestBed.createComponent(AppComponent);
  app = fixture.componentInstance;
  store = TestBed.inject(MockStore);
});
```
<!--more-->

To stub the values from the `selectors`, we can use the `overrideSelector(<Selector>, <Value>)` method and `fixture.detectChanges()` to refresh the data.

```
store.overrideSelector(Selector.books, sampleBooks);
fixture.detectChanges()
```

If we need to reuse the overriden selector, we can assign it to a variable like `let showDetailSelector = store.overrideSelector(Selector.showDetail, true);`. We can update the values by using the `setResult(<Value>)` method. But we'll need to call `store.refreshState()` after that to refresh the store.

```
showDetailSelector.setResult(false);
store.refreshState();
fixture.detectChanges();
```

A sample of how all these can be done are available on [https://github.com/thecodinganalyst/bookstore/blob/master/src/app/app.component.spec.ts](https://github.com/thecodinganalyst/bookstore/blob/master/src/app/app.component.spec.ts).

## Testing Selectors

Supposing we have a selector as such

```
const books = createSelector( bookStore, (bookStoreState => bookStoreState.books));
```

To test our selectors, we can use the `projector(<State>)` method of our selector to get the expected value. 

```
it('should get the books', () => {
  const result = Selector.books.projector(initialState.bookStore);
  expect(result.length).toBe(5);
});
```

A full example is on [https://github.com/thecodinganalyst/bookstore/blob/master/src/app/state/books.selectors.spec.ts](https://github.com/thecodinganalyst/bookstore/blob/master/src/app/state/books.selectors.spec.ts).

## Testing Reducers

As `reducers` are simply pure functions to get a new state from a current state with a specific action, we just need to stub our state and create an action object to pass to our reducer. 

Supposing we have an action as such

```
const booksLoaded = createAction("[BookList] Books Loaded", props<{ books: ReadonlyArray<Book> }>())
```

We can directly reference the `booksLoaded` action and pass in the books parameter - `const action = BookStoreActions.booksLoaded({books: [sampleBook]});`. Then we can pipe it to our reducer by - `const state = booksReducer(initialState, action);`, and do assertions directly on the state object.

A full example is on [https://github.com/thecodinganalyst/bookstore/blob/master/src/app/state/books.reducer.spec.ts](https://github.com/thecodinganalyst/bookstore/blob/master/src/app/state/books.reducer.spec.ts).

## Testing Effects

We'll need to add the `effects` as it is, and `action` and `store` as mocks.

```
beforeEach(() => {
  TestBed.configureTestingModule({
    providers: [
      BooksEffects,
      provideMockActions(() => actions$),
      provideMockStore({initialState}),
    ],
    imports: [HttpClientModule]
  })
  booksService = TestBed.inject(BooksService)
  effects = TestBed.inject(BooksEffects)
  store = TestBed.inject(Store)
})
``` 

For an action - `loadBooks`, a side effect of the action is to call the service to load the books from the database. And after the books are loaded, another action - `booksLoaded` is triggered with the books as a payload. 

```
loadBooks$ = createEffect(() => this.actions$.pipe(
  ofType(BookStoreActions.loadBooks),
  mergeMap(() => this.booksService.getBooks().pipe(
    map(books => BookStoreActions.booksLoaded({books})),
    catchError(() => EMPTY)
  ))
));
```

To test the above effect, 
1. first we put a spy on the service to return a mock list of books, so that the workings of the service do not interfere with our unit test. 
2. Then we trigger the action - `actions$ = of(BookStoreActions.loadBooks);`. 
3. Then we subscribe to the effect, and put our assertions

```
describe('getBooks action', function () {
  it("should call getBooks and redirect to booksLoaded action", (done) => {
    spyOn(booksService, "getBooks").and.returnValue(of(books))
    actions$ = of(BookStoreActions.loadBooks);
    effects.loadBooks$.subscribe(res => {
      expect(booksService.getBooks).toHaveBeenCalled()
      expect(res).toEqual(BookStoreActions.booksLoaded({books: books}))
      done()
    })
  })
});
```

A full example is available on [https://github.com/thecodinganalyst/bookstore/blob/master/src/app/state/books.effects.spec.ts](https://github.com/thecodinganalyst/bookstore/blob/master/src/app/state/books.effects.spec.ts).

