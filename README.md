# Composabe C++ futures (Cf) library
This is an implementation of composable, continuing c++17 like [futures](http://en.cppreference.com/w/cpp/experimental/future). I'm done with the most useful/interesting parts as I personally see it. Some other, currently not implemented features will come soon I hope, while the rest (like void future/promise specializations) are not likely to ever emerge to life.

Cf library consists of just one header with no dependencies except of c++14 compliant standard library.

The most significant Cf difference from standard futures is the Executor concept. Executor may be an object of virtually any type which has `post(std::function<void()>)` member function. It enables continuations and callables passed to the `cf::async` be executed via separate thread/process/coroutine/etc execution context.
Cf comes with three executors shipped. They are: 
* `cf::sync_executor` - executes callable in place. This is just for the generic code convinience.
* `cf::async_queued_executor` - non blocking async queued executor. May be used as a base of an event loop.
* `cf::async_thread_pool_executor` - almost same as above, except posted callables may be executed on one of the free worker thread.

## Cf current state
|Feature name|Standard library (including c++17)|CF   |Compliance|
|------------|:--------------------------------:|:---:|----------|
|[future](http://en.cppreference.com/w/cpp/experimental/future)|Yes|Yes|No share() member function. No void (use cf::unit instead) and T& specializations.|
|[promise](http://en.cppreference.com/w/cpp/thread/promise)|Yes|Yes|No set_\*\*_at_thread_exit member functions. No void ans T& specializations.|
|[async](http://en.cppreference.com/w/cpp/thread/async)|Yes|Yes|No launch policy.|
|[packaged_task](http://en.cppreference.com/w/cpp/thread/packaged_task)|Yes|No||
|[shared_future](http://en.cppreference.com/w/cpp/thread/shared_future)|Yes|No||
|[when_all](http://en.cppreference.com/w/cpp/experimental/when_all)|Yes|Yes||
|[when_any](http://en.cppreference.com/w/cpp/experimental/when_any)|Yes|Yes||
|[make_ready_future](http://en.cppreference.com/w/cpp/experimental/make_ready_future)|Yes|Yes||
|[make_exceptional_future](http://en.cppreference.com/w/cpp/experimental/make_exceptional_future)|Yes|Yes||

## Examples
For the basic future/promise/async examples please refer to http://en.cppreference.com/w/cpp/thread#Futures.
### Async && Then
```c++
cf::async_queued_executor executor;
auto f = cf::async([] {
  std::this_thread::sleep_for(std::chrono::milliseconds(10)); // This is executed on the separate standalone thread
  return std::string("Hello ");                               // Result, when it's ready, is stored in cf::future<std::string>
}).then([] (cf::future<std::string> f) {                      // Which in turn is passed to the continuation.
  std::this_thread::sleep_for(std::chrono::milliseconds(10)); // The continuation may be executed on different contexts.
  return f.get() + "futures ";                                // This time - on the same thread as async.
}).then([] (cf::future<std::string> f) {
  std::this_thread::sleep_for(std::chrono::milliseconds(10)); // And this time on the async_queued_executor context.
  return f.get() + "world!";
}, executor);

assert(f.get() == "Hello futures world!");
```
