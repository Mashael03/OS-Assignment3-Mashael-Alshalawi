# Assignment 3 - Complete Documentation

**Student Name**: [Mashael]  
**Student ID**: [445052156]  
**Date Submitted**: [4 May]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [May 4, 2026, 10:00 AM]
**What I implemented**: 
I started by reviewing the starter code and the assignment requirements. I identified the shared resources in the program, including `contextSwitchCount`, `completedProcessCount`, `totalWaitingTime`, and `executionLog`.
**Challenges encountered**: 
The main challenge was understanding which variables were shared between threads and why they could cause race conditions.

**How I solved it**: 
I analyzed the `SharedResources` class and connected each shared variable to the synchronization concepts from Operating System Concepts, especially critical sections and mutual exclusion.

**Testing approach**: 
I ran the original program and checked the output values, including the number of processes, time quantum, context switches, completed processes, and execution log entries.

**Time spent**: 
40 minutes
---

### Entry 2 - [May 4, 2026, 11:30 AM]
**What I implemented**: 
I added the required synchronization imports: `Semaphore` and `ReentrantLock`. Then I created shared synchronization objects inside the `SharedResources` class.

**Challenges encountered**: 
I needed to decide whether to use one lock for all counters or separate locks.

**How I solved it**: 
I used one `ReentrantLock` called `counterLock` for the counter variables and another `ReentrantLock` called `logLock` for the execution log. This keeps the design simple while still protecting the critical sections.

**Testing approach**: 
I compiled the program after adding the imports and synchronization objects to make sure there were no syntax errors.

**Time spent**: 
35 minutes
---

### Entry 3 - [May 4, 2026, 1:00 PM]
**What I implemented**: 
I protected the shared counter methods: `incrementContextSwitch()`, `incrementCompletedProcess()`, and `addWaitingTime(long time)` using `counterLock`.

**Challenges encountered**: 
The main challenge was making sure the lock is always released even if an error occurs.

**How I solved it**: 
I used `try-finally` blocks. The lock is acquired before modifying the shared variable and released in the `finally` block.

**Testing approach**: 
I ran the program and verified that the final statistics still displayed correctly, including total context switches, completed processes, and total waiting time.

**Time spent**: 
50 minutes
---

### Entry 4 - [May 4, 2026, 3:00 PM]
**What I implemented**: 
I protected the `executionLog` ArrayList using `logLock`. This prevents unsafe concurrent modification of the shared list.

**Challenges encountered**: 
`ArrayList` is not thread-safe, so multiple threads adding messages at the same time could cause inconsistent behavior.

**How I solved it**: 
I wrapped the `executionLog.add(message)` statement inside a lock-protected critical section.

**Testing approach**: 
I verified that the program completed without `ConcurrentModificationException` and that the final execution log summary displayed the correct number of entries.

**Time spent**: 
35 minutes
---

### Entry 5 - [May 4, 2026, 5:00 PM]
**What I implemented**: 
I added a binary semaphore to control CPU access. I used `cpuSemaphore.acquire()` before process execution and `cpuSemaphore.release()` in a `finally` block.

**Challenges encountered**: 
The `acquire()` method can throw `InterruptedException`, so it had to be placed inside a try block.

**How I solved it**: 
I handled `InterruptedException` properly and used `finally` to guarantee that the semaphore is released after execution.

**Testing approach**: 
I ran the program multiple times. The output showed 16 completed processes, 35 context switches, and 70 execution log entries.

**Time spent**: 
55 minutes
---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**One race condition exists in the shared counter variables such as `contextSwitchCount`, `completedProcessCount`, and `totalWaitingTime`. These variables are updated by multiple process threads, and operations such as incrementing a counter are not atomic. For example, two threads could read the same value of `contextSwitchCount`, increment it separately, and write back the same result, causing a lost update.

A second race condition exists in the `executionLog` shared `ArrayList`. Since `ArrayList` is not thread-safe, multiple threads adding log messages at the same time may corrupt the internal structure of the list or cause inconsistent log entries. This could lead to missing log messages, incorrect log size, or possible `ConcurrentModificationException`.
**:



---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

