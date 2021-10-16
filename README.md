# Go Concept

## Arcitecture

I apply [Clean Architecture](http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) for any project including my Go projects.
The main reason why use this architecture is to achieve [the separation of concerns](https://deviq.com/principles/separation-of-concerns). The general idea is that we should separate different concerns (e.g., Domain, UI, Application and so on) into its corresponding layer to establish well-organized systems. It has the following benefits:

- __testability__: easy to test each component/concern
- __independence__: easy to replace a component with a new one including external/internal dependencies (e.g., DB, Web Framework, any external API)
- __decouping__: reduce regression errors if one of the component/concern need to be changed. 

There is important rule to accomplish the the above benefits, which is called the Depednency Rule. The rule is pretty simple. The components (e.g., classes/structs) in the higher layer can use or have dependencies of interfaces in the lower layer, not vice versa. For instance, a component in the Infrastructure layer can have dependencies of components in the Application layer, but components in the Domain layer cannot have dependencies of the Application layer. 

I think that this rule strongly contributes to the independence of components/layers. For example, your team decided to use RDBMS such as MySQL for your project initially. After while you release the project, you need to scale up the system for some reason. Then, you team decided to use NoSQL to take an advantage of the scalability. If you apply the Clean Architecture, you can easily replace RDBMS with NoSQL.     

## Main Dependencies

- [__Go Module__](https://go.dev/blog/using-go-modules): used for the main package management system. a setup instruction 

- [__wire__](https://github.com/google/wire): used for the dependency management system.

---

## Technical Knowledge

### Basics

#### Modules (PMS)

- when run test ('go test'), go cli automatically install necessary deps and list to go.mod  
- [setup]: https://github.com/golang/go/wiki/Modules
- module name (at 'go mod init <module_name>) must match with git repository (e.g., github.com/stsiwo/repo)
      
- mode: 2 types
1. module-aware mode: 
  
- find versioned dependencies, and it typically loads packages out of the module cache, downloading modules if they are missing.
- if you run 'go' command inside modules (a directory has go.mod file)

3. GOPATH mode: ignores modules and it looks in vendor directories to find dependencies.

- vendoring: ??
ref: https://golang.org/ref/mod#vendoring
             
#### syntax

##### package = directory

- executable package
  - 'go install' create binary executable file
- utility package
  - 'go install' create archive file to help executable file (xxx.a file)

##### import & export

- __export if name of variables/functions/anything is capital__ (if not capital letter, not exported)
- this rule applies to __fields (e.g., variables)__ when different package

```
ex) 

// pkg aaa
type A struct {
  id
}

// pkg bbb
A.id <- NO (you can't do that) // in order to access, you need to use capital (e.g., Id) or provide getter method with capital (e.g., GetId)
```

##### variable declaration and initialization

code)
```
var <variable_name> SomeType // this declare variable and initialize SomeType (create empty struct instance)
var <pointer_name> *SomeType // this declare a pointer to SomeType but it is not assigned yet so hold 'nil'
```

##### function

- can return any number of values

```
func xxx(x, y string) (string, string) {
  return x, y // return x and y
}
```
  
- __naked return__: return statement without arguments

its return named return values.

```
  func split(sum int) (x, y int) {
    x = ...
    y = ...
    return   // return x and y as defined at type (x, y int)
  }
```

- var keyword: declares a list of variables

it can be at pacakge or function level

if use the declaration with initializer (with "="), you can omit type for the variable when declare with initializer

```
var c = "hey"
```

- short variable declarations (":=")

this is only available inside function (not package level)

you can omit 'var' with implicit type.

```
k := 3 // assign int 3 to k
```

- patterns you find at code

```
1. var varName Type  // only declaration
2. var varName = new(Type) // declaration & initialization (you can omit Type at left-hand)
3. varName := xxx // only inside function and can omit 'var' and 'explicit type'
```

- 'const': constant value
it can be character, string boolean, or numeric
it __cannot__ use short variable declarations (:=)

- nil: literal representation of zero for variables

- '_': skip a specific variable when receiving result from function 

```
for _, value =  range xxx { ... } // skip index but get value variable
```

- __defer__ keyword: A defer statement defers the execution of a function until the surrounding function returns.

- __pointers__: 
    
__IMPORTANT NOTE__: reason why use 'pointers' is to enable to modify properties of object (struct) at any point.

'*': there 2 ways to use

1. indicate the variable hold a pointer
      
```
var p *int // p is a pointer

```
        
2. indicate the pointer's underlying value (not memory address)

```
var p = &i // generate pointer to i and assign the pointer to p
```

        fmt.Println(*p) // print underlying value of p (pointer)
      ex) assume p is pointer (var p *int = xxx or p = &i )
        *p // represent underlying value of pointer p (actual value)
           // also the way to access underlying value via pointer is called 'dereference'
    - "&": generate a pointer to tis operand
      ex)
        p = &i // generate a pointer of i and assign to p
  
    # different btw * vs &
      - *: use in front of type (not variable)
        - the variable hold the memory address of the data
        - to store a memory address (not data) to variable
        - also used to actual data via pointer variable (e.g., *p (p: pointer variable) => access to actual data)
      - &: use in front of data variable (not type or pointer variable)
        - used to access to memory address of the data variable
  
  - struct: a collection of fields
  - struct pointer: access value of field via pointer of struct
    - don't need to explicitly dereference // instead of *p.X, you can do p.X 
  ex)
    type Vertex struct {
      X int
      Y int
    }
    v = Vertext{1,2}
    p = &v
    p.X = 3
    fmt.PrintLn(v) // { 3, 1 }
    
  - array: 
    - declaration) letters := [4]string{"a", "b", "c", "d"}
      - note) when specify the length, this means array
    - predefined size
    - [n]T
    - array variable does not hold an pointer to the first array element. the array variable refers to the entire array.
      -> this means when you send it as argument, a copy of its contents (not pointer)
    - usually you should use slice rather than array (since array is inflexibile (fixed length))
    
  - slice: partial view of array
    - declaration) letters := []string{"a", "b", "c", "d"}
      - note) when don't specify the length, this means slice
    - a slice consists of a pointer to the array (modifying element of a slice cause modifying the element of the original array)
    - dynamically-sized
    - []T
    ex)
      a[1:4] // a slice includes elements 1 through 3 of a (the last one is exclusive)
    - all slices have its underlying array and any change to those slices affect underlying array.
    ex)
      []bool{1,2,3} 
      -> create array with above element with size = 3 and then generate a slice that reference to that array
    - len(): length => the number of elements it contains
    - cap(): capacity => the number from the first element of the slice and the last element of underlying array
    ex)
      s = int{1,2,3,4,5}
      s = s[2:] // cap = 3 
       => {{1,2},3,4,5}
    - extend: you can extend a slice based on the underly array if there is suffient capacity
    - make(): create slice with all nil element with specified size
    - append(): slice can be appended with values
      - if underlying array is too small to append the values, Go automatically re-size the underlying array.
    ex)
      a = make([]int, 5) // a is a slice hold 5 'nil' elements
    - slice can be nested; a slice can contains another slice
    
    - when you want to pass the array as argument, use slice instead of passing the pointer to the array
      - this is because any change to the slice also affect to the original array
      - so you don't need to pass the pointer to the array as argument.
      
      - ALSO, when you pass the slice as argument, it pass a copy of slice (its content) so if you need to modify the slice inside the called function, you need to return the updated slice and assign at the calling function.
      
  - escape analysis (golang feature)
    - in C/C++, you can't return a pointer from a function. 
    - this is because those variables are stored in stack and if the function return the pointer and done, the function including those variables are deleted once the function is done. 
    - in golang, you can return the pointer from a function because of this escape analysis.
    - escape analysis check the pointer of a variable is shared among diff function or not. 
      - if so, the variables are stored in Heap (not Stack) because the variable are shared among diff function
      - if not (only shared on the single function), the variables are stored in Stack
      - so you can return a pointer from a function. However, this also comes with cost
      - cost: GC.
        - if the variable are stored in heap, GC must check the variable is used or no longer used. 
        - therefore, if there are a lot of those variables are in Heap, it increases the time for GC.
        - during GC, the program is getting slow. 
        - therefore, it might affect performance. 
        - the best choice is not to use those variables as much as possible.
        - conclusion: avoid to return pointer from function as much as possible
    
  - function: is value too
    - can be passed around just like other values as parameter.
    - closures: a function value that references from outside its body. temp variable storage for outer function
    ex)
      func adder() func(int) int { // outer function
        sum = 0
        return func(x int) int { // inner function
          ... sum
          }
      }
      var test = adder() // a close is created for test varaible, this means test variable has dedicated sum variable (=0)
      var test1 = adder() // same. a new closure is created for test1 variable and it gets own sum variable (=0)

  - method: function with a special receiver argument
    - connect function to type with a special receiver argument
    ex)
      type Vertex struct {
        X, Y float64
      }
      func (v Vertex) abs() float64 { // (v Vertex): receiver argument
        return math.Sqrt(v.X*v.X + v.Y*v.Y) // can use v as belonged type
      }
      v = Vertex{3, 4}
      v.abs() // 
      
    - pointer receivers: receiver arugment with pointer
      - used to modify properties of the type it blongs
      ex)
        func (v *Vertex) Scale(f float64) { // (v *Vertex) => pointer receivers
          v.X = v.X * f
          v.Y = v.Y * f
        }
        v = Vertex{3, 3}
        v.Scale(2) // v.X = 6 and v.Y = 6
        // without pointer receivers (receiver argument) like '(v Vertex)', it just pass the copy of properties
        // and above code (Scale(2)) does not affect properties of v (v.X = 3, and v.Y = 3)
        
    ex)
      func (v *Vertex) Scale(f float64)
  - Go does not have classes.
    
  - interface: define a set of methods
    - 'type InterfaceName interface { ... }' // define interface
    - 'var a InterfaceName' // declare inteface variable
    - 'v = DetailImpl{3, 4}' // instantiate 'DetailImpl' type
    - 'a = v' // assign detail impl 'v' to interface 'a' and 'v' must define methods defnied in 'a' inteface
    - no coupling to interface from detail implementation (like implements keyword). you don't need to explicitly code.
    
    - IMPORTANT NOTE: interesting points of 'interface'
      - any type (primitive, struct, channel, function and so on) can implement an interface
      ex)
        type TargetInterface interface {
          XXX()
        }
        
        type A ANY_TYPE 
        
        func (a A) XXX() {
          // do something
        }
        -> now type "A" implements 'TargetInterface' and you can use this 'A" for any function with this interface as argument.
        ex)
          func YYY(x TargetInterface) { ... }
          // you can do like 'YYY(a)' (a is object of A)
        
    - interface type check:
      - interface type check at run time
        - type assertion: val.(TYPE_NAME)
          - return the typed value and error
          ex)
            v, err := val.(json.Marchaler)
    
    - a pointer to interface:
      - an interface can store either a struct directly or a pointer to a struct.
      ex)
        var a Aer // Aer is interface
        a = X{} // X is struct which implement Aer
        a = &X{}
          -> interface can hold struct itself or pointer to its struct so you don't need to use a pointer to interface
    
      - don't use a poiner to interface
      - should store pointers to struct in interface in order to modify the struct itself (e.g., a = &X{}); you can't modify the struct if you store the struct in interface (e.g., a = X{})
