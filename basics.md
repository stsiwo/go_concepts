# Basics

## Modules (PMS)

- when run test ('go test'), go cli automatically install necessary deps and list to go.mod  
- setup: https://github.com/golang/go/wiki/Modules
- module name (at 'go mod init <module_name>) must match with git repository (e.g., github.com/stsiwo/repo)
      
- mode: 2 types
  1. module-aware mode: 
  
    * find versioned dependencies, and it typically loads packages out of the module cache, downloading modules if they are missing.
    * if you run 'go' command inside modules (a directory has go.mod file)

  2. GOPATH mode: ignores modules and it looks in vendor directories to find dependencies.

- vendoring: ??
ref: https://golang.org/ref/mod#vendoring
             
## syntax

### package = directory

- executable package
  - 'go install' create binary executable file
- utility package
  - 'go install' create archive file to help executable file (xxx.a file)

### import & export

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

### variable declaration and initialization

code)
```
var <variable_name> SomeType // this declare variable and initialize SomeType (create empty struct instance)
var <pointer_name> *SomeType // this declare a pointer to SomeType but it is not assigned yet so hold 'nil'
```

### tuple assignment

assign multiple variables at once in a single line

```
// swaping two variable
x, y = y, x
```

### function

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

### var keyword: 

it declares variables

it can be at pacakge or function level

if use the declaration with initializer (with "="), you can omit type for the variable when declare with initializer

```
var c = "hey"
```

### short variable declarations (":=")

this is only available inside function (not package level)

you can omit 'var' with implicit type.

```
k := 3 // assign int 3 to k
```

### patterns you find at code

```
1. var varName Type  // only declaration
2. var varName = new(Type) // declaration & initialization (you can omit Type at left-hand)
3. varName := xxx // only inside function and can omit 'var' and 'explicit type'
```

##### 'const': constant value

it can be character, string boolean, or numeric
it __cannot__ use short variable declarations (:=)

### nil: literal representation of zero for variables

### '_': skip a specific variable when receiving result from function 

```
for _, value =  range xxx { ... } // skip index but get value variable
```

### __defer__ keyword: 

A defer statement defers the execution of a function until the surrounding function returns.

### __pointers__: 
    
__IMPORTANT NOTE__: reason why use 'pointers' is to enable to modify properties of object (struct) at any point.

__asterisk__ indicate the variable hold a pointer
      
```
var p *int // p is a pointer
```
        
__ampersand (&)__: indicate the pointer's underlying value (not memory address)

```
(example 1)
var p = &j // generate pointer to variable j and assign the pointer to p
fmt.Println(*p) // print underlying value of p (pointer) called 'dereference'

(example 2) assume p is pointer (var p *int = xxx or p = &i )
  *p // represent underlying value of pointer p (actual value)
     // also the way to access underlying value via pointer is called 'dereference'
   
   p = &i // generate a pointer of i and assign to p
```

__different btw * vs &__: 

- __ * __ : __use in front of type (not variable)__


```
1. to store a memory address (not data) to variable

var p *int

2. also used to actual data via pointer variable (called dereference)

*p (p: pointer variable) => access to actual data
```

- __ & __ : __use in front of data variable (not type or pointer variable)__

```
1. it is used to access to memory address of the data variable

var p = &j (j: variable and assign a pointer of j to p)
```

- __pointers vs values__: 

* passing pointers in Go is often slower than passing value.

  this is because Go needs to perform the escape analysis to figure out if the variable should be stored on the heap or the stack. 

* use pointers when copying large structs. when struct has a lot of data, you should pass a pointer rather than values

* use pointers when you need to mutate. Otherwise, the variables cannot be mutated in another function.

* use pointers if your API uses a pointer receiver for the consistency (other parts also use pointer receiver everywhere)

* use pointers when signify true absence, which means that you want to know the value is really assigned to variables. if you use value variables, it always has a default zero value. But, if you use pointer variables, the default value is 'nil'.

ref: https://medium.com/@meeusdylan/when-to-use-pointers-in-go-44c15fe04eac

### __struct__: 

a collection of fields.

you can use comparable operator directly to structs if data type of all of the fields are comparable.

#### struct embedding and anonymous fields

