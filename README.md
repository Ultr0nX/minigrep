**MINI GREP is a CLI tool built in Rust by following `The Rust Book`** 

**A simple version of `ripgrep` created or developed by *Andrew Gallant***

Rust’s speed, safety, single binary output, and cross-platform support make it an ideal language for creating command line tools,


# Table of contents
- [What Is It and What It does ?](#what-is-it)
- [Set up / How to run](#quick-start)
- [code Walkthrough](#code-walkthrough)

# What is it

It Is an CLI tool built using **RUST** Programming language while following the Rust book.

It searches for a particular word in a specified file and prints lines that contains word. 
This tool takes two arguments while running the application, one is the word and the other is fileName

command to run the Application : 
```bash
cargo run -- word file.txt
```

# Quick start

1. clone the repo
2. cd minigrep
3. run the command :

   ```bash
   cargo run -- word_to_search filename.txt
   ```
⚠ make sure that , you have that file( you specified in the command) in the folder
( I have added poem.txt in the repo , you can search for words in that poem file )


# Code walkthrough 

<details>
<summary> What happens before program even starts ( some low level stuff) </summary>

🚨 Before going into the code , let's see what happens when we run :
```bash
cargo run -- hello sample.txt
```
`--` is a separator used by Cargo. Everything after `--` is passed as arguments to our program.

OK let's begin ,  

- when we run that command the shell parses and understands the command and arguments .
- `cargo run`
   - compiles our program
   - locates the executable (e.g., `target/debug/minigrep`)
   - requests the OS to start the executable (internally this results in an `execve` system call)
- When `execve` is called, the kernel:
   - replaces the current process image with the new program
   - assigns a new program image to the same PID
   - creates a fresh virtual address space
   - loads the executable into memory
   - sets up registers and the initial stack
   - copies command-line arguments into process memory 

- The OS creates the argument array and places it into the new process memory (Our program does not create this array , programs receives it)
- The kernel constructs this structure in the new process 
   - [
      "program name",
      "Argument  1",
      "Argument  2"
      ]

   - **program name as the first argument in the array is UNIX standard**
- How does execution actually starts :
   
   After setting up memory :
   
   1. Kernel sets CPU registers
   2. Sets `Instruction Pointer` to Program entry point
   3. Jumps into run time startup code , `not main()`
   ```text
   _start --> Rust Runtime --> main()
   ```
   - Run time reads raw argc , argv , and converts them safely into Rust types 

  ⚠ Rust copies the argument data into its own memory and does not keep references to OS-managed memory. This is one of the reasons Rust provides strong memory safety guarantees. 
</details>

## Now actual code walkthrough

### src/main.rs:
<details> 
<summary> Code </summary>

```rust 
use std::env;
use std::fs;
use std::process;
use std::error::Error;
use minigrep::{search, search_case_insensitive};

fn main() {

    let args : Vec<String> = env::args().collect();
    let config = Config::build(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    if let Err(e) = run(config) {
        eprintln!("Application error: {e}");
        process::exit(1);
    }

}

// fs::read_to_string takes the file_path , opens that file, and returns a value of type std::io::Result<String> that contains the file’s contents

fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;
    let results = if config.ignore_case {
        search_case_insensitive(&config.query, &contents)
    } else {
        search(&config.query, &contents)
    };
    for line in results {
        println!("{line}");
    }
    Ok(())
}

struct Config {
    query : String,
    file_path : String,
    pub ignore_case: bool,
}

impl Config {
    fn build (args : &[String]) -> Result<Config , &'static str> {

        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path= args[2].clone();

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config { query , file_path, ignore_case, })
    }
}
```
</details> 

Let's start from the main function that is where our program starts running .

### Reading Arguments
```rust
let args: Vec<String> = env::args().collect();
```
- Here , we collect the arguments as vector of strings 
- To read values of command line arguments we pass to it , we'll need the `std::env::args` functions provided in Rust's standard library. This function returns an **iterator** of the command line arguments passed.
    - Iterator produces  a series of values .
- So here , the iterator produces a series of values of command line arguments and we can collect them using `collect()`to turn it into a collection , such as a vector , that contains all the elements the iterator produces (it is a method on that iterator) 
- So now the `args` is an Vector of String type that contains command line argument values.

### Configuration 
<details>
<summary> code </summary>

```rust
struct Config {
    query : String,
    file_path : String,
    pub ignore_case: bool,
}


impl Config {
    fn build (args : &[String]) -> Result<Config , &'static str> {

        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path= args[2].clone();

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config { query , file_path, ignore_case, })
    }
}
```
</details>

- here , we configured our values as a struct `Config`
- And implemented build function on it 
- `build()` function takes arguments and returns `Result<Config , &'static str>`
- It gives error if number of arguments are less than three(3) 
- And here the values are cloned , copied into variables and then returned as `Ok(Config { query , file_path, ignore_case, })`
- That returned `Config` is catched by config variable in the main function (see code below)
<details>
<summary>code</summary>

```rust 
let config = Config::build(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });
```

</details>

- If we get `Config` from build function then `unwrap_or_else()` works as `unwrap()`
- Otherwise ( error) , the Closure function is executed and program terminates as we called `process::exit(1)` with exit code `1`.
- here we used `eprintln!()` to print error message in standard error stream.

### Running the logic

<details>
<summary>code</summary>

```rust
fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;
    let results = if config.ignore_case {
        search_case_insensitive(&config.query, &contents)
    } else {
        search(&config.query, &contents)
    };
    for line in results {
        println!("{line}");
    }
    Ok(())
}
```
</details>

- In `main()` after getting or building the config , we call run function by passing that config as an argument
- <details>

    <summary>code</summary>
        
    ```rust
        if let Err(e) = run(config) {
            eprintln!("Application error: {e}");
            process::exit(1);
        }

    ```
    </details>

- `run()` function takes Config and returns `Result<() ,Box<dyn Error>>`
- This line takes file path / name and reads the content into `contents` var.

```rust
    let contents = fs::read_to_string(config.file_path)?;
```
- this function calls search functions based on **environment varible `IGNORE_CASE`** . If it is set then it calls `search_case_insensitive()` function , otherwise it calls `search()` ( these two functions are defined in lib.rs file)

- And prints the result ( which is from any one of those two function)

### src/lib.rs

<details>
<summary>code</summary>

```rust
pub fn search<'a> (query: &str , contents: &'a str) -> Vec<&'a str> {
    
    let mut results = Vec::new(); 


    for line in contents.lines() {
        if line.contains(query){
            results.push(line);
        }
    }
    results
}

pub fn search_case_insensitive <'a> (
    query: &str ,
    contents : &'a str
) -> Vec<&'a str> {
    let query = query.to_lowercase();
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.to_lowercase().contains(&query){
            results.push(line);
        }
    }
    results
}

#[cfg(test)]
mod tests {
    use super::* ;

    #[test]
    fn case_sensitive() {
        let query = "duct" ;
        let contents = "\
Rust : 
safe , fast , productive.
pick three.
Duct tape.";

        assert_eq!(vec!["safe , fast , productive."] , search(query , contents));
    }


    #[test]
    fn case_insensitive() {
        let query = "rUst";
        let contents = "\
Rust:
safe , fast , productive.
Pick three.
Trust me.";

    assert_eq!(vec!["Rust:" , "Trust me."] , search_case_insensitive(query , contents));
    }
}
```
</details>

- Both the functions contains same logic , just in `search_case_insensitive()` function we convert query to lower case and we shadow the query
- `lines()` - gives an iterator over contents
- `contains()` check whether that word present in that line or not .
- If any line contains that word we push that line into result and at last we return that `results`

- In `run()` function in main.rs file ,we print the results line by line using for loop.

- And I wrote Tests for it in lib.rs file

## Final Notes

This project was built by following *The Rust Book* to better understand how real command-line tools are structured in Rust.

The goal of MiniGrep is learning — especially ownership, error handling, iterators, and basic OS-level execution flow — rather than replacing existing tools like `ripgrep`.

More detailed memory-level topics (stack setup, unwinding, runtime internals) are left out on purpose, as the README is already quite large.

---
Built for learning.