### unit testing
  - use 'testify' and 'mockery'
    - "mockery" package installation guide
      1. install the package
      2. set env path for mockery executable file
        2.1. make sure "GOPATH" is already set (export GOPATH="$HOME/go") as default and put this into /etc/profile (system wide) or ~/.prifle (user wide)
        2.2. add mockery executable file directory to "PATH" env 
          - export PATH="$PATH:$GOPATH/bin" (put this into 'profile' file)
      - reference: https://stackoverflow.com/questions/36083542/error-command-not-found-after-installing-go-eval
    
      * mockery errors
        - go latest version (1.14) with 'nil pkg importing error of golang/x/net or built-in golang tools"
        solution: use older version 
          - when i switched to version 1.13.8, the error has gone!!!
          
### error handling
  - one line error handling with if statement
  ex)
    if err := doStuff(); err != nil {
      // handle the error here
    }
    
### concurrency
  # goroutine (GR): lightweight version of OS thread allows us to write concurrent program
    - main() generate 'main goroutine" 
      - if you start another GR inside main(), it does not start immediately.
        - it wait until main GR finish its task (until execute all of code in main function)
        - but when main GR finished its job, Go close its program as main GR has done.
        - in order to run another GR, you need to tell scheduler to shcedule for the another GR
        - ex, you need to insert 'time.Sleep()' in main GR to make execution available to another GR
    - scheduler:
      - time.Sleep(n): sleeps n time and won't be scheduled again for another n time
      - only non-sleeping GRs are candiate for scheduling.
      ex)
        |main GR|
            |------------>|GR1|
            sleep(10 ms) -> | <- start GR1 since main GR in sleep
            |               | <- done of GR1
            |               
            |               
            | - done 10 ms
            | <- start executing the rest of main GR
           - although time.Sleep can give sheduler to switch GRs, it is not flexible in production. that's where 'channel' comes into play. make more strong relationship btw 2 GR (wait until another GR has done it job)
  # channel: a pipe which two goroutines can use to communicate each other; one GR send data through channel and the other GR receive the data via the channel
    - only single data type per channel
    - only two GRs (not multiple (>2) GRs)
    - receiveing always block until channel gets data from another GR (regardless of buffer or non-buffer)
    - unbuffered channel:
      - sending (c <-data) will be blocked until receivng (data := <-c) 
      ex)
        |receiver| <- |channel| <- |sender|
        channel <- data // sending will be blocked until another GR receive (non-buffered)
    - buffered channel:
      - sending (c <-data) will be blocked when buffer is full
      ex)
        |receiver| <- |channel (buffer=4)| <- |sender|
        channel <-data (4 times) // this will block here until another GR receive; make channel empty
  # mutex: guarantee only single GR access to shared resource at a time
  
  
