---
title: "How to test HttpClient in Angular"
excerpt_separator: "<!--more-->"
categories:
  - Knowledgebase
tags:
  - Angular
  - HttpClientModule
  - HttpClientTestingModule
---

This is a simple method to call http using the HttpClientModule. 

```
getBooks(): Observable<Book[]>{
    return this.http.get<Book[]>(BooksService.booksUrl);
}
```

To test it, first we need to ensure the HttpClientTestingModule is in the imports when we configure the testing module. 

```
beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule]
    });

    service = TestBed.inject(BooksService);
    httpClient = TestBed.inject(HttpClient);
    httpTestController = TestBed.inject(HttpTestingController);
});
```

The `TestBed.inject(<ClassName>)` is how we can get the injected instances from our module. The `httpClient` is the same as from the `HttpClientModule`, but a new `HttpTestingController` is introduced in the `HttpClientTestingModule` to mock the receiving end of the http call. 

A unit test for the above `getBooks` function will be as such

```
it('should be able to get books', () => {
    service.getBooks().subscribe(books => {
      expect(books.length).toBe(5);
    });

    const req = httpTestController.expectOne(BooksService.booksUrl);
    expect(req.request.method).toEqual('GET');
    req.flush(books);
});
```

First, the `getBooks` method is called and subscribed to. Then, we call our `httpTestController` to expect a single call from the specific url which `getBooks` is requesting from. We can then add the assertion that the request method should be a `GET` - `expect(req.request.method).toEqual('GET');`. Last but not least, we flush our desired mock result so that it can be picked up by the prior subscription of the `getBooks` method.

Last of all, we need to make sure `httpTestController.verify()` is called after each test to state that there shouldn't be anymore outstanding requests. We do this by having the call in the `afterEach()` function.

```
afterEach(() => {
    httpTestController.verify();
})
```

A working example of the [service](https://github.com/thecodinganalyst/bookstore/blob/master/src/app/books.service.ts) and [unit test](https://github.com/thecodinganalyst/bookstore/blob/master/src/app/books.service.spec.ts) is available on my [repository](https://github.com/thecodinganalyst/bookstore). 
