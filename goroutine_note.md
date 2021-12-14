# goroutine

##  basic

### communicating sequential processing (CSP)

a model how diffferent goroutine communicate each other (sending/receiving data)

### shared memory multithreading

a traditional model how different threads communicate each other.

this model is commonly used in mainstream programming language.

## options

in Go, you can achieve concurrency with the following techniques:

1. channel
2. mutex (mutual exclusive) locking
3. immutable data structure
4. Data protected by confinement; use non-concurrently coding and manage the concurrent situation to avoid the usage of concurrent code at all. this might be hard to achieve.

### goroutine (GR): lightweight version of OS thread allows us to write concurrent program

1. GR is not garbage collected by the runtime so make sure you clean up all GRs you generated.

use can use cancelation (e.g., 'done' channel) and share it with other goroutines. if you want to stop the program, you can use the 'done' channel to stop other goroutines too.

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

### channel: a pipe which two goroutines can use to communicate each other; one GR send data through channel and the other GR receive the data via the channel

- only single data type per channel
- only two GRs (not multiple (>2) GRs)
- receiveing always block until channel gets data from another GR (regardless of buffer or non-buffer)
- unbuffered channel:
- buffered channel:
  - sending (c <-data) will be blocked when buffer is full
  ex)
    |receiver| <- |channel (buffer=4)| <- |sender|
    channel <-data (4 times) // this will block here until another GR receive; make channel empty

- closing: if a channel is closed, you cannot send a message any more and if you try, it will panic.
- it is not mandatory to close channels since it is GCed. but it is useful to tell to receivers that senders sent all data so no more data by close the channel. reference: https://stackoverflow.com/questions/8593645/is-it-ok-to-leave-a-channel-open

### pipeline

a series of stages connected by channels, where each stage is a group of goroutines running the same function.

| first stage | --- multiple inbound/outbound channels --- | 2nd stage | --- multiple inbound/outbound channels --- | 3rd stage | ... | the last stage |

this is a another way to form abstraction esp in communication btw GRs.

benefits:

1. you can combine each stage as you like and create your own pipeliine to accommodate your requirement.
2. each stage executes its task concurently; this means each stage just wait for the input and be able to produce out independently. (a.k.a., ramification; devide tasks into smaller one and each GR is in charge of each task)

```

main GR 
	-> stage 1
	-> stage 2 // wait for stage 1 sends out its data to me
	-> stage 3 // wait for stage 2 sends out its data to me

```

implementation:

```
// best practice for constructing pipline

// overview

| Main GR | => | Generator | => | stage 2 | => | stage 3 | => ... => | Main GR |

// sample code

generator := func(done <-chan interface{}, integers ...int) <-chan int {
    intStream := make(chan int)
    go func() {
        defer close(intStream)
        for _, i := range integers {
            select {
            case <-done:
                return
            case intStream <- i:
            }
        }
    }()
    return intStream
}

multiply := func(
  done <-chan interface{},
  intStream <-chan int,
  multiplier int,
) <-chan int {
    multipliedStream := make(chan int)
    go func() {
        defer close(multipliedStream)
        for i := range intStream {
            select {
            case <-done:
                return
            case multipliedStream <- i*multiplier:
            }
        }
    }()
    return multipliedStream
}

add := func(
  done <-chan interface{},
  intStream <-chan int,
  additive int,
) <-chan int {
    addedStream := make(chan int)
    go func() {
        defer close(addedStream)
        for i := range intStream {
            select {
            case <-done:
                return
            case addedStream <- i+additive:
            }
        }
    }()
    return addedStream
}

done := make(chan interface{})
defer close(done)

intStream := generator(done, 1, 2, 3, 4)
pipeline := multiply(done, add(done, multiply(done, intStream, 2), 1), 2)

for v := range pipeline {
    fmt.Println(v)
}

```

what if one of stage is computationally expensive? is this rate-limit our entire pipeline?

the solution is that you can use [**fan-out**](#fan-out-fan-in) and **fan-in** pattern to tackle this issue:)

### stages

def) something which takes data in, perform transformation on it, and send the data to another stage.

```
// example of a stage

// data in 
multiply := func(values []int, multiplier int) []int {
    multipliedValues := make([]int, len(values))
    
    // transform
    for i, v := range values {
        multipliedValues[i] = v * multiplier
    }
    
    // send the data out
    return multipliedValues
}
```

each stage has its own responsibility (i.e.,, sepration of concerns)

there are two type of what stages can perform: 

1. batch processing: operate chunks of data all at once instead of one discrete value at a time (e.g., slices)
2. stream processing: operate a single data at a time.

#### Generator

def) a stage which is usually the first one on a pipeliine to convert discrete values into a serie of values (e.g., slices) on a single channel

### Fan-Out Fan-In 

__fan out__: a process of starting multiple GRs to handle inputs from your pipeline

__fan in__: a process of combineing multiple results into a single channel

### buffered channels

- sending (c <-data) will be blocked when buffer is full

```
|receiver| <- |channel (buffer=4)| <- |sender|

channel <-data (4 times) // this will block here until another GR receive; make channel empty
```

### unbuffered channels

- sending and receiving are __synchronized__ (e.g., the upstream goroutine will be blocked until the downstream one receives)

- sending (c <-data) will be blocked until receivng (data := <-c) 

