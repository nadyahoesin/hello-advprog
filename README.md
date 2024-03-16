# Reflection

### Milestone 1: Single threaded web server

This milestone is for successfully building a single threaded web server that clients can connect to but isn't yet sending any data back. We did this by firstly creating an instance of [`TcpListener`](https://doc.rust-lang.org/std/net/struct.TcpListener.html) that is bound to the local address `127.0.0.1:7878`. The method `incoming` on the listener returns an iterator over incoming TCP streams. Each streams represent a connection between the client and the server. We iterate over these streams to call our `handle_connection` function on them. 

The `handle_connection` function will first create a [`BufReader`](https://doc.rust-lang.org/std/io/struct.BufReader.html) instance from the input stream. `BufReader` is a struct that creates a buffer to any reader. The reason it's preferred to use a buffer rather than directly working with the reader is because every call to read on `TcpStream` results in a system call, which could be excessively inefficient. We then use a series of method to convert this buffer into an iterator over the lines in the buffer (`lines`), map those lines to their values rather than in `Result` structs (`map`), and take lines until we get a line that is the empty string (`take_while`). Finally, we print these lines out using debug formatting to examine them.

<br>

### Milestone 2: Returning HTML

TBA

<br>

### Milestone 3: Validating request and selectively responding

TBA

<br>

### Milestone 4: Simulation slow response

TBA

<br>

### Milestone 5: Multithreaded server

TBA

<br>

### Bonus: Function improvement

TBA