### Memory Management
  # Memstats properties:
    - Alloc: bytes of allocated heap objects.
    - TotalAlloc: cumulative bytes allocated for heap objects
      - does not decrease even when objects are freed
    - Sys: total btes of memory obtained from the OS
    - Lookups: the number of pointer lookups performed by the runtime
    - Mallocs: the cumulative count of heap objects allocated.
      - the number of live objects is Mallocs - Frees?
    - Frees: the cumulative count of heap objects freed
    - HeapAlloc: bytes of allocated heap objects
      - include reachable objects and unreachable objects
      - unreachable objects are targets of GC
      - increase when you create objects and decrease when unreachable objects are swept
      - sweeping occurs when GC cycles
    - Heapsys: bytes of heap memory obtained from the OS
      - the amount of virtual address space reserved for the heap (all available space)
    - HeapIdle: bytes in idle (unused) spans
      - could be returned to the OS or resued for heap allocation, or used for stack memory
      - HeapIdel - HeapReleased = the amount of memory that could be returned to the OS
    - HeapInuse: bytes in in-use spans
      - it has at least one object in them??
      - HeapInuse - HeapAlloc = the amount of memory that has been dedicated to particular size classes but is not currently being used.
    - HeapReleased: bytes of physical memory returned to the OS
    - HeapObjects: the number of allocated heap objects (interesting)
    - StackInuse: bytes in stack spans
      - no StackIdle because unused stack spans are returned to the heap and counted toward HeapIdle (becomes HeapIdle)
    - Stacksys: bytes of stack memory obtained from the OS
      - stackInuse + any memory obtained directly from the OS for OS thread stacks
    - MSpanInuse: bytes of allocated mspan structures.
    - MSpanSys: bytes of memory obtained from the OS for mspan
    - MCacheInuse: btes of allocated mcache structures
    - MCacheSys: bytes of memory obtained from the OS for mcache structures
    - BuckHashSys: bytes of memory in profiling bucket hash tables??
    
    reference: https://golang.org/pkg/runtime/#MemStats
     