```
|receiver| <- |channel| <- |sender|
channel <- data // sending will be blocked until another GR receive (non-buffered)
```

### sync.WaitGroup

used when you want to make sure all goroutines launched are done. also, when you want to make sure sending to a channel is done before calling close. 

how it works)

```
func main() {

	// 1. initialize 
    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        
	// 2. increment value for every goroutine launched
	wg.Add(1)

	// avoid the closure trap
        i := i

        go func() {
		// make sure to call 'wg.Done()' if the task is done on each goroutine
            defer wg.Done()
            worker(i)
        }()
    }

	// this waits until all goroutines launched are done
    wg.Wait()
}
```

### Explict Cancellation

how a GR in the downstream tell a GR in the upstream to stop any incoming message anymore?

__solution__: create a channel on the downstream and share it with all GR (e.g., upstream GRs)

```
// simple pattern to close GR without leaking. (a.k.a., Preventing Goroutine Leaks pattern)

func main() {

	done := make(chan interface{})
	defer close(done) // this is important when this defer function is called, the child GR is also close since 'case <- done:' clause is called.
	
	go func() {
	
		for {
		
			select {
			
			case <- done: // when close the 'done' channel, this code is executed, which means that this child GR is closed automatically so that no GR leaks.
				return 
			case ...:
				...
			
			}
		}
	}
}
```

### Unidirectional channel

receive-only/send-only channels (e.g., separate the two responsibilities)

### Goroutine leak

goroutine stuck because of waiting for sending or receiving message via a channel forever. thsi results in that the GR is not GCed.

__solution__: make sure all GRs are terminated themselves when no longer needed.

### Race Condition

it occures when multiple GRs access the same data.

since OS thread scheduling algorithm swap the active thread any time, there is no way for you can guaranteen the order of which thread access first. 

solution: you need to use locking the resource.

detail: [here](https://stackoverflow.com/questions/34510/what-is-a-race-condition)

### Data Race

occurs when one thread accesses a mutable object while another thread is writing to it.

solutions:

1. make the shared data read-only
2. use channels (don't communicate by sharing memory. share memory by communicating)
3. mutual exclusion (only one GR access the shared data at a time)

### semaphore 

A semaphore controls access to a shared resource through the use of a counter. If the counter is greater than zero, then access is allowed. If it is zero, then access is denied. What the counter is counting are permits that allow access to the shared resource. Thus, to access the resource, a thread must be granted a permit from the semaphore.

### mutex (mutual exclusion): guarantee only single GR access to shared resource at a time 
  
- __Mutex__: normal mutex locking

only a single GR acquires a lock and the others cannot access unless the 1st GR release the lock

- __RWMutex__: read/write mutex locking

either multiple readers or a single writer

### race detector 

Go has built-in flag to detect any concurrency mistake you made. 

use '-race' flag when build or test command

### Goroutine vs Thread

__threads__:

- fix size of memory (e.g., 2MB)
- context switching is slow (switch from active thread to another thread is relative expensive like save/restore data in memory)
- has an identity for each thread (e.g., so that it can has its own thread local storage)

__Groutines__:

- flexible size of memory (e.g., starting 2KB and expand or shrink as needed)
- M:N scheduling is faster than the OS context switching
- does not have an identity 


### M:N scheduling

Go has iwo own schduler that map M (Goroutines) to N (OS threads).

you can think of the Go scheduler in a Go program as OS scheduler on the OS

Go scheduler is invoked by certain Go language constructs (e.g., Sleep, blocks, mutex stuff). so, Goroutine switching is invoked only when needed.

the OS scheduler is invoked periodically.
  
Go scheduler is much cheaper than the OS scheuler

__GOMAXPROCS__: to check how many OS threads are actively running the Go code simulateneously.
  
## practical stuff

### send a channel as arugment 

there are 2 types to enforce type error for channel when sending it as argument: 

```
func send(ch chan<- []byte) { 

	... <- ch // compiler error
	ch <- data // ok

} // this only allows 'ch' to sending. if you do receiving with this ch, it generate compiler error

func receive(ch <-chan []byte) { 

	... <- ch // ok
	ch <- data // compiler error

} // this only allows 'ch' to receiving. if you do sending with this ch, it generate compiler error

```

ref: https://golangbyexample.com/channel-function-argument-go/

### don't use multiple unbufferred channels consequently

```

channel1 := make(chan string)
channel2 := make(chan string)

for i := 0; i < 2; i++ {
  go func() {
    
    channel1 <- "A" // stuck since the channel is full
    // never reach here and below
    channel2 <- "B"
  
  }()
}

...

```

solution) use bufferred channel

```

channel1 := make(chan string, 2)
channel2 := make(chan string, 2)

for i := 0; i < 2; i++ {
  go func() {
    
    channel1 <- "A" // now it works since the channel is not full yet
    channel2 <- "B"
  
  }()
}

...

```

### for-select pattern

this is useful when using channel.  Looping infinitely waiting to be stopped.

```
for {
	select {
	case s := <- channel:
		// do something with received data from the channel and continue
	case <- done: // if done channel receives data, done this for loop.
		return 
	}
}

```

### error handling

main GR should take a control of what to do when any child GR produces errors. 

so error handling (e.g., if err := ...) should be in main GR, and any child GR should return the error to the main GR. 

create a struct where error embeds in and return the struct as a result of the child goroutine so that it looks like a usual synchronous function. 

```
```
