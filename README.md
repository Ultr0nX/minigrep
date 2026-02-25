**MINI GREP is a CLI tool built in Rust by following `The Rust Book`** 

**A simple version of `ripgrep` created or developed by *Andrew Gallant***


# Table of contents
- [What Is It and What It does ?](#what-is-it)
- [Set up / How to run](#quick-action)
- [code Walkthrough](#code-walkthrough)

# What is It

It Is an CLI tool built using **RUST** Programming language while following the Rust book.

It searches for a particular word in a specified file and prints lines that contains word. 
This tool takes two arguments while running the application, one is the word and the other is fileName

command to run the Application : 
```bash
cargo run -- word file.txt
```

# Quick action

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

   

