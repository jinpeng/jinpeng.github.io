---
title: Go Programming Lanauges Reading Notes Chapter 1
date: 2015-12-17 10:32:52
tags:
---

Tutorial
==========
1.1 Hellow, World
------------------

1. Go is compiled language, go toolchain convert source code into native machine instructions.
2. go command, and its subcommands: run, build, get
3. Go source code file structure: package, imports, declarations of functions, variables, constants and types.
4. Go code is organized into packages. Standard library has more than 100 packages.
5. Package main is a special package for standalone executable program.
6. Import exactly the packages needed.
7. Function declaration: func NAME PARAM_LIST RETURN_LIST.
8. Semicolon is not required at the end of line, but newline following certain tokens is converted into semicolon.
9. Strong stance on code formatting. gofmt tool and go command fmt subcommand.
10. goimports tool: go get golang.org/x/tools/cmd/goimports

1.2 Command-Line Arguments
---------------------------
1. os package provides functions and values for dealing with OS in a platform independent way.
2. os.Args is a variable for command-line arguments. It is a slice of strings.
3. Slice s: s[i], s[m:n], len(s)
4. All indexing in Go uses half-open intervals: include the first and exclude the last, s[m:n].
5. os.Args[0] is the name of the command itself. Arguments are os.Args[1:len(os.Args)], or os.Args[1:].
6. By convention, we describe package in a comment immediately preceding package declaration. For main package, this comment describes the program as a whole.
7. Variable not explicitly initialized is implicitely initialized to zero value for its type, 0 for numeric types and empty string "" for strings.
8. Go provides usual arithmetic and logical operators.
9. Each arithmetic and logical operator has corresponding assignment operator, like +=, *=.
10. Arithemtic operator + could concatenate two strings.
11. Short variable declaration i := 1.
12. Increment and decrement statements i++/i-- are statements, not expressions in C-like languages, and are only postfix. So j = i++ or ++i are both illegal.
13. for loop has several forms:

		for initialization; condition; post {
		}

		for condition {
		}

		for {
		}

		for _, arg := range os.Args[1:] {
		}

14. Blank identifier _ is used whenever syntax requires a variable name but program logic does not.
15. Several ways to declare varaible:

		s := ""           // shore variable declaration
		var s string      // default initialization
		var s = ""        // ususally for multiple variables
		var string = ""   // used when variable type and intial value type are not same

16. string.Join function
17. fmt.Println(os.Args[1:])

1.3 Finding Duplicated Lines
-----------------------------

1. A map hold a set of key/value pairs and provides constant-time operations to store, retrieve, or test for an item in the set.
2. map key can be any type whose values can be compared with ==, value can be any type.
3. If map does not contain key, map[key] return zero value to its type.
4. The order of map iteration is intentional random.
5. bufio.Scanner is the most useful feature for input/output.
6. fmt.Printf

		%d				decimal integer
		%x, %o, %b		hexadecimal, octal, binary
		%f, %g, %e		floating point number
		%t				boolean
		%c				rune(Unicode code point)
		%s				string
		%q				quoted string "abc" or rune 'c'
		%v				any value in natural format
		%T				type of any value
		%%				literal percent sign(no operand)

7. escape sequences: \t \n
8. fmt.Println formats arguments as if by %v, followed by a newline.
9. os.Open returns two values, an open file (*os.File) and error type. If error equals nil, the file opened successfully.
10. Functions and other package level entities may be declared in any order.
11. A map is a reference to the data structure created by make. Pass a map to a function, the function receives a copy of the reference, any modification in the function will be visible to reference held by the caller.
12. ioutil.ReadFile return a byte slice that must be converted into a string so it can be split by strings.Split.
13. bufio.Scanner, iotutil.ReadFile, ioutil.WriteFile use the Read and Write methods of *os.File.

1.4 Animated GIFs
------------------

1. Import a package whose path has multiple components, like image/color, we refer to the package with a name that comes from the last component.
2. The value of a constant must be a number, string, or boolean.
3. Composite literals: []color.Color{...}, gif.GIF{...}.
4. A struct is a group of values called fields, often of different types, that are collected together in a single object that can be treated as a unit.
5. When init a variable of a struct with some fields' value, other fields' value are zero value of their type.
6. Individual fields can be accessed with dot notation.


1.5 Feteching a URL
--------------------

1. http.Get function makes an HTTP request and, if there is no error, returns the result in the response struct resp. The Body field contains the server response a a readable stream.
2. ioutil.ReadAll function reads the entire response.


1.6 Feteching URLs Concurrently
--------------------------------

1. A gorountine is a concurrent function execution.
2. A channel is a communication mechanism that allows one gorountine to pass values of specified type to another gorountine.
3. The main function runs in a goroutine and go statement create additional gorountines.
4. When one gorountine attempts a send or receive on a channel, it blocks until another goroutine attempts the corresponding receive or send operation, at which point the value is transfered and both gorountines proceed.
5. ioutil.Discard is an io.Writer on which all Write calls succeed without doing anything.


1.7 A Web Server
-----------------

1. A handler pattern that ends with a slash matches any URL that has the pattern as a prefix.
2. Behind the scenes, the server runs the handler for each incoming request in a separate gorountine.
3. Race condition, two concurrent requests try to update the same variable at the same time.
4. One could use sync.Mutex.Lock and sync.Mutex.Unlock to avoid race conditions.
5. Four output streams, os.Stdout, a file, ioutil.Discard, http.ResponseWriter, share a common interface, io.Writer.
6. function literal, an anonymous function defined at its point of use.


1.8 Loose Ends
---------------

1. Control flow: if, for, switch. 
2. Switch cases do not fall through from one to the next as C-like languages. fallthrough overrides this behavior. default case is optional.
3. A switch does not need an operand. It is called tagless switch. It is equivalent to switch true.
4. Switch may include an optional simple statement--a short variable declaration, an increment or assignment statement, or function call--that can be used to set a value before it is tested.
5. The break and continue statements modify the flow of control. Statements may be labeled so that break and continue can refer to them.
6. The goto statement is intended for machine-generated code.
7. Named types: A type declaration makes it possible to give a name to an existing type.
8. Pointers: The & operator yields the address of a variable, and * operator retrieves the variable that the pointer refers to, but there is no pointer arithmetic.
9. Methods and interfaces: A method is a function associated with a named type. Interfaces are abstract types that treat different concrete types in the same way based on what methods they have, not how they are represented or implemented.
10. Packages: standard packages and community shared ones. Standard packages: https://golang.org/pkg, community shared packages: https://godoc.org. go doc tool.
11. Comments: documentation comments at the beginning of a program or package, comments before declaration of each function are important. They can be used by go doc or godoc to locate and display documentation.
12. Comments do not nest.