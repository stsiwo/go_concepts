# goroutine

## textbook

https://www.oreilly.com/library/view/concurrency-in-go/9781491941294/ch04.html

##  basic

### preemption

n computing, preemption is the act of temporarily interrupting an executing task, with the intention of resuming it at a later time.

### instantaneous 

occurring or done in an instant or instantly.

if your stage is done quickly rather than taking a certain time, it is called instantaneous.

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

- closing: if a channel is closed, you cannot **send** a message any more and if you try, it will panic.
  - it is ok to receive values from closed channel. closing only affects sending (e.g., no more sending)
- it is not mandatory to close channels since it is GCed. but it is useful to tell to receivers that senders sent all data so no more data by close the channel. reference: https://stackoverflow.com/questions/8593645/is-it-ok-to-leave-a-channel-open

- **reange over channel**: you can use 'range' key for a channel.

```

for v := range channel {
	// every time a data is sending to the channel, this body is executed.
	// assuming that the channel is unbufferred.
}


// when the channel is closed, the for loop is done. this means that if you don't close the channel, it will ends up deadlock since 'range channel' block forever.
```

- **concatenation of receiving and sending**:

```
// use two arrows
takeStream <- <- valueStream

// = takeStream <- (<- valueStream)

// 1. receiving a data from 'valueStream'
// 2. sending the data to 'takeStream'

```

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

- you can implement fan out with for loop

```
numFinders := runtime.NumCPU()
finders := make([]<-chan int, numFinders)
for i := 0; i < numFinders; i++ {
    finders[i] = TargetStageToBeFannedOut(done, PreviousInputs)
}
```

__fan in__: a process of combineing multiple results into a single channel

```
fanIn := func(
    done <-chan interface{},
    channels ...<-chan interface{},
) <-chan interface{} { 1
    var wg sync.WaitGroup 2
    multiplexedStream := make(chan interface{})

    multiplex := func(c <-chan interface{}) { 3
        defer wg.Done()
        for i := range c {
            select {
            case <-done:
                return
            case multiplexedStream <- i:
            }
        }
    }

    // Select from all the channels
    wg.Add(len(channels)) 4
    for _, c := range channels {
        go multiplex(c)
    }

    // Wait for all the reads to complete
    go func() { 5
        wg.Wait()
        close(multiplexedStream)
    }()

    return multiplexedStream
}
```

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

### or-done channel

??

### tee channel

a stage which split values from a channel and separate them into two channels.

it works like tee command in Linux (see [this](https://www.geeksforgeeks.org/tee-command-linux-example/)). for example, input sends to both a file and stdout.

use this when you need to send values from a channel to two different channels. the same value is sent to both channels.

```

| channel A | --> | tee | -> | channel B |
			  -> | channel C |

```

### bridge-channel

convert a channel of channels to a channel. 

- **a channel of channels**: a channel which contains a series of channel as data

```
channel 1 contains channel 1.a, 1.b, and 1.c
---------------------------------------
channel 1

|-------------||-------------||-------------|
| channel 1.a || channel 1.b || channel 1.c | 
|    |A|      ||     |B|     ||     |C|     |
|-------------||-------------||-------------|

---------------------------------------

==> convert into a simple channel

---------------------------------------
channel 2

|-------------||-------------||-------------|
|    |A|      ||     |B|     ||     |C|     |
|-------------||-------------||-------------|

---------------------------------------


```

### queuing 

accept work even if it is not ready yet.

in channels, you can use buffered channels.

**you should use queuing as the last resort** since Adding queuing prematurely can hide synchronization issues such as deadlocks and livelocks.

common mistake is that queuing boosts performance. this is not true. **Queuing will almost never speed up the total runtime of your program; it will only allow the program to behave differently**.

the main use case of queue is to **reduce blocking time of a stage (a.k.a., decoupling a stage from other stages) so that the stage keep available to accept a new job**.

when to use queue to improve performance:

1. If batching requests in a stage saves time.

ex)

chuncking: accumulate something until a certain amount and process those at once (e.g., input/output buffer: rather than processing every single bit/byte, use buffer to accumulate the input/output until enough to process. it is more efficient. 


2. If delays in a stage produce a feedback loop into the system.

- feebback loop (a.k.a., negative feedback loop, death spiral, downward spiral):

If the efficiency of the pipeline drops below a certain critical threshold, the systems upstream from the pipeline begin increasing their inputs into the pipeline, which causes the pipeline to lose more efficiency, and the death-spiral begins. Without some sort of fail-safe, the system utilizing the pipeline will never recover.

If the rate of ingress exceeds the rate of egress, your system is unstable and has entered a death-spiral.

ex) 

one of stage cannot handle input smoothly so the previous stage acceping more input and waiting for the stage handling the current job to send a new one from the previous stage.

so queuing should be implemented either: 

1. At the entrance to your pipeline.
2. In stages where batching will lead to higher efficiency.

you should not add queue where the stage handling an expensive computational task. Why?

to understand this, you need to know how to calculate throughput of your pipeline.

in order to do this, you need **Little's Law** 

```
Little's Law formula:

L=λW where:

L: the average number of units in the system.
λ: the average arrival rate of units.
W: the average time a unit spends in the system.
```

this equation only applies if the system is **stable**

- **stable** system: the rate of input (a.k.a., **ingress**) and the rate of output (a.k.a., **egress**) is equal.
- **unstable**: if the above condition does not apply. esp, 
	- if ingress exceeds egress, it causes **death spiral (feedback loop)**.
	- if egress exceeds ingress, you have not utilize your system completely in terms of efficiency).

still don't understand how to use **Little's Law** in practice, you should see the textbook.

### The Context Package

in concurrent program, it is often necessary to preempt (instantaneous) operation because of timeouts, cancellation, or failure of another portion of the system.

we used to use 'done' channel, but it's somewhat limited.

that's where the context package comes in. they can communicate extra info alongside the simple notification to cancel: why the cancellation was occuring, or whether or not our function has a deadline bhyy which it needs to complete.

* **Deadline**: indicate if a goroutine will be canceled after a certain time. this is an absolute time like the machine's clock advances past the given deadline.
* **Timeout**: relative duration when you want to cancel it. e.g., 10sec timeout means the goroutine cancels after 10sec.
* **Err**: will return non-nil if the goroutine was canceled. 
* **Value**: pass information along to child goroutine.
* **Background**: simply return empty Context.
* **TODO**: return empty Context. but this is not meant for use in production. use this when you need a placebolder for when you don't knwo which Context to utilize, or if you expect your code to be provided with a Context, but the upstream code hasn't yet furnished one.


* **call-graph**: a graph starting from main goroutine to child goroutines.

two primary purpose of the context package:

1. to provide an API for canceling branches of your call-graph.
2. to provide a data-bag for transporting request-scoped data through your call-graph.

#### cancellation

* a goroutine's parent may want to cancel it.
* a goroutine may want to cancel its children.
* any blocking operations within a goroutine need to be preemptable (might be interrupted and resume at the later time) so that it may be canceled.


it is important that you do not make the context be one of your member variables. Instead, always pass instances of Context into your functions. this is for the following reasons:

1. any child goroutine does not really know about what kind of cancelation (e.g., Deadline, Timeout, Cancel). => decouple the parent goroutine which does cancelation and the children goroutines which are cancelled.
2. any child goroutine can also build its own context type (e.g., Deadline, Timeout, Cancel) without affecting the parent goroutine or any other child goroutines.
