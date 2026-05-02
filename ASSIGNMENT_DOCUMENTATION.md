# Assignment 3 - Complete Documentation

**Student Name**: [Mohammed Zamail Alsharif]  
**Student ID**: [444050659]  
**Date Submitted**: [5/2/2026]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [ https://drive.google.com/file/d/1ACr2qJNGTJdUFEZa7Ag90c8X_aIHwuz0/view?usp=sharing ]

**Video filename**: `444050659_Assignment3_Synchronization.mp4`

**Verification**:
- [ Y ] Link is accessible (tested in incognito mode)
- [ Y ] Video is 3-5 minutes long
- [ Y ] Video shows code walkthrough and commits
- [ Y ] Video has clear audio
- [ Y ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)
Y= YES 
---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [May 1, 2026, 4:00 PM]
**What I implemented**: 
Project setup, repository cloning, and initializing the simulation by configuring the studentID variable to 444050659.

**Challenges encountered**: 
None
**How I solved it**: 
Used the VS Code Source Control tab to stage the initial ID change and successfully pushed the first commit to the remote repository.

**Testing approach**: 
Ran the starter code to verify that the random seed generated a specific and consistent number of processes and time quantum based on my ID.
**Time spent**: 
~~ 20 Mint 
---

### Entry 2 - [May 1, 2026, 5:30 PM]

**What I implemented**: 
Task 1 - Added a ReentrantLock in SharedResources to protect shared counter variables (contextSwitchCount, completedProcessCount, totalWaitingTime).

**Challenges encountered**: 
Structuring the lock mechanism to guarantee that the lock is released even if an exception occurs during variable modification, preventing potential deadlocks.

**How I solved it**: 
Implemented strict try-finally blocks inside incrementContextSwitch(), incrementCompletedProcess(), and addWaitingTime(), placing lock.unlock() strictly in the finally clause.

**Testing approach**: 
Reviewed the code structure visually to ensure no paths bypass the finally block before committing the changes.

**Time spent**: 
~~ 50 min .

---

### Entry 3 - [May 2, 2026, 5:15 AM]

**What I implemented**: 
Task 2 - Secured the executionLog ArrayList within the logExecution method using the previously instantiated ReentrantLock.

**Challenges encountered**: 
The ArrayList structure is not inherently thread-safe, leading to potential ConcurrentModificationException when multiple processes attempt to log data simultaneously.

**How I solved it**: 
Reused the existing mutual exclusion lock by calling lock.lock() before the add() operation and lock.unlock() in the subsequent finally block.

**Testing approach**: 
Ran the simulation with multiple concurrent threads to ensure the application completes without throwing list modification exceptions

**Time spent**: 
~~ 35 min
---

### Entry 4 - [May 2, 2026, 1:30 PM]

**What I implemented**: 
ask 3 - Integrated a binary Semaphore (initialized with 1 permit) to control the CPU access within the run() and runToCompletion() methods of the Process class.

**Challenges encountered**: 
Handling the InterruptedException forced by the acquire() method without compromising the integrity of the subsequent try-finally block that handles the release() logic.

**How I solved it**: 
Separated the acquire() call into its own try-catch block before the main execution block. This architectural decision ensures release() is never incorrectly called if acquire() fails.

**Testing approach**: 
Executed the simulation to verify that the console output correctly shows processes running sequentially on the CPU without overlapping quantum executions.
**Time spent**: 
~~ 75 min .
---

### Entry 5 - [Date, Time]
**What I implemented**: 

**Challenges encountered**: 

**How I solved it**: 

**Testing approach**: 

**Time spent**: 

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

[ The first race condition occurs with the primitive counters, specifically contextSwitchCount++. The shared resource affected is the contextSwitchCount integer. Concurrent access is a problem because the increment operation is not atomic (it involves reading the value, adding one, and writing it back); if two threads read the value simultaneously before either writes it back, a "lost update" incorrect behavior occurs, leading to an inaccurate total count.
The second race condition involves the executionLog.add(message) method. The shared resource is the executionLog (an ArrayList). Because ArrayList is not thread-safe, concurrent structural modifications by multiple threads corrupt its internal array management. This could lead to a ConcurrentModificationException crashing the program, or silent data corruption where log entries are overwritten or nullified. ] 

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

[A ReentrantLock is a mutual exclusion (mutex) mechanism owned by the specific thread that acquires it, ensuring only one thread accesses a critical section at a time. A Semaphore is a signaling mechanism based on permits that controls access to a shared resource pool and doesn't inherently have "ownership" by a single thread.
In my code, I used a ReentrantLock in the SharedResources class to protect primitive counters and the ArrayList, as these simply required strict, quick mutual exclusion during updates. I used a Binary Semaphore (initialized with 1 permit) in the Process class to control access to the simulated CPU. This acts as a gatekeeper, ensuring only one simulated process thread can "execute" its quantum at a given time, accurately modeling a single-core CPU scheduler.]

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

[A deadlock is a situation where two or more threads are permanently blocked, each waiting for a resource held by the other, bringing the application to a halt. Two common prevention techniques are "Lock Ordering" (ensuring all threads request multiple locks in the exact same sequence) and "Proper Resource Release" (guaranteeing locks are freed even if an operation fails).
To prevent deadlocks in my code, I strictly implemented proper resource release using the try-finally block pattern. Whenever a ReentrantLock.lock() or Semaphore.acquire() was called, the corresponding unlock() or release() was explicitly placed inside a finally block. This guarantees that if a thread encounters an InterruptedException or a runtime error while executing, it will always release its hold on the resource, allowing other waiting threads to proceed.]

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

[ For Task 1, I chose to implement a coarse-grained locking approach by using ONE single ReentrantLock (lock) for all three counters. I made this choice for architectural simplicity and to minimize the overhead of managing multiple lock objects in a straightforward simulation.
The trade-off is between performance and complexity. Coarse-grained locking is easier to implement and less prone to logic errors, but it reduces concurrency (a thread updating the context switch counter blocks another thread trying to update the completed process counter). Fine-grained locking requires more memory and careful management but maximizes concurrency. Given that the three counters are completely independent, a fine-grained approach (one distinct lock per counter) would inherently provide better concurrency, as processes could update different counters simultaneously without unnecessarily blocking one another. However, coarse-grained was sufficient for the scale of this specific assignment. ]

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: 
contextSwitchCount, completedProcessCount, and totalWaitingTime 

**Why they need protection**: 
These are shared global state variables accessed and modified by multiple concurrent process threads. Modifications (like count++ or time += value) are not atomic operations. Without protection, a race condition occurs where simultaneous reads and writes cause "lost updates", leading to incorrect final statistics.

**Synchronization mechanism used**: 
Mutual exclusion using ReentrantLock.

**Code snippet**:
```java
public static void incrementContextSwitch() {
        lock.lock();
        try {
            contextSwitchCount++;
        } finally {
            lock.unlock();
        }
    }
```

**Justification**: 
A ReentrantLock provides the strict, simple mutual exclusion required to safely update primitive variables. Using a try-finally block is a critical design choice here; it guarantees that the lock is always released even if an unexpected error occurs during the update, preventing system-wide deadlocks.
---

### Critical Section #2: Execution Log

**What resource**: 
The executionLog list (ArrayList<String>).

**Why it needs protection**: 
The standard Java ArrayList is not a thread-safe data structure. If multiple processes attempt to append a log message at the exact same millisecond, it corrupts the internal array size and structure, causing the JVM to throw a ConcurrentModificationException and crash the program.

**Synchronization mechanism used**: 
Mutual exclusion using the existing ReentrantLock.

**Code snippet**:
```java
public static void logExecution(String message) {
        lock.lock();
        try {
            executionLog.add(message);
        } finally {
            lock.unlock();
        }
    }
```

**Justification**: 
By reusing the ReentrantLock created for the counters, I established a consistent locking strategy across all shared resources. The lock serializes access to the ArrayList, ensuring only one thread can modify its internal structure at any given time, thus making the logging operation entirely thread-safe.

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 

To strictly control and limit concurrent execution access to the simulated Central Processing Unit (CPU).

**Number of permits and why**: 
1 permit. This makes it a Binary Semaphore. It was configured with 1 permit to accurately simulate a single-core processor where only one process can be executing its time quantum at any single moment.

**Where implemented**: 
nside the Process class, specifically wrapping the core execution logic in both the run() and runToCompletion() methods.

**Code snippet**:
```java
@Override
    public void run() {
        try {
            SharedResources.cpuSemaphore.acquire();
        } catch (InterruptedException e) {
            System.out.println(Colors.RED + "  ✗ " + name + " was interrupted while waiting." + Colors.RESET);
            return;
        }
        
        try {
            // ... (Process execution logic: running quantum, printing progress) ...
        } finally {
            SharedResources.cpuSemaphore.release();
        }
    }
```

**Effect on program behavior**: 

The semaphore fundamentally changes the simulation from parallel execution to concurrent, scheduled execution. Instead of all threads printing their progress bars simultaneously in a chaotic jumble, the threads wait in line. The output now perfectly reflects a single-core CPU scheduler, displaying one process running at a time, yielding the CPU, and allowing the next process to execute.

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results as required by the assignment guidelines.

**Testing procedure**: 
```bash
# Commands used (run the program at least 5 times)
javac SchedulerSimulationSync.java
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync

```

**Results**: 
(The simulation completed successfully across all 5 runs. Because the Random object is seeded with my student ID, the generated processes and time quantum were identical every time. Most importantly, the final synchronization statistics (Total Context Switches, Completed Processes, and Total Waiting Time) were perfectly consistent across all 5 executions, proving the counters are secure)

**Why synchronization is necessary**: 
(Without synchronization, race conditions COULD occur because incrementing primitive variables (like contextSwitchCount++) is not atomic. Multiple threads might read the old value simultaneously, increment it, and overwrite each other, causing a "lost update." The shared counters and the execution log ArrayList desperately need protection to maintain data integrity when accessed concurrently by multiple thread objects.)

**Conclusion**: 
The implemented ReentrantLock completely eliminated the possibility of lost updates on the shared counters, ensuring 100% deterministic and consistent output across multiple test runs.
---

### Test 2: Exception Testing

**What I tested**: Checking for ConcurrentModificationException when modifying the shared execution log

**Testing procedure**: 
Executed the full multithreaded simulation where 10+ process threads continuously yield, finish, and log their execution status to the shared executionLog list simultaneously.

**Results**: 
The program ran to completion without crashing or throwing a single ConcurrentModificationException in the terminal.

**What this proves**: 

This practically proves that the mutual exclusion lock (ReentrantLock) placed inside the logExecution(String message) method effectively serializes access to the ArrayList. It successfully prevents the threads from corrupting the internal array structure of the list during concurrent append operations.

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total completed processes, context switches, etc.) against the initial parameters generated by my student ID seed.

