# Concurrency

## Thread
* sequence of instructions that execute sequentially.
* programmer has no control how the threads will get to execute
* a need for synchronisation arises

## Solutions
1. Shared variables
2. Message passing

## Message passing
* synchronisation can be done using message passing between the threads

## Shared variables
* shared memory can be a way to synchronise threads

## Semaphores
* like an integer with following differences :
	1. can initialise value to any integer
	2. only increment or decrement
	3. can not read the current value
	4. if after decrement, value becomes negative the thread blocks itself
	5. on increment, one of the waiting threads gets unblocked

### Implications:

* there is no way a thread can know whether it will get blocked before decrementing
* after increment, if another thread gets woken up, both the threads continue running concurrently

## Syntax
* **Increment** - signal
*  **Decrement** - wait

## Advantages of Semaphores
* Impose deliberate constraints
* Solutions are clean and organised
* Can be implemented efficiently

## Serialisation
* One thread signals the other thread to indicate that something has happened
* Can help in enforcing serialisation (one portion of code in one thread executes before other portion of code in another thread)
### Thread A
```python
statement a1
sem.signal()
```
### Thread B
```python
sem.wait()
statement b1
```
Order of execution: a1<b1

## Rendezvous
* Both the threads wait for each other at a point in their individual codes
### Thread A
```python
statement a1
sem1.signal()
sem2.wait()
statement a2
```
### Thread B
```python
statement b1
sem2.signal()
sem1.wait()
statement b2
```
### Initialisation
```
sem1=Semaphore(0)
sem2=Semaphore(0)
```
Order of execution:
a1<b2 & b1<a2

### Deadlock
* Can happen if both the threads arrive and wait before signalling

## Mutual Exclusion
* guarantee that only one thread accesses the shared variable at a time
* behaves like a token - the one who has can only access the shared variable

### Thread A
```
mutex.wait()
count=count+1
mutex.signal()
```
### Thread B
```
mutex.wait()
count=count+1
mutex.signal()
```

### Initialisation
```
mutex=Semaphore(1)
```

## Critical section
* code that needs protection - exclusive access at a time

## Multiplex
* allows multiple process to be in the critical section

### Threads
```
mutex.wait()
# critical section
mutex.signal()
```

### Initialisation
```
mutex=Semaphore(n)
```

## Barrier
* all threads reach rendezvous
* critical point to be executed concurrently by all the threads
* no thread executes critical section until after all threads have executed rendezvous

### Threads
```
mutex.wait()
	count+=1
mutex.signal()
if count==n:
	barrier.signal()
barrier.wait()
barrier.signal()

# critical point
```

### Initialisation
```
barrier=Semaphore(0)
mutex=Semaphore(1)
n=number of threads
count=0
```

## Reusable barrier
* set of cooperating threads
* perform a series of steps in loop
* synchronise after each step

### Threads
```
mutex.wait()
	count+=1
	if count==n:
		turnstile2.wait()
		turnstile1.signal()		
mutex.signal()

turnstile1.wait()
turnstile1.signal()

# critical point

mutex.wait()
	count-=1
	if count==0:
		turnstile1.wait()
		turnstile2.signal()
mutex.signal()

turnstile2.wait()
turnstile2.signal()
```

### Initialisation
```
turnstile1=Semaphore(0)
turnstile2=Semaphore(1)
mutex=Semaphore(1)
count=0
```

## Preloaded turnstile
* turnstile forces threads to go sequentially
* context switching overhead
* open turnstile so that correct number of threads can pass through

### Threads
```
mutex.wait()
	count+=1
	if count==n:
		turnstile.signal(n)
mutex.signal()

turnstile.wait()

mutex.wait()
	count-=1
	if count==0:
		turnstile2.signal(n)
mutex.signal()

turnstile2.wait()
```

## Queue
* to represent a queue

### follower thread
```
leaderQ.signal()
followerQ.wait()
# dance
```

### leader thread
```
followerQ.signal()
leaderQ.wait()
# dance
```

### Initialisation
```
followerQ=Semaphore(0)
leaderQ=Semaphore(0)
```