[`ReentrantLock` is a mutual exclusion mechanism used to protect critical sections so that only one thread can execute a protected block of code at a time. I used `ReentrantLock` to protect the shared counters and the execution log because these resources must be updated safely and consistently.

A `Semaphore` controls access to a limited number of permits. In my code, I used a binary semaphore with one permit to represent CPU access. This means only one process thread can acquire the CPU semaphore and execute at a time. Therefore, `ReentrantLock` was used for protecting shared data, while `Semaphore` was used for controlling access to the simulated CPU resource.
]

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

[Deadlock occurs when two or more threads are blocked forever because each thread is waiting for a resource held by another thread. One prevention technique is to release locks and semaphores inside `finally` blocks, which ensures that resources are released even if an exception occurs. I applied this technique by unlocking `ReentrantLock` and releasing the CPU semaphore inside `finally`.

Another prevention technique is to avoid holding multiple locks unnecessarily or to use a consistent lock order when multiple locks are required. In my implementation, I used separate locks for different resources and kept each critical section short. This reduces the chance of circular waiting and helps prevent deadlock.
]

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

[For Task 1, I used one coarse-grained lock called `counterLock` to protect the three shared counter variables: `contextSwitchCount`, `completedProcessCount`, and `totalWaitingTime`. I chose this design because the counter operations are short and simple, so using one lock makes the implementation easier to understand and less error-prone.

The advantage of coarse-grained locking is simplicity and lower risk of programming mistakes. However, it may reduce concurrency because only one thread can update any counter at a time. A fine-grained approach would use separate locks for each counter, which could provide better concurrency because the counters are independent. Since these operations are very small, I decided that the simplicity of one lock was more appropriate for this assignment.
]

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: 
`contextSwitchCount`, `completedProcessCount`, and `totalWaitingTime`.
**Why they need protection**: 
They are shared variables updated by multiple threads. Without synchronization, updates may be lost or calculated incorrectly due to race conditions.

**Synchronization mechanism used**: 
`ReentrantLock` using `counterLock`.
**Code snippet**:
```java
// public static void incrementContextSwitch() {
    // Protect context switch counter
    counterLock.lock();
    try {
        contextSwitchCount++;
    } finally {
        counterLock.unlock();
    }


public static void incrementCompletedProcess() {
    // Protect completed process counter
    counterLock.lock();
    try {
        completedProcessCount++;
    } finally {
        counterLock.unlock();
    }
}

public static void addWaitingTime(long time) {
    // Protect total waiting time
    counterLock.lock();
    try {
        totalWaitingTime += time;
    } finally {
        counterLock.unlock();
    }
}
```

**Justification**: 

---

### Critical Section #2: Execution Log

**What resource**: 
executionLog, which is a shared ArrayList<String>.
**Why it needs protection**: 
ArrayList is not thread-safe. Multiple threads adding messages at the same time can cause inconsistent log entries or runtime errors.
**Synchronization mechanism used**: 
ReentrantLock using logLock.
**Code snippet**:
```java
// public static void logExecution(String message) {
    // Protect execution log
    logLock.lock();
    try {
        executionLog.add(message);
    } finally {
        logLock.unlock();
    }

```

**Justification**: 

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 
The semaphore controls access to the simulated CPU.
**Number of permits and why**: 
I used one permit because the simulation represents a single CPU resource. Only one process should execute on the CPU at a time.
**Where implemented**: 
The semaphore was implemented in the run() method and the runToCompletion() method.
**Code snippet**:
```java
// try {
    // Acquire CPU access
    SharedResources.cpuSemaphore.acquire();

    if (startTime == -1) {
        startTime = System.currentTimeMillis();
    }

    // Process execution code

 catch (InterruptedException e) {
    System.out.println(Colors.RED + "\n  ✗ " + name + " was interrupted." + Colors.RESET);
    Thread.currentThread().interrupt();
} finally {
    // Release CPU access
    SharedResources.cpuSemaphore.release();
}
```

**Effect on program behavior**: 

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results
I tested whether the program produces consistent and correct final statistics after adding synchronization.
**Testing procedure**: 
```bash
# Commands used (run the program at least 5 times)
```

**Results**: 
(Show that running multiple times produces consistent, correct results)
javac SchedulerSimulationSync.java
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync


The program completed successfully. Using student ID 445052156, the output generated 16 processes and a time quantum of 4000ms. The final statistics showed 35 total context switches, 16 completed processes, and 70 execution log entries.
**Why synchronization is necessary**: 
(Explain what race conditions COULD occur without synchronization, even if you didn't observe them. Explain which shared resources need protection and why.)

Synchronization is necessary because shared counters and shared lists can be accessed by multiple threads. Without locks, counter updates may be lost, and the ArrayList execution log may become inconsistent. Even if the issue does not appear in every run, the program still contains potential race conditions without synchronization.
**Conclusion**: 
The synchronized version protects all shared resources and produces correct final statistics.
---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException
I tested whether the program runs without ConcurrentModificationException or synchronization-related runtime errors.
**Testing procedure**: 
I ran the program multiple times after protecting the executionLog with logLock.
**Results**: 
The program completed successfully and printed the execution log summary. The output showed Total log entries: 70.
**What this proves**: 

This proves that the shared execution log was updated safely and that the synchronized list access did not cause runtime errors.
---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)
I verified that the final values match the expected behavior of the scheduler.
**Expected values**: 
All processes should finish execution. Therefore, the completed process count should equal the number of generated processes. The execution log should contain entries for process execution, yielding, and completion.
**Actual values**: 
The output showed:

Number of processes: 16
Time quantum: 4000ms
Total context switches: 35
Total completed processes: 16
Total execution log entries: 70
**Analysis**: 

The completed process count matched the number of created processes, which confirms that all processes finished correctly. The context switch count increased each time a process executed a CPU quantum. The execution log also recorded process activity, which confirms that logging worked correctly.
---

### Test 4: Different Scenarios
**Scenario tested**: [e.g., different time quantum, more processes, etc.]

I tested the program by running it multiple times with the same student ID and checking that the generated values remained consistent.
**Purpose**: 
The purpose was to confirm that the synchronization changes did not break the scheduler logic.
**Results**: 

The program consistently generated the same process count and time quantum because the random generator uses the student ID as the seed.
**What I learned**: 

I learned that deterministic testing is useful because using the student ID as a seed allows the same scheduling scenario to be repeated and verified.
---

## Part 5: Reflection and Learning

### What I learned about synchronization:

[I learned that synchronization is necessary when multiple threads access shared data. A race condition can occur even when the program appears to work correctly in some runs. I also learned that simple operations such as incrementing a counter are not always atomic. Using ReentrantLock helps protect critical sections and prevents lost updates. I learned that Semaphore is useful when the program needs to control access to a limited resource, such as the CPU in this simulation. I also learned that try-finally is important because it guarantees that locks and semaphores are released. Overall, this assignment helped me understand how operating systems coordinate processes and protect shared resources.]

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 
Banking systems require synchronization when multiple transactions update the same account balance. Without synchronization, two withdrawals or deposits could produce an incorrect final balance.


**Example 2**: 
Online booking systems require synchronization when many users try to reserve the same seat or room. Without synchronization, the system may allow double booking.

---

### How I would explain synchronization to others:

[Synchronization is like allowing only one person to use a shared notebook at a time. If many people write in the notebook at the same time, the writing may overlap or become incorrect. In programming, shared variables are like that notebook. Locks are used to make sure only one thread writes to shared data at a time. Semaphores are like a limited number of keys for a shared resource. If there is only one key, only one thread can use the resource until the key is returned.]

---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/Mashael03/OS-Assignment3-Mashael-Alshalawi.git

**Number of commits**: 7

**Commit messages**: 
1. Changed student ID
2. Added imports, locks and semaphore
3. Protected completed process counter with ReentrantLock
4. Protected total waiting time with ReentrantLock
5. Protected execution log with ReentrantLock
6. Add CPU semphore control to run method
7. Add CPU semaphore control to runToCompletion method

---

## Summary

**Approximately 2 Days**: 

**Key takeaways**: 
Shared resources must be protected to avoid race conditions.
ReentrantLock is useful for protecting critical sections.
Semaphore is useful for controlling access to limited resources such as CPU access.
**Most challenging aspect**: 

The most challenging aspect was handling InterruptedException correctly when using Semaphore.acquire().
**What I'm most proud of**: 

I am most proud of completing the synchronization implementation while keeping the original scheduler structure unchanged.
---

**End of Documentation**