**Expected values**: 

Total Completed Processes should be exactly equal to the initial number of generated processes: [14] processes.
The average waiting time should be a logically sound positive number.

**Actual values**: 
total Completed Processes: [14]

Total Context Switches: [30]

Total Waiting Time: [855017ms]

**Analysis**: 
The actual values match the expected logical outcomes. The fact that the "Completed Processes" count exactly matches the initial number of processes proves that no process threads were skipped, and the counter was safely incremented exactly once per finished thread without any lost updates.
---

### Test 4: Different Scenarios
**Scenario tested**: [Temporarily commenting out the Semaphore lock (acquire() and release()) in the Process class to simulate the exact same workload without CPU access control.]

**Purpose**: 
To visually confirm the necessity of the binary semaphore in regulating CPU time and preventing visual output corruption on the terminal.

**Results**: 
Without the semaphore, all threads immediately started printing their execution quantum bars at the exact same time. The terminal output became a scrambled, unreadable mess of overlapping text, and the logical concept of a single-core CPU was completely broken.

**What I learned**: 

I learned that while locks (ReentrantLock) are excellent for protecting data (variables and lists), a Semaphore is the superior tool for controlling access to a limited resource (like a CPU core). The semaphore acts as a traffic light, forcing threads to queue up and execute sequentially, which is crucial for building accurate system simulations.

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