## Exclusive queue
* one leader and one follower only dance at a time
* similar to a couple

### leader thread
```
mutex.wait()
if followers>0:
	followers-=1
	followerQ.signal()
else:
	leaders+=1
	mutex.signal()
	leadersQ.wait()
dance()
redezvous.wait()
mutex.signal()
```

### follower thread
```
mutex.wait()
if leaders>0:
	leaders-=1
	leadersQ.signal()
else:
	followers+=1
	mutex.signal()
	followersQ.wait()
dance()
rendezvous.signal()
```

### Initialisation
```
leaders=followers=0
mutex=Semaphore(1)
leadersQ=Semaphore(0)
followersQ=Semaphore(0)
rendezvous=Semaphore(0)
```

## Producer-consumer problem
* producer thread creates an event object and adds to event buffer
* consumer thread take event out of the buffer and process them
* buffer must be accessed exclusively
* consumer should wait until producer produces an event in case buffer is empty

### Producer thread
```
event=create_event()
mutex.wait()
	buffer.add(event)
mutex.signal()
items.signal()
```

### Consumer thread
```
items.wait()
mutex.wait()
	event=buffer.get()
mutex.signal()
process_event(event)
```

### Initialisation
```
mutex=Semaphore(1)
items=Semaphore(0)
```

## Finite buffer

### Consumer thread
```
items.wait()
mutex.wait()
	event=buffer.get()
mutex.signal()
spaces.signal()

process_event(event)
```

### Producer thread
```
event=create_event()
spaces.wait()
mutex.wait()
	buffer.add(event)
mutex.signal()
items.signal()
```

### Initialisation
```
spaces=Semaphore(N)
mutex=Semaphore(1)
items=Semaphore(0)
```

## Readers
* data structure, database or file system is read and modified by concurrent threads

### Reader thread
```
mutex.wait()
	readers+=1
	if readers==1:
		roomEmpty.wait()
mutex.signal()
# read
mutex.wait()
	readers-=1
	if readers==0:
		roomEmpty.signal()
mutex.signal()
```

### Writer thread
```
roomEmpty.wait()
# write
roomEmpty.signal()
```

### Initialising
```
readers=0
mutex=Semaphore(1)
roomEmpty=Semaphore(1)
```

* writer can starve

## No starve readers-writers
* if a writer arrives, no new reader is allowed to read before the writer writes

### Reader thread
```
writer.wait()
writer.signal()	

mutex.wait()
	readers+=1
	if readers==1:
		roomEmpty.wait()
mutex.signal()
# read
mutex.wait()
	readers-=1:
		roomEmpty.signal()
mutex.signal()
```

### Writer thread
```
writer.wait()
	roomEmpty.wait()
writer.signal()
# write
roomEmpty.signal()
```

### Initialisation
```
writer=Semaphore(1)
roomEmpty=Semphore(1)
readers=0
```

## Priority writers-readers

### Reader thread
```
noReaders.wait()
	readSwitch.lock(noWriters)
noReaders.signal()
# write
readSwitch.lock(noWriters)
```

### Writer thread 
```
writeSwitch.lock(noReaders)
	noWriters.wait()
		# read
	noWriters.signal()
writeSwitch.unlock(noReaders)
```

### Initialisation
```
readSwitch=Lightswitch()
writeSwitch=Lightswitch()
noReaders=Semaphore(1)
noWriters=Semaphore(1)
```

## No-starve mutex
* issue of thread starvation
* requirement of bounded waiting

### Morris's algorithm
```
mutex.wait()
	room1+=1
mutex.signal()

t1.wait()
	room2+=1
	mutex.wait()
	room1-=1
	if room1==0:
		mutex.signal()
		t2.signal()
	else:
		mutex.signal()
		t1.signal()

t2.wait()
	room2-=1
	# critical section
	if room2==0:
		t1.signal()
	else:
		t2.signal()
```


### Initialisation
```
room1=room2=0
mutex=Semaphore(1)
t1=Semaphore(1)
t2=Semaphore(0)
```