## error handling:
  - 'go run' with multiple main package file
    - might end up with undefined variable error
    - this is because when you run 'go run' you need to provide all of file of main package
    - ex) go run main.go additional.go more-additional.go
    
  - implementing interface with pointer or data
    - when to implementing an interface, if you use the struct as data variable (not pointer), you need to implement the interface as data variable
    ex)
      func (c SomeStruct) SomeMethod { ... }
    - if you use the struct as pointer variable, you need to implement the interface as pointer variable
    ex)
      func (c *SomeStruct) SomeMethod { ... }
    
    
### testify (unit testing library)
  - run only single test in test suite
  cmd) go test -run TestSuite ./test/functional/users/ -v -count 1 -testify.m TestUserUpdatePutEndpointShouldUpdateDataExceptPasswordSuccessfully

### Commands

- __go build__: build target package. this depends on the package you try to build. if main package, it build the whole app and generate an executable. if non-main package (e.g., service package), it builds the non-main package, and then discard built file.

- __go get__: adjust the current module's dependencies without building packages.

- __go install__: recommended way to build and install packages in module mode.

#### Module Commands

- __go mod vendor__: copies of all packages needed to support builds and tests of packages in the main module in the /vendor folder.
- __go mod init <main_module_name>__: init go.mod 
- __go list -m all__: list all dependencies and sub dependencies (direct & indrect dependency)
- __go get <package_name>[@vx.x.x]__: upgrade a specific dependency with specific version (optionally)
- __go mod tidy__: safely remove removed dependencies from go.mod
- __go doc <package_name>[@vx.x.x]__:  check docs for specific package
- __go run <main_go>__:  compile and execute app

