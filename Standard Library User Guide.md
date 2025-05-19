# Using the D++ Standard Library

This guide teaches you how to perform common programming tasks using the basic modules and structures provided by the D++ Standard Library (`std`). We assume you have read the basic D++ syntax guide and understand the concepts of ownership and borrowing before reading this guide.

The D++ Standard Library is organized into modules for common programming needs: `core`, `io`, `fs`, `collections`, `math`, `net`, `thread`, `time`, and more.

## 1. Accessing the Standard Library (`use`)

To access items in the `std` module, you can use the `std::` prefix or bring items or modules into the current scope with the `use` keyword.

Some basic items from the `core` module (like `Option`, `Result`) might be accessible at all times (like D's `object` module), but this depends on the D++ design. Explicit `use` statements generally make your code more readable.

```dplusplus
// Access using the std:: prefix
fn example_std_prefix() -> std::io::Stdout {
    std::io::stdout()
}

// Import a specific item with use
use std::collections::Vec;

fn example_use_vec() -> Vec<int> {
    let mut v = Vec::new();
    v.push(1);
    v // returns Vec<int>
}

// Import a module with use (the module name is used as a prefix)
use std::io;

fn example_use_module() -> void {
    let stdout = io::stdout();
    // ... use stdout ...
}

// Import multiple items with use
use std::math::{sin, cos, PI};

fn example_use_multiple() -> void {
    let angle = PI / 2.0;
    println!("sin(PI/2) = {}", sin(angle));
    println!("cos(PI/2) = {}", cos(angle));
}

// Renaming imported items with use
use std::io::Error as IoError;

fn example_use_as() -> Result<(), IoError> {
    // ... use IoError ...
    Ok(())
}

// Importing all public items with use (generally not recommended)
// use std::collections::*; // Now Vec, HashMap can be used directly

fn main() -> void {
    let v = example_use_vec();
    println!("Vector length: {}", v.len());

    let stdout = example_std_prefix(); // or io::stdout();
    // ...
}
use statements are usually placed at the beginning of the file or module.

2. Core Types and Traits (std::core)
The std::core module (or the core of the language) contains fundamental structures and traits like Option, Result, String, Copy, Clone, Debug, Display. These items are used in almost every D++ program.

Using Option&lt;T>
Represents a value that may or may not be present. Used with the match expression.

Kod snippet'i

use std::core::Option::{Some, None}; // Or just use std::core::Option;

fn find_first_even(numbers: &[int]) -> Option<int> {
    for &num in numbers {
        if num % 2 == 0 {
            return Some(num); // Even number found
        }
    }
    None // No even number found
}

fn main() -> void {
    let my_numbers = [1, 3, 5, 6, 8, 9];
    let first_even = find_first_even(&my_numbers);

    match first_even {
        Some(value) => println!("First even number: {}", value),
        None => println!("No even number in the array."),
    }

    let empty_numbers: [int; 0] = []; // Empty array
    let no_even = find_first_even(&empty_numbers); // Passed as &[]

    // if let (If D++ supports it)
    if let Some(value) = no_even {
        println!("Even number found with if let: {}", value);
    } else {
        println!("if let: No even number in the array.");
    }
}
Using Result&lt;T, E>
Represents an operation that can either succeed (Ok) or fail (Err). Used for error propagation and typically handled with match.

Kod snippet'i

use std::core::Result::{Ok, Err};
use std::io::Error as IoError; // We will use io::Error as the error type

// An operation that might fail, like reading a file
fn read_config(path: &str) -> Result<std::core::String, IoError> {
    // We are hypothetically using the fs module here
    // use std::fs; // Assume it's also used in main

    let content = std::fs::read_to_string(path)?; // The '?' operator propagates the error
    // If read_to_string returns Ok(s), s is assigned to the 'content' variable, and the code continues.
    // If read_to_string returns Err(e), the function immediately returns that Err(e).

    // Other operations...
    // For example, process the content and create an error condition
    if content.is_empty() { // Assume is_empty() is a String method
        return Err(IoError::InvalidData); // Return an invalid data error
    }

    Ok(content) // Success, return the content inside Ok
}

fn main() -> void {
    let config_result = read_config("config.txt"); // Needs an actual config.txt file to exist

    match config_result {
        Ok(config_str) => {
            println!("Configuration successfully read:");
            println!("{}", config_str);
            // Use the configuration string...
        },
        Err(error) => {
            println!("Configuration read error: {}", error.description()); // Use the Error trait
            // Different actions can be taken based on the error type:
            // match error {
            //     IoError::NotFound => println!("Error: config.txt not found."),
            //     IoError::PermissionDenied => println!("Error: File permission denied."),
            //     _ => println!("Other I/O error: {}", error.description()),
            // }
        }
    }
}
String and &amp;str Difference
std::core::String is a heap-allocated, growable (mutable) text type. &str is a slice (immutable view) into a String or static text literals (string literals).

Kod snippet'i

use std::core::String;

fn main() -> void {
    let s1: String = String::from("Hello"); // Create a String from an &str literal

    let s2 = "World"; // This is an &str literal (immutable)

    println!("{} {}", s1, s2); // String and &str implement the Display trait

    let mut s3 = String::new(); // An empty, mutable String
    s3.push_str(&s1); // Append the &str slice of s1 to s3
    s3.push_str(s2);  // Append the &str (s2) slice to s3
    s3.push_str("!"); // Append an &str literal directly
    println!("Concatenated String: {}", s3);

    fn take_slice(text: &str) -> void {
        println!("Function received slice: {}", text);
    }

    take_slice(&s1); // Pass the &str slice of the String
    take_slice(s2);  // Pass the &str literal
    take_slice(&s3); // Pass the &str slice of the String
}
Using Common Traits (Debug, Display, Clone)
Traits specify shared behaviors that different types can implement.

Display: Used in places like println!("{}", value) or format!("{}", value). Provides human-readable formatting.
Debug: Used in places like println!("{:?}", value). Provides a useful debug format for developers.
Clone: Allows creating a deep copy using the value.clone() method. Used when types that are not Copy need to be explicitly duplicated.
Copy: A marker trait indicating that assignment (=) or passing a value performs an automatic bitwise copy.
<!-- end list -->

Kod snippet'i

use std::core::{Debug, Display, Clone};

#[derive(Debug, Clone)] // Can be automatically implemented by the compiler
struct MyStruct {
    id: int,
    data: std::core::String, // String is not Copy, so MyStruct is not Copy
}

// Let's manually implement the Display trait for MyStruct
impl Display for MyStruct {
    fn fmt(&self, f: &mut core::Formatter) -> core::Result<(), core::Error> {
        // Use the Formatter to write the formatted output
        // f.write_str("MyStruct { id: ")?; // Hypothetical write_str method
        // f.write_int(self.id)?;          // Hypothetical write_int method
        // f.write_str(", data: \"")?;
        // f.write_str(&self.data)?;
        // f.write_str("\" }")?;
        // Ok(()) // Success result

        // Or if format! macro is available (often used in Display implementations)
         format!("MyStruct {{ id: {}, data: \"{}\" }}", self.id, self.data).fmt(f)
    }
}


fn main() -> void {
    let original = MyStruct { id: 1, data: String::from("hello") };

    // let copy = original; // ERROR: MyStruct is not Copy, this would be a move

    let clone = original.clone(); // Valid: Explicit cloning

    println!("Original (Debug): {:?}", original); // Uses the Debug trait
    println!("Clone (Debug): {:?}", clone);

    println!("Original (Display): {}", original); // Uses the Display trait
    // println!("Clone (Display): {}", clone); // Same output

    // Since String is not Copy, 'original' is still valid because cloning does not move ownership.
    println!("Original is still valid: {}", original.id);

    // If original had been moved, this would cause an error
    // let moved_original = original; // Ownership moved to moved_original
    // println!("Original (Debug): {:?}", original); // ERROR! original is moved
}
3. Collections (std::collections)
Provides various data structures. The most common ones are Vec and HashMap.

Using Vec&lt;T>
A resizable array.

Kod snippet'i

use std::collections::Vec;
use std::core::Option::{Some, None}; // For using Option

fn main() -> void {
    let mut numbers: Vec<int> = Vec::new(); // Create an empty vector of ints

    numbers.push(10); // Add element to the end
    numbers.push(20);
    numbers.push(30);

    println!("Vector length: {}", numbers.len()); // length method

    // Safe access to elements by index (returns Option)
    let first = numbers.get(0); // Option<&int>
    match first {
        Some(value_ref) => println!("First element: {}", *value_ref), // Dereference the reference
        None => println!("Vector is empty."),
    }

    // Pop elements (returns Option and returns ownership of the element)
    let last = numbers.pop(); // Option<int>
    if let Some(value) = last { // Extract the value from the Option
        println!("Popped last: {}", value);
    }
    println!("Length after pop: {}", numbers.len());

    // Iteration over the vector (by default using immutable references)
    for number_ref in &numbers {
        println!("Vector element: {}", *number_ref); // Dereference the reference
    }

    // Mutable iteration
    // for number_mut_ref in &mut numbers {
    //    *number_mut_ref += 1; // Change the value via mutable reference
    // }
}
Using HashMap&lt;K, V>
Stores key-value pairs. Keys (K) must implement the Hash and Eq traits.

Kod snippet'i

use std::collections::HashMap;
use std::core::String; // Need String for keys (String implements Hash and Eq)
use std::core::Option::{Some, None}; // For using Option

fn main() -> void {
    let mut scores: HashMap<String, int> = HashMap::new();

    // Insert key-value pairs (insert returns Option<V>, containing the old value if any)
    scores.insert(String::from("Alice"), 10);
    scores.insert(String::from("Bob"), 25);
    scores.insert(String::from("Charlie"), 15);

    // Safe access to values by key reference (returns Option<&V>)
    let alice_score = scores.get(&String::from("Alice")); // Pass a reference to the key
    match alice_score {
        Some(score_ref) => println!("Alice's score: {}", *score_ref), // Dereference the reference
        None => println!("Alice not found in map."),
    }

    // Mutable access (returns Option<&mut V>)
    // if let Some(score_mut_ref) = scores.get_mut(&String::from("Bob")) {
    //     *score_mut_ref += 5; // Update the score
    //     println!("Bob's new score: {}", *score_mut_ref);
    // }

    // Removing elements (returns Option<V>, containing the removed value if it existed)
    let removed_score = scores.remove(&String::from("Charlie"));
    if let Some(score) = removed_score {
        println!("Charlie removed from map, score: {}", score);
    }

    // Iteration over the map (by default using (&K, &V) reference pairs)
    println!("Remaining scores:");
    for (name_ref, score_ref) in &scores {
        println!("{}: {}", *name_ref, *score_ref); // Dereference the references
    }
}
4. Standard Input/Output (std::io)
Interacts with the console and other I/O sources.

Kod snippet'i

use std::io::{self, Read, Write}; // io module and Read/Write traits
use std::core::String; // Need String
use std::core::Option::{Some, None}; // Functions like read_line might return Option
use std::core::Result::{Ok, Err}; // I/O functions return Result

fn main() -> Result<(), io::Error> { // main can also return an error!
    // Writing to console (println! macro uses io::stdout())
    println!("Please enter your name:");

    // Reading from console (hypothetical read_line method)
    let mut input_line = String::new();
    // let read_result = io::stdin().read_line(&mut input_line); // read_line might return Option<usize> or Result<usize, Error>
    // match read_result {
    //     Some(bytes_read) => { println!("Read {} bytes.", bytes_read); },
    //     None => { println!("End of input."); }, // EOF
    // }

    // Alternatively, reading raw bytes
    println!("Please enter a few characters:");
    let mut buffer = [0u8; 10]; // 10-byte buffer
    let read_res = io::stdin().read(&mut buffer); // Use the Read trait

    match read_res {
        Ok(bytes_read) => {
            println!("Read {} bytes.", bytes_read);
            // Process the bytes read (perhaps convert to String)
            // let read_string = String::from_utf8(&buffer[0..bytes_read]); // Hypothetical from_utf8 function
            // match read_string {
            //     Ok(s) => println!("Read text: {}", s),
            //     Err(_) => println!("Read bytes are not valid UTF-8."),
            // }
        },
        Err(err) => {
             println!("Read error: {}", err.description());
             return Err(err); // Propagate the error
        }
    }

    // Writing directly to stdout/stderr
    let data_to_write: [u8; 6] = [72, 69, 76, 76, 79, 10]; // ASCII for "HELLO\n"
    io::stdout().write(&data_to_write)?; // Use the Write trait, propagate error
    io::stderr().write_all("This is an error message.\n".as_bytes())?; // write_all and as_bytes() hypothetical
    io::stdout().flush()?; // Flush the buffer, propagate error

    Ok(()) // Success return
}
5. File System (std::fs)
Used for working with files and directories.

Kod snippet'i

use std::fs::{self, File}; // fs module and File struct
use std::io::{self, Read, Write}; // File implements Read/Write
use std::core::Result::{Ok, Err};
use std::core::String; // Need String

fn main() -> Result<(), io::Error> { // main can return an error!
    let filepath = "example.txt";
    let content_to_write = "This is the file content.\nSecond line.";

    // Writing to a file (hypothetical fs::write convenience function)
    println!("Writing to '{}'...", filepath);
    fs::write(filepath, content_to_write)?; // Propagate error

    println!("'{}' successfully written.", filepath);

    // Reading from a file (hypothetical fs::read_to_string convenience function)
    println!("Reading from '{}'...", filepath);
    let read_content = fs::read_to_string(filepath)?; // Propagate error

    println!("Read content:\n{}", read_content);

    // Manual file opening, reading, and closing
    // File is automatically closed when it goes out of scope (thanks to the Drop trait).
    println!("Manually opening '{}'...", filepath);
    let mut file = File::open(filepath)?; // Open file in read mode, can return error

    let mut buffer = [0u8; 50]; // 50-byte buffer
    let bytes_read = file.read(&mut buffer)?; // Read from file into buffer, can return error

    println!("Manually read {} bytes.", bytes_read);
    // Use the data in buffer[0..bytes_read]...

    // Deleting a file
    // fs::remove_file(filepath)?; // Propagate error
    // println!("'{}' deleted.", filepath);


    Ok(()) // Success return
}
6. Math (std::math)
Contains basic mathematical functions and constants.

Kod snippet'i

use std::math; // Import the math module

fn main() -> void {
    let radius = 5.0;
    let area = math::PI * radius * radius;
    println!("Area of circle with radius {}: {}", radius, area);

    let angle_degrees = 90.0;
    let angle_radians = angle_degrees * math::PI / 180.0; // Convert degrees to radians

    let sine_val = math::sin(angle_radians);
    let cosine_val = math::cos(angle_radians);
    println!("sin({}) = {}", angle_degrees, sine_val); // Approx 1.0
    println!("cos({}) = {}", angle_degrees, cosine_val); // Approx 0.0

    let num = 25.0;
    let sqrt_num = math::sqrt(num);
    println!("{}.0's square root: {}", num, sqrt_num);

    let base = 2.0;
    let exp = 3.0;
    let power_val = math::pow(base, exp);
    println!("{}^{} = {}", base, exp, power_val);

    let integer_val = -10;
    let abs_val = math::abs(integer_val); // Calls the integer abs function
    println!("|{}| = {}", integer_val, abs_val);

    let float_val = -10.5;
    let abs_float_val = math::abs(float_val); // Calls the float abs function
    println!("|{}| = {}", float_val, abs_float_val);

    let max_int = math::max(100, 200); // Calls the generic max function
    println!("max({}, {}) = {}", 100, 200, max_int);
}
7. Networking (std::net) - A Brief Look
Provides basic networking capabilities (TCP, UDP). Typically involves error handling.

Kod snippet'i

// use std::net::{self, TcpListener, TcpStream, SocketAddr, IpAddr, Ipv4Addr}; // Import needed items
// use std::io::{Read, Write}; // TcpStream implements Read/Write
// use std::core::Result::{Ok, Err};
// use std::thread; // May need to spawn new threads for a server
// use std::time::Duration; // For sleep

/*
// A Simple TCP Server (Example) - Full implementation details are omitted
fn start_tcp_server() -> Result<(), net::Error> {
    // Bind to address 127.0.0.1:8080 (start listening)
    let addr_str = "127.0.0.1:8080";
    // let addr = addr_str.parse::<SocketAddr>()?; // Hypothetical parse method and ? operator

    let listener = net::TcpListener::bind(addr)?; // Binding can fail

    println!("Listening on {}", addr);

    // Accept incoming connections
    // listener.incoming() is a hypothetical iterator
    // for stream_result in listener.incoming() {
    //     match stream_result {
    //         Ok(mut stream) => {
    //             // Spawn a new thread for each connection
    //             thread::spawn(move || -> Result<(), io::Error> { // Hypothetical move closure
    //                 println!("Connection arrived: {:?}", stream.peer_addr()?); // Get peer address

    //                 let mut buffer = [0u8; 512];
    //                 loop {
    //                     // Read from the connection
    //                     let bytes_read = stream.read(&mut buffer)?; // Use Read trait

    //                     if bytes_read == 0 {
    //                         println!("Connection closed.");
    //                         break; // EOF
    //                     }

    //                     // Write back what was read (echo)
    //                     stream.write_all(&buffer[0..bytes_read])?; // Use Write trait
    //                 }
    //                 Ok(()) // Thread task finished successfully
    //             });
    //         },
    //         Err(err) => {
    //             println!("Connection accept error: {}", err.description());
    //         }
    //     }
    // }
    Ok(()) // Server loop would not normally finish, added for example
}

// A Simple TCP Client (Example)
fn connect_to_server() -> Result<(), net::Error> {
     let addr_str = "127.0.0.1:8080";
     // let addr = addr_str.parse::<SocketAddr>()?;

     println!("Connecting to server: {}", addr);
     let mut stream = net::TcpStream::connect(addr)?; // Establish connection, can return error

     println!("Connection successful! Sending message...");

     // Write to the server
     stream.write_all("Hello server!".as_bytes())?; // as_bytes() converts &str to &[u8]
     stream.flush()?;

     // Read response from the server
     let mut buffer = [0u8; 512];
     let bytes_read = stream.read(&mut buffer)?;

     let response = std::core::String::from_utf8_lossy(&buffer[0..bytes_read]); // Hypothetical from_utf8_lossy
     println!("Response from server: {}", response);

     Ok(())
}

fn main() -> void {
    // start_tcp_server(); // Start the server
    // connect_to_server(); // Run the client
}
*/
8. Concurrency (std::thread) - A Brief Look
Used for creating and managing new threads of execution.

Kod snippet'i

use std::thread; // Import the thread module
use std::time::Duration; // Duration is needed for thread::sleep from the time module
use std::core::Result::{Ok, Err}; // For thread::JoinHandle::join

fn main() -> void {
    println!("Main thread started.");

    // Spawn a new thread
    // The spawn function takes a closure (anonymous function)
    // The 'move' keyword (if D++ supports it) ensures that variables captured by the closure
    // have their ownership moved into the thread.
    let handle = thread::spawn(move || -> int { // The thread will return an integer
        println!("New thread started.");
        // Wait for a moment inside the thread
        thread::sleep(Duration::from_secs(1)); // Uses time::Duration
        println!("New thread finishing.");
        return 100; // The thread returns a value
    });

    println!("Main thread doing something else...");
    // The main thread can continue doing other work...

    // Wait for the spawned thread to finish and get its result
    let thread_result = handle.join(); // Returns Result<int, thread::JoinError>

    match thread_result {
        Ok(value) => println!("Thread finished successfully, result: {}", value),
        Err(err) => println!("Thread panicked or an error occurred: {}", err.description()), // JoinError should implement Error trait
    }

    println!("Main thread finished.");
}
9. Time (std::time)
Used for managing durations (Duration) and points in time (Instant, SystemTime).

Kod snippet'i

use std::time::{Instant, Duration, SystemTime}; // Import needed items
use std::thread; // For thread::sleep

fn main() -> void {
    // Get an instant point in time
    let start_time = Instant::now();
    println!("Start time recorded.");

    // Simulate some work (e.g., sleep for a short duration)
    thread::sleep(Duration::from_millis(500)); // Sleep for 500 milliseconds

    // Calculate the elapsed time
    let elapsed_duration = start_time.elapsed();
    println!("Elapsed time: {} seconds", elapsed_duration.as_secs()); // Whole seconds
    println!("Elapsed time: {} milliseconds", elapsed_duration.as_millis()); // Hypothetical as_millis method
    println!("Elapsed time: {} nanoseconds", elapsed_duration.as_nanos()); // Full nanoseconds

    // Creating specific durations
    let five_seconds = Duration::from_secs(5);
    let ten_minutes = Duration::from_secs(60 * 10);
    // let total_duration = five_seconds + ten_minutes; // If Duration implements the Add trait

    // Using system time
    let system_now = SystemTime::now();
    println!("System time recorded.");
    // Operations like calculating the duration since/until the Unix epoch can be done with SystemTime
    // let duration_since_epoch = system_now.duration_since(SystemTime::UNIX_EPOCH)?; // Unix_EPOCH constant and Result

    // Sleep for another duration
    thread::sleep(five_seconds);
    println!("Slept for another 5 seconds.");

    let total_elapsed = start_time.elapsed(); // Total time elapsed
    println!("Total elapsed time: {} seconds", total_elapsed.as_secs());
}
10. Using Traits
You can call trait methods defined on various types. You can achieve dynamic polymorphism with &dyn Trait.

Kod snippet'i

use std::io::Write; // Need to use the Write trait

// Function that writes data to anything that implements the Write trait
fn write_data_to_writer<W: Write>(writer: &mut W, data: &[u8]) -> Result<(), std::io::Error> {
    writer.write_all(data)?; // write_all is a method on the Write trait (hypothetical convenience)
    writer.flush()?;       // flush is a method on the Write trait
    Ok(())
}

fn main() -> Result<(), std::io::Error> { // main can return an error!
    use std::io;
    use std::fs;

    let mut stdout = io::stdout(); // Stdout implements Write
    let mut file_handle = fs::File::create("output_trait.txt")?; // File implements Write

    let data = [1, 2, 3, 4, 5]; // A u8 array ([u8; 5]), can be passed as &[u8]

    // Call the function with Stdout
    println!("Writing to Stdout...");
    match write_data_to_writer(&mut stdout, &data) {
        Ok(_) => println!("Stdout write successful."),
        Err(err) => println!("Stdout write error: {}", err.description()),
    }

    // Call the function with File
    println!("Writing to file...");
    match write_data_to_writer(&mut file_handle, &data) {
        Ok(_) => println!("File write successful."),
        Err(err) => println!("File write error: {}", err.description()),
    }

    // The File handle is automatically closed when it goes out of scope (thanks to the Drop trait)
    Ok(()) // Successful return
}
11. Error Handling Patterns (Result and ?)
Using Result and the ? operator when calling functions that might fail is a common D++ pattern.

Kod snippet'i

use std::io;
use std::fs;
use std::core::Result::{Ok, Err};
use std::core::String;

// An operation involving multiple steps that might fail
fn copy_file_content(source_path: &str, dest_path: &str) -> Result<(), io::Error> {
    // 1. Open the source file
    let mut source_file = fs::File::open(source_path)?; // ? operator immediately returns Err if open fails

    // 2. Create the destination file
    let mut dest_file = fs::File::create(dest_path)?; // ? operator immediately returns Err if create fails

    // 3. Create a buffer to read into
    let mut buffer = [0u8; 1024]; // 1KB buffer

    loop {
        // 4. Read from the source file
        let bytes_read = source_file.read(&mut buffer)?; // ? operator immediately returns Err if read fails

        if bytes_read == 0 {
            break; // EOF (end of file)
        }

        // 5. Write to the destination file
        dest_file.write_all(&buffer[0..bytes_read])?; // ? operator immediately returns Err if write fails (write_all hypothetical)
    }

    // 6. Flush the destination file
    dest_file.flush()?; // ? operator immediately returns Err if flush fails

    Ok(()) // Return Ok(()) if all steps were successful
}

fn main() -> void {
    // Example usage: Try to copy a file
    let copy_result = copy_file_content("source.txt", "destination.txt"); // Needs source.txt to exist

    match copy_result {
        Ok(_) => println!("File successfully copied."),
        Err(err) => {
            println!("File copy error: {}", err.description());
            // Handle the error...
        }
    }
}
```
This guide provides an introduction to interacting with the basic modules of the D++ Standard Library. As more advanced features (generics, iterators, macros, concurrency synchronization primitives, etc.) are added to the library, this guide can be expanded. By using these fundamental structures and patterns, you can create safer, more performant, and readable code in D++.