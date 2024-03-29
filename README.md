# Reflection

### Milestone 1: Single-threaded web server

This milestone is for successfully building a single-threaded web server that clients can connect to but isn't yet sending any data back. We did this by firstly creating an instance of [`TcpListener`](https://doc.rust-lang.org/std/net/struct.TcpListener.html) that is bound to the local address `127.0.0.1:7878`. The method `incoming` on the listener returns an iterator over incoming TCP streams. Each streams represent a connection between the client and the server. We iterate over these streams to call our `handle_connection` function on them. 

The `handle_connection` function will first create a [`BufReader`](https://doc.rust-lang.org/std/io/struct.BufReader.html) instance from the input stream. `BufReader` is a struct that creates a buffer to any reader. The reason it's preferred to use a buffer rather than directly working with the reader is because every call to read on `TcpStream` results in a system call, which could be excessively inefficient. We then use a series of method to convert this buffer into an iterator over the lines in the buffer (`lines`), map those lines to their values rather than in `Result` structs (`map`), and take lines until we get a line that is the empty string (`take_while`). Finally, we print these lines out using debug formatting to examine them.

<br>

### Milestone 2: Returning HTML

This milestone is for successfully returning an HTML file to clients who connected to the web server. We did this by adding code in the `handle_connection` function to write a response to the stream. In order to be valid, the response must consist of status line (which in our case is `HTTP/1.1 200 OK`), `Content-Length` header, and the contents. The contents are from a simple HTML file we created called `hello.html`, attached below when rendered by the browser. This HTML file is written into the `contents` string by the `fs::read-to_string` function. The response is first converted into bytes by `as_bytes` method, before being written to the stream by `write_all` method. 

![commit 2 screenshot](<img/commit2.png>)

<br>

### Milestone 3: Validating request and selectively responding

This milestone is for successfully validating requests and returning different responses based on the validity of the requests. We validate requests in the `handle_connection` function by examining the first line of each request. If it equals `GET / HTTP/1.1`, then it is a valid request. Clients trying to access other path (e.g. `127.0.0.1:7878/hello`) would send requests with different first line than above, which will be regarded as invalid requests. We handle valid requests the same way as before. For invalid request, we write a response to the stream with `HTTP/1.1 404 NOT FOUND` status line and contents from `404.html`, attached below when rendered by the browser. Initially, we did all this using similar code in two if blocks, only differing in status line and contents' filename. We then refactored the code to avoid repetitions by only using the if-clause to determine the status line and contents' filename, then executing the same code for both scenario (valid and invalid request).

![commit 3 screenshot](<img/commit3.png>)

<br>

### Milestone 4: Simulating slow response

This milestone is for successfully simulating slow responses due to a slow-processing request. Because the server is single-threaded, all requests are handled sequentially, one by one. One slow-processing request could affect other requests by delaying the handling of those requests. This slow-processing request is simulated by the path `127.0.0.1:7878/sleep`, which would cause the server to sleep for 5 seconds before responding. It's easy to imagine how multiple requests, especially if they made the same slow-processing request, would cause an excessively long delay in response time. This scenario shows the need for multithreading, which would make the application handle multiple requests simultaneously.  

<br>

### Milestone 5: Multithreaded server

This milestone is for successfully turning the single-threaded server into a multi-threaded server. We did this by implementing a new struct `ThreadPool`. `ThreadPool` is a pool of a small number of spawned threads that are waiting and ready to handle a task. It has two methods, `new` and `execute`. `new` function creates a new instance of `ThreadPool` with the inputted amount of threads and return that instance. It also creates an `mpsc` channel with itself as a sender and every spawned threads as receivers. Spawned threads are wrapped in a newly implemented struct `Worker`, which stores the id of the thread and the thread itself. `execute` function sends incoming tasks to the `mpsc` channel that one of the `workers`, or threads, can accept. 

We modified the main function to first create a new instance of `ThreadPool` with 4 threads and for each incoming connection, the task of handling connection done by the `handle_connection` function is executed by one of the waiting threads.

<br>

### Bonus: Function improvement

This milestone is for successfully changing the `new` method that returns an instance of `ThreadPool` with the `build` method that returns an instance of `Result` that contains either a `ThreadPool` instance or a `PoolCreationError` error. Returning a `Result` instance allows the method to not have an assertion to make sure the inputted size is not 0, but rather return an error when the inputted size is 0. Changing the method name from `new` to `build` follows Rust convention, as `new` methods normally return the instances of the struct and `build` methods return `Result` instances. Because of this, we also modified the main function to use `build` method and unwrap the return to get the `ThreadPool` instance. The advantage of using a `build` method instead of the `new` method is for the struct user to be able to decide what to do in the case of an error, whether to ignore it (which we did when we use the method `unwrap` on the result) or display an error message. Assertions are generally avoided in struct methods.