[Through this assignment, I learned that while multithreading improves performance, it introduces severe risks like race conditions and deadlocks if left unmanaged. I discovered that simple operations like incrementing a counter (count++) are not atomic and require strict mutual exclusion to prevent "lost updates" when accessed by multiple threads. Implementing ReentrantLock taught me how to manually secure these critical sections, especially when working with non-thread-safe data structures like ArrayList. I also learned the distinct difference between locks and semaphores; a lock protects data by ensuring only one thread accesses it, whereas a semaphore acts as a signaling mechanism to control access to limited resources, like our simulated single-core CPU. The most crucial insight was the necessity of the try-finally design pattern, proving that resource release must be guaranteed regardless of runtime exceptions to prevent system-wide deadlocks.]

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 
High-Frequency Trading Platforms (e.g., TradeStation)
In financial markets, trading software must process live market data, update technical charts, and execute client orders concurrently. Synchronization is critical to prevent a race condition where an account balance is read and modified by two different threads simultaneously, which could result in double-spending or executing a trade with insufficient funds.

**Example 2**: 
Formula 1 Live Telemetry Systems
An F1 car transmits thousands of data points per second (tire temps, engine RPM, brake bias) to the pit wall. The data logging system uses multiple threads to receive this massive influx of data. Without synchronization, the thread responsible for displaying the real-time dashboard might read the data array exactly while another thread is halfway through updating it, resulting in a corrupted display or partial state that could ruin race strategy.

