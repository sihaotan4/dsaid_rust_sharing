
# Cult of the Crab

An introduction to the Rust language

![[DALL·E 2023-10-16 12.18.03 - top down occult oil painting of a red crab.png]]
DSAID QA Sharing, Nov 2023

notes:

That’s the core of what Rust is doing: it provides you with a language to precisely express the contracts between components, such that components can be integrated in a machine-checkable way.

---

# Preface

1. In the next 40 mins I promise to get you curious, and maybe even excited to try Rust. 
2. This presentation is heavily inspired by [NoBoilerplate](https://www.youtube.com/c/NoBoilerplate).
3. The entire presentation and script is a `.md` document on my github page, complete with all links and resources. 
:::<!-- element align="left" -->
---
# section 1: 
# history lesson

---

# Rust

A general purpose programming language that emphasizes performance, memory safety, type safety, and concurrency.

Rust is named after a particularly robust type of fungi that is “over-engineered for survival”.

:::<!-- element align="left" -->

notes:
- What is Rust - its a language that's runs relatively fast, close to C speeds about 100x faster than vanilla python. It is notorious for giving new developers grief at compile time but you are left with code that almost never crashes at runtime. I'll touch on this more.
- Microsoft estimates that 70% of the vulnerabilities in its code are due to memory errors from code written in these languages.
- He began designing a new computer language, one that he hoped would make it possible to write small, fast code without memory bugs. He named it Rust, after a group of remarkably hardy fungi that are, he says, “over-engineered for survival.”

---

# Trust in Rust?

Rust has a halo effect for computer nerds:
- 2023 Stack Overflow Survey's most loved language for 8 years in a row
- Rust Foundation is backed by five horsemen of the apocalypse: AWS, Huawei, Google, Microsoft, and Mozilla. Mission critical code is written in Rust. 
- In 2022, it joined C and assembly to be supported in the development of the Linux kernel.

:::<!-- element align="left" -->
notes:
- While it started as a research platform - it was always meant as a programming language solving real tangible issues in large codebases. 
---
Some notable examples of industry adoption:
- Cloudfare - [Machine learning inference in microseconds](https://blog.cloudflare.com/how-cloudflare-runs-ml-inference-in-microseconds/)
- Deliveroo - [Fast route matching algorithm in Rust](https://deliveroo.engineering/2019/02/14/moving-from-ruby-to-rust.html)
- Figma - [Multiplayer web service in Rust](https://www.figma.com/blog/rust-in-production-at-figma/)
- Discord - [Reduced CPU and latency spikes switching from Go to Rust](https://discord.com/blog/why-discord-is-switching-from-go-to-rust)
- AWS - Critical infrastructure like Firecracker (AWS Lambda), S3, EC2, etc

:::<!-- element align="left" -->
notes:
We can see that they are using one language to do a lot of different things - across many different domains. 
There is code that is doing very high-level application type code - and there is code that needs to touch the low-level memory and resource control. 
So this is very interesting - one do-it-all language with both high-level ergonomics and low-level control

---
# section 2: 
# Rust features

---
Is this python code syntactically correct? What problems can you see? 
```python
int(item["ViewCount"]["N"])
```

+ Is item a dict?
+ Does the ViewCount attribute even exist? Is it itself a dict?
+ Is there a N attribute in ViewCount?
+ Can the value be parsed as an integer with int()?

notes:
Our lives, as developers, are governed by errors.  
These are the problems that will crash your programs at runtime right before lunch or at 6pm when you're going home:

1. The `item` is a `hashmap`,
2. There is a `ViewCount` attribute,
3. The `ViewCount` attribute is a `hashmap`,
4. There is an `N` attribute in the `ViewCount` hashmap, and
5. The value can be parsed as an integer with `int()`.

1. That was code which could have been written to prod. Unit testing and peer review might not be sufficient to catch everything. 
2. This gets even harder when you're trying to coordinate within a team and with a large codebase. 
3. If we want this to never crash, we need to handle all the possible runtime errors. 

---
### Rust keeps you safe

- The Rust compiler uses static analysis and enforces strict rules.
- Your Rust programme (*if it compiles*) will be highly resistant to common programming errors, making it less likely to crash.

notes:

Static analysis in Rust works by applying a set of small atomic rules and principles, and these small rules collectively create more complex guarantees of code safety.

While the Rust compiler doesn't guarantee that code can "never" crash, it significantly reduces the likelihood of many types of crashes by using static analysis and enforcing strict rules.

---

### Your first Rust program?

```Rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.height * rectangle.width
} 

fn main() {
    let rectangle: Rectangle = Rectangle {width: 4, height: 2 };
    let area = area(&rectangle);
    println!("{}", area);
}
```

notes:
that whole last section was a lot of theory - so let's look at some rust code and see if we can make sense of it. 

explain at a high level and skim through

---

```Rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rectangle = Rectangle { width: 4, height: 2 };
}
```

notes:
Rust is a statically-typed language, which means that both the types and sizes of variables are determined at compile time, and the compiler enforces type safety based on this information. 

This struct (equivalent to a class) has two attributes - both u32. We know the type and size and this gives us performance - it also means a smart compiler can make use of static analysis at compile time to do these checks for us.

If we were to use floats or negative values when we declare rectangle in main - the compiler will immediately throw an error. 

---
Enums let us capture the idea of "variants"

```Rust
enum Shape {
    Rectangle { width: u32, height: u32 },
    Circle { radius: f64 },
}
```

notes:
The client wants to change requirements - our inputs are not just rectangles but also circles.

We can use enums - enums give you a way of saying a value is one of a possible set of values. To do this, Rust allows us to encode these possibilities as an enum.

Why not just create another struct? - enums can be more concise than structs. We can attach different data to each variant of the enum directly, so there is no need for an extra struct. 

Functions can also receive enums as inputs...

---
This creates a pattern of having to deal with each Enum variant

```Rust
fn area(shape: &Shape) -> f64 {
    match shape {
        Shape::Rectangle { width, height } => {
		    (*width as f64) * (*height as f64)
	    },
        Shape::Circle { radius } => {
		    std::f64::consts::PI * radius * radius
	    },
    }
}
```

notes:
here we use a match control flow. 

Two key points:

First is that the function signature has changed - we must uphold the type contract. Can't just have a function that accepts circles or rectangles willy nilly. They have different data structures inside!
So we allow the function to take in an enum of Shape instead. 

Second - matches in Rust are _exhaustive_: we must exhaust every last possibility in order for the code to be valid. 

Both the input type guarantee (that this is a shape enum) plus the guarantee that all variants are matched - means this function must deal with all edge cases within its scope. 

You don't have to intentionally remember this - if you forget the compiler will make sure you do it. 

---
###  What about memory safety?

notes:
memory management is critical to software development
- Efficient memory management ensures that your software uses system resources (RAM) judiciously
- memory leaks
- security - preventing memory-related security exploits, such as buffer overflows and dangling pointers.
- performance - allocation and deallocation is costly

---
#### One strategy: Garbage Collection

![[discord_service_rewrite.png]] <!-- element class="fragment" -->
Source: [Discord](https://discord.com/blog/why-discord-is-switching-from-go-to-rust)

notes:
Garbage collection (GC) is a form of automatic memory management. The garbage collector attempts to reclaim memory which was allocated by the program, but is no longer referenced; such memory is called garbage. 

Java Python, Go, Ruby... they all rely on this excellent strategy. 

But GC has limitations... (print animation)

This is a case study from discord:
the “Read States” service. Its sole purpose is to keep track of which channels and messages you have read. Read States is accessed every time you connect to Discord, every time a message is sent and every time a message is read. In short, Read States is in the hot path.

It was a pretty simple service, but it had slow tail latencies. Because Go is a garbage collection (GC) language, as objects are created and released, every so often, the garbage collector needs to stop execution of the program and run a garbage collection pass. While the GC is running, the process is unable to respond to requests, and you can see the spikes on the CPU and response time graphs when it’s running.

To fix the issue, Discord decided to try rewriting the service in Rust, and these are the results. The Go implementation is on the left and the Rust implementation is on the right. While the GC spike pattern is gone on the Rust graph, the really amazing difference is the magnitude of the change. The Go and Rust graphs are actually using different units.

The Rust version is more than 10 times faster over all with the worst tail latencies reduced 100 times.

---
### No GC: Rust uses ownership

1. Rust doesn't have a garbage collector. 
2. Memory is managed through a system of ownership with a set of rules that the compiler checks. 
3. If any of the rules are violated, the program won’t compile. 
4. None of the features of ownership will slow down your program while it’s running.

---
Snippet of how ownership works

```Rust
fn take_ownership(some_string: String) {
    println!("Received: {}", some_string);
    // some_string goes out of scope and is dropped here
}

fn main() {
    let my_string = String::from("Hello, Rust!");

    take_ownership(my_string); 
    // Ownership of my_string is transferred to take_ownership

    // Uncomment the line below to get an error
    // println!("Trying to use my_string: {}", my_string);
}
```

notes:
In this example, we have two functions: main and take_ownership. 
main creates a String and owns it. It then calls take_ownership and transfers ownership of the String to that function. 
After the function call, my_string is no longer valid, and trying to use it results in a compile-time error.

---
```Rust
fn borrow_string(some_string: &String) {
    println!("Borrowed: {}", some_string);
    // some_string borrows the String but does not own it
}

fn main() {
    let my_string = String::from("Hello, Rust!");

    borrow_string(&my_string); 
    // my_string is borrowed by borrow_string, but ownership remains in main

    println!("Still using my_string: {}", my_string);
}
```

notes:
In this example, `borrow_string` borrows the `String` without taking ownership. The ownership remains in the `main` function. This allows you to use `my_string` after the function call to `borrow_string`.

These two functions illustrate the concept of ownership in Rust. In the first function, ownership is transferred, and the variable cannot be used afterward. In the second function, ownership is not transferred, and the variable remains valid for further use.

rule of thumb is that if you don't need to mutate it - pass by reference instead of passing the variable. 

This distinction is very important - because these rules are upheld by the compiler, simple rules but they prevent complex bugs like memory leaks and data races.

---

```Python
def parse_thing(item):
	return int(item["ViewCount"]["N"])
```

```Rust
fn parse_thing(item: ExampleDataType) 
	-> Result<i32, Box<dyn std::error::Error>> {
	
    let viewcount = item.get("ViewCount").ok_or("ViewCount key not found")?;

    let n = viewcount.get("N").ok_or("N key not found in ViewCount")?;

    let parsed_result = n.parse::<i32>()?;

    Ok(parsed_result)
}
```

notes:
to close this off - let's rewrite that earlier python snippet.

i wrote a more verbose version in rust. Rust will not let you write unsafe code from the start. You must handle all errors.

- function signature
- 3 operations that can each fail with different errors
- Returning an Ok a the end

---
### pick your poison

![[compile_time_errors.png]]

notes:

It depends on your situation. Pick python if you need Rapid Development and Prototyping
Pick Rust when you need reliability and performance.  

---
# section 3
## developer experience

---
# Rust toolkit
1. rustup - installer
2. rustc - compiler
3. cargo - build tool
4. rust-analyzer (optional)

notes:
it's time to step back from the language and talk about tooling around it
essentially this is your developer experience

rustup
rustc

**Cargo** is the official package manager and build tool for the Rust programming language. It is an essential part of the Rust ecosystem. It is very very good. I actually enjoy using it and it's never given me problems with dependencies ever. Compared to pip/conda - cargo is in a league of its own. 

rust-analyzer is an open source project. an implementation of [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) for the [Rust](https://www.rust-lang.org/) programming language. It provides features like completion and goto definition for many code editors.

---

### Inner Development Loop

change code > compile the app > run test > run

1. lld or mold for faster linking of binaries
<!-- element class="fragment" -->
2. cargo bacon to reduce perceived compilation time
<!-- element class="fragment" -->

notes:
so this starts to look a bit different from python.

This is also known as the inner development loop.
The speed of your inner development loop is as an upper bound on the number of iterations that you can complete in a unit of time.
If it takes 5 minutes to compile and run the application, you can complete at most 12 iterations in an hour.

You might find that as the project becomes very large - this is slowing you down.

A sizeable chunk of time is spent in the linking phase - assembling the actual binary given the outputs of the earlier compilation stages.

bacon is a background rust code checker - this reduces perceived compilation time - it monitors for changes and runs `cargo check` 

---
### continuous integration pipeline

automated checks run on every commit:
1. cargo test (run unit and integration test)
2. cargo tarpaulin (code coverage)
3. cargo clippy (linter)
4. cargo fmt (formatter)
5. cargo audit (vulnerability scanning)

[Luca Palmieri - github actions template](https://github.com/LukeMathWalker/zero-to-production/tree/root-chapter-03-part0/.github/workflows) <!-- element class="fragment" -->

notes:
In trunk-based development we should be able to deploy our main branch at any point in time.
Every member of the team can branch off from main, develop a small feature or fix a bug, merge back into main and release to our users.

We use automated checks that run on every commit

- Tests are a first-class concept in the Rust ecosystem and you can leverage cargo to run your unit and integration tests
- Check if portions of the codebase are overlooked and not tested
- clippy is the official rust linter - spots unidiomatic code and common mistakes/inefficiencies
- cargo fmt is the official formatter
- sub-command to check if vulnerabilities have been reported in the dependency tree of your project

These few elements form a solid Rust CI pipeline for your projects that you can tweak further - you can click this link to get the template for github/gitlab.

---
Short break: How is everyone doing?
![[kidnapping_meme.png]]
---

# section 4
# Ecosystem

notes:
now we cover your potential relationship with rust
domain specific areas
how someone can learn this language

---
## [Are we learning yet?](https://www.arewelearningyet.com/) 

Rust's performance, low-level control, and zero-cost high-level abstractions make it a compelling alternative to more established ecosystems for Machine Learning. While the Rust ML ecosystem is still young and best described as experimental, several ambitious projects and building blocks have emerged.

notes:
Are we learning yet - is a catalogue for the state of machine learning in Rust.
This is the preamble that you will see on the website.

---
data processing
- serde 
- polars 
- tokenizers

data science 
- linfa
- smartcore

deep learning
- burn
- candle

:::<!-- element align="left" -->
notes:
serde - serializing and deserializing data (data structures to things like JSON and back)
polars - dataframe library, very fast multithreaded processing, larger than RAM, and a syntax closer to sql/spark
tokenizers - huggingface tokenizer library. Extremely fast (both training and tokenization), thanks to the Rust implementation. Takes less than 20 seconds to tokenize a GB of text on a server's CPU.

data science - think in terms of scikit-learn (not so good in rust). iterative nature is that well supported without notebook type environments

NN is where things change. lots of action but just highlighting the two big ones
-  Burn is looking for performance and flexibility (multiple backends, not just CUDA). Candle is focusing on a lightweight library that can target serverless and WASM. 

last thing is there is also activity on evolutionary algorithms and optimisation type problems - i didn't list them here because i don't know enough

---

Other "Are We Something Yet" catalogues:
- Are we async yet?
- Are we distributed yet?
- Are we game yet?
- Are we web yet?
- Are we quantum yet?

notes:
testament to how rust just creeps into every nook and cranny. very versatile language.

---

#### special mention: cloud deployments

- AWS 🤩💯💯💯
- Azure 🤔
- GCP ☠️

notes:
AWS SDK for Rust (Developer Preview) - very very good experience with deploying serverless aws lambdas with Rust. We can actually compile binaries that run natively on lambda hardware. 

Rust is a good fit for serverless functions with easy packaging of dependencies, good cold starts and nice ecosystem. 

If you want to do this you can try AWS Serverless Application Model Command Line Interface (AWS SAM CLI). Chef's kiss.

Azure - a bit later to the game
Azure believes in Rust - Azure CTO said this "it's time to halt starting any new projects in C/C++ and use Rust for those scenarios where a non-garbage collected language is required," he said. "For the sake of security and reliability, the industry should declare those languages as deprecated."

GCP - doesn't seem to be investing in Rust like the other two. So we probably won't see anything for the short-term

---
# learning rust

- [the book](https://doc.rust-lang.org/book/)
- [rustlings](https://github.com/rust-lang/rustlings)
- [advent of code](https://adventofcode.com/)
- [exercism](https://exercism.org/tracks/rust)
- [zero2prod](https://www.zero2prod.com/)

notes:
Small exercises to get you used to reading and writing Rust code

---

# THANK YOU
---