embed a struct into another. when using struct embedding, you can omit the inner struct name when accessing fields in the inner struct. 

```
type Point struct {
	X, Y int
}

type Circle struct {
	Point // <- struct embedding
	radius int
}

var c Circle
c.X // you don't need to write c.Point.X (but you could write this though)
```

but when you need to instantiate the struct, you need to follow the hierarchy of the struct

```
c = Circle{Point{X: 1, Y: 2}, 3} // you have to write like this.
```

### __struct pointer__: 

to access value of field via pointer of struct

don't need to explicitly dereference // instead of *p.X, you can do p.X 

```
    type Vertex struct {
      X int
      Y int
    }
    v = Vertext{1,2}
    p = &v
    p.X = 3
    fmt.PrintLn(v) // { 3, 1 }
 ```

__if you need to modify the struct in a called (inner) function, you need to pass the pointer of the struct (not pass by value). 

### array

- declaration) letters := [4]string{"a", "b", "c", "d"}
- note) when specify the length, this means array
- predefined size
- [n]T
- array variable does not hold an pointer to the first array element. the array variable refers to the entire array.
-> this means when you send it as argument, a copy of its contents (not pointer)
- usually you should use slice rather than array (since array is inflexibile (fixed length))
- The in-memory representation of [4]int is just four integer values laid out sequentially:
    
- if a type of an array's element is comparable, the array itself is also comparable so that you can use comparable operators (e.g., ==, <=, >=) when comparing the array.

```
a := [2]int{1, 1}
b := [...]int{1, 1} // ... is used when you want to delegate the length later 

a == b // true
```
    
### slice: 
- (partial view of array) does not store any data, it just describes a section of an underlying array.
- a descriptor of an array segment. It consists of a pointer to the array, the length of the segment, and its capacity (the maximum length of the segment).

```
type SliceHeader struct {
      Data unitptr // a pointer to the backing array (esp the starting element of the array)
      Len int // length
      Cap int // capacity
}
```

- declaration) letters := []string{"a", "b", "c", "d"}
- note) when don't specify the length, this means slice
- dynamically-sized
- []T

```
ex)
  a[1:4] // a slice includes elements 1 through 3 of a (the last one is exclusive)
  // all slices have its underlying array and any change to those slices affect underlying array.
ex)
  []bool{1,2,3} 
  // create array with above element with size = 3 and then generate a slice that reference to that array    
```

- len(): length => the number of elements it contains
- cap(): capacity => the number from the first element of the slice and the last element of underlying array

```
ex)
s = int{1,2,3,4,5}
s = s[2:] // cap = 3 
 => {{1,2},3,4,5}
```

- extend: you can extend a slice based on the underly array if there is suffient capacity
- make(): create slice with all nil element with specified size (dynamically-sized arrays).


- append(): slice can be appended with values

if underlying array is too small to append the values, Go automatically re-size the underlying array.


the detailed logic is here. if the underlying array is too small to append a new element, it creates a double size of a new array and return a slice of the double-sized array to the client.


```
ex)
a = make([]int, 5) // a is a slice hold 5 'nil' elements
- slice can be nested; a slice can contains another slice
```
ref (official tutorial): https://tour.golang.org/moretypes/11
ref (official docs): https://go.dev/blog/slices-intro

- when you want to pass the array as argument, use slice instead of passing the pointer to the array
- this is because any change to the slice also affect to the original array
- so you don't need to pass the pointer to the array as argument.
- you can simply send a variable of the slice, if you need to modify the backing array inside the called function. 

```
a := [...]int{1, 2, 3, 4, 5} // backing array
yourFunc(a[:]) // create a slice and send it as argument.
// you can edit the backing array inside 'yourFunc' since the slice contains a pointer to the starting element, len(), cap()
```

- it is possible to modify the backing array when you pass the value of slice to a paramter in a function. 

```
func XXX(mySlice *[]int) // pass by refernce of a slice if you need to modify the slice (not array). this means changing a pointer, len, or cap of the slice
func XXX(mySlice []int) // pass by value of a slice if you don't need to modofy the slice.

* in both cases,  it still possible to modify the underline array.
```

- slices are not comparable.

##### escape analysis (golang feature)

- if a variable is still *reachable*, the variable __escapes__ from the function and it is stored in the heap.

- __reachable__: a variable is if still access from the program. e.g., returning a variable from a function or assigning a pointer of a local variable to a global variable. in this case, the variable is still reachable from the program.

- does not matter if a variable holds primitive or object (created with 'new') for Go to decide the variable should be escaped or not. for example, an object will stay in the stack if the variable becomes unreachable when the function is closed. __The idea that all objects goes into the heap is wrong in Go__.  

- in C/C++, you can't return a pointer from a function. 
- this is because those variables are stored in stack and if the function return the pointer and done, the function including those variables are deleted once the function is done. 
- in golang, you can return the pointer from a function because of this escape analysis.
- escape analysis check the pointer of a variable is shared among different functions or not. 
- if so, the variables are stored in Heap (not Stack) because the variable are shared among differernt functions
- if not (only shared on the single function), the variables are stored in Stack
- so you can return a pointer from a function. However, this also comes with cost
- cost: GC.
  - if the variable are stored in heap, GC must check the variable is used or no longer used. 
  - therefore, if there are a lot of those variables are in Heap, it increases the time for GC.
  - during GC, the program is getting slow. 
  - therefore, it might affect performance. 
  - the best choice is not to use those variables as much as possible.
        - conclusion: avoid to return pointer from function as much as possible
    
```
// option 1) create variable at top level function and send it by a pointer and you can mutate inside called function. then, you don't need to return the pointer to the calling function.

var a = A{xxxxx}

callingFunc(&a) // you can mutate the struct inside the 'callingFunc' so the change affect 'a'.
```
    
### function: is value too

- can be passed around just like other values as parameter.
- closures: a function value that references from outside its body. temp variable storage for outer function

```
func adder() func(int) int { // outer function
  sum = 0
  return func(x int) int { // inner function
    ... sum
    }
}
var test = adder() // a close is created for test varaible, this means test variable has dedicated sum variable (=0)
var test1 = adder() // same. a new closure is created for test1 variable and it gets own sum variable (=0)
```

### method: function with a special receiver argument

- __receiver argument__: allows to connect a function to a type

```
type Vertex struct {
  X, Y float64
}
func (v Vertex) abs() float64 { // (v Vertex): receiver argument
  return math.Sqrt(v.X*v.X + v.Y*v.Y) // can use v as belonged type
}
v = Vertex{3, 4}
v.abs() // now 'abs()' function belongs to Vertex struct
```
      
- __pointer receivers__: receiver arugment with pointer
      
- used to modify properties of the type it blongs

```
func (v *Vertex) Scale(f float64) { // (v *Vertex) => pointer receivers
  v.X = v.X * f
  v.Y = v.Y * f
}
v = Vertex{3, 3}
v.Scale(2) // v.X = 6 and v.Y = 6
// without pointer receivers (receiver argument) like '(v Vertex)', it just pass the copy of properties
// and above code (Scale(2)) does not affect properties of v (v.X = 3, and v.Y = 3)
```

### maps

in Go, map = hash map.

CRUD operation with the map is constant time on the average.

the key value type must be comparable but never use 'float' type. this is because 'float' might produce 'NaN' beause of a dubious operation (e.g., divided by 0, ...). and this always results in 'false' when comparison. that's why you should avoid using the 'float'.


### class

Go does not have classes.
    
### interface: define a set of methods

```
// define an interface
type MyInterface interface {
	say() string
	walk()
}

// define an implementation
type MyStruct struct { // don't need to use 'implements' keyword like other language this is good since decouping 
	name string
}

// implement its method with a pointer receiver
func (s *MyStruct) say() {
	fmt.Println("say " + s.name)
}

// implement its method with a pointer receiver
func (s *MyStruct) walk() {
	fmt.Println("walk with " + s.name)
}

func main() {
      // instantiate a struct
      var myStruct MyInterface = &MyStruct{name: "satoshi"}
	
      // call
      myStruct.say()
	myStruct.walk()
}

```

- no coupling to interface from detail implementation (like implements keyword). you don't need to explicitly code.

- __any type (primitive, struct, channel, function and so on) can implement an interface

```
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

```

ref: https://medium.com/golangspec/interfaces-in-go-part-i-4ae53a97479c

- __how to check the concrete type of an interface (type assertion) at runtime__
        
use val.(TYPE_NAME): return the typed value and error
          
```
v, err := val.(json.Marchaler)
```

- __pointer with interface__

an interface can store either a struct directly or a pointer to a struct.

```
var a Aer // Aer is interface
a = X{} // X is struct which implement Aer
a = &X{}
-> interface can hold struct itself or pointer to its struct so you don't need to use a pointer to interface
```
    
- __pointer to interface__
    
__don't use a poiner to interface__. pointers to interfaces are almost never useful (see: https://stackoverflow.com/questions/44370277/type-is-pointer-to-interface-not-interface-confusion)

should store pointers to struct in interface in order to modify the struct itself (e.g., a = &X{}); you can't modify the struct if you store the struct in interface (e.g., a = X{}) __when you pass by values__.


### unit testing

#### testify

#### mockery

- "mockery" package installation guide
  1. install the package
  2. set env path for mockery executable file
    2.1. make sure "GOPATH" is already set (export GOPATH="$HOME/go") as default and put this into /etc/profile (system wide) or ~/.prifle (user wide)
    2.2. add mockery executable file directory to "PATH" env 
      - export PATH="$PATH:$GOPATH/bin" (put this into 'profile' file)
  - reference: https://stackoverflow.com/questions/36083542/error-command-not-found-after-installing-go-eval

- mockery errors
  - go latest version (1.14) with 'nil pkg importing error of golang/x/net or built-in golang tools"
  - solution: use older version (when i switched to version 1.13.8, the error has gone!!!)
    
### concurrency

#### goroutine (GR): lightweight version of OS thread allows us to write concurrent program

- main() generate 'main goroutine" 
  - if you start another GR inside main(), it does not start immediately.
  - it wait until main GR finish its task (until execute all of code in main function)
  - but when main GR finished its job, Go close its program as main GR has done.
  - in order to run another GR, you need to tell scheduler to shcedule for the another GR
  - ex, you need to insert 'time.Sleep()' in main GR to make execution available to another GR
  
- __scheduler__:
  - time.Sleep(n): sleeps n time and won't be scheduled again for another n time
  - only non-sleeping GRs are candiate for scheduling.
```
    |main GR|
        |------------>|GR1|
        sleep(10 ms) -> | <- start GR1 since main GR in sleep
        |               | <- done of GR1
        |               
        |               
        | - done 10 ms
        | <- start executing the rest of main GR
       - although time.Sleep can give sheduler to switch GRs, it is not flexible in production. that's where 'channel' comes into play. make more strong relationship btw 2 GR (wait until another GR has done it job)
```

##### channel: a pipe which two goroutines can use to communicate each other; one GR send data through channel and the other GR receive the data via the channel

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

#### mutex: guarantee only single GR access to shared resource at a time 
  
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

  - one line error handling with if statement
  ex)
    if err := doStuff(); err != nil {
      // handle the error here
    }

## Naming Convension

### MixedCaps

#### visible outside the package

e.g., MxiedCaps

#### hidden outside the package

e.g., mixedCaps

### Interface

Methodname + er = InterfaceName

e.g., Reader, Writer, Formatter, CloseNotifier

### Shorter variable names (Unwritten Rule)

### single-letter identifier

used for local variables with limited scope.

```
// i is local variables and scope is limited inside the for loop
for i := 0; i < len(pods); i++ {
   //
}
```

### Shorthand Name

Shorthand names are recommended when possible as long as they are easy to understand by anyone reading the code for the first time. The wider the scope of use is, the more it needs to be descriptive.

```
pid // Bad (does it refer to podID or personID or productID?)
spec // good (refers to Specification)
addr // good (refers to Address)
```

### unique names

keep in their original form:

```
userID instead of userId
productAPI instead of productApi
```


###

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

#### Testing

- __go test -run TestSuite ./test/functional/users/ -v -count 1 -testify.m TestUserUpdatePutEndpointShouldUpdateDataExceptPasswordSuccessfully__: run a single test case in a test suite.