---

### How I would explain synchronization to others:

[Think of the CPU scheduler from Assignment 1 like an office trying to use a single, highly precise V60 coffee maker. In Assignment 1, we just let 10 people (the processes) try to pour their hot water and coffee grounds into the same filter at the exact same millisecond. It was a chaotic mess, and the coffee (our data) was ruined. Synchronization is just adding some simple office rules. A Semaphore is like a physical token; only the person holding the token is allowed to stand in front of the coffee maker. A Lock (ReentrantLock) is putting a padlock on the shared jar of coffee beans; you unlock it, take your exact amount of beans so no one bumps your hand or miscounts, and then you must lock it back up (the finally block) so the next person can safely use it.]

---

## Part 6: GitHub Repository Information

**Repository URL**: 
https://github.com/m7mdpsau/OS-Assignment3-MohammedAlsharif
**Number of commits**: 
4
**Commit messages**: 
1. Set my student ID: 444050659
2. Task 1: Add ReentrantLock to protect counter variables
3. Task 2: Add ReentrantLock to protect execution log ArrayList
4. Task 3: Add Semaphore to control concurrent CPU access
5. save DOC file

---

## Summary

**Total time spent on assignment**: 
Approximately 6-7 hours
**Key takeaways**: 
The functional difference between mutual exclusion (Locks) for data protection and signaling (Semaphores) for resource control.

The absolute necessity of using try-finally blocks to guarantee that locks and semaphores are released, thereby preventing deadlocks.

How silent data corruption (like missed increments on a shared counter) can occur without explicit synchronization, proving that thread-safe architecture is essential.

**Most challenging aspect**: 
Properly handling the InterruptedException when calling Semaphore.acquire(). It required restructuring the code to isolate the acquire() method in its own try-catch block before the main try-finally block, ensuring that a thread wouldn't accidentally try to release() a semaphore permit it never successfully acquired.

**What I'm most proud of**: 
Successfully blending both Locks and Semaphores to transform a chaotic, unreadable multithreaded output into a perfectly ordered and accurate simulation of a single-core CPU scheduler.
---

**End of Documentation**
