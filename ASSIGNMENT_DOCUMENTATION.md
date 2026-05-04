# Assignment 3 - Complete Documentation

**Student Name**: [Your Full Name]  
**Student ID**: [Your ID]  
**Date Submitted**: [Submission Date]

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

### Entry 1 - [April 27,4:00 PM]
**What I implemented**: Initial fork of the repository and set my Student ID in the code.

**Challenges encountered**:Understanding the existing codebase from Assignment 1. 

**How I solved it**: Reviewed the TODO comments and the README.md file carefully.

**Testing approach**: Compiled the code to ensure it runs without syntax errors.

**Time spent**: 45minutes.

---

### Entry 2 - [April 28, 5:00 PM]
**What I implemented**: Task 1: Integrated a ReentrantLock to protect the global counters (contextSwitchCount, completedProcessCount, totalWaitTime).

**Challenges encountered**: Ensuring the lock was released even if an exception occurred.

**How I solved it**: Wrapped the increment logic in a try-finally block, calling unlock() in the finally section.

**Testing approach**:Ran the code to see if counters were still incrementing correctly. 

**Time spent**: 2.5 hours.

---

### Entry 3 - [April 29, 8:00 AM]
**What I implemented**: Task 2 & 3: Added a lock for the executionLog ArrayList and a Semaphore for CPU access.

**Challenges encountered**: Identifying the exact "Critical Section" where the CPU is used.

**How I solved it**: Placed the semaphore.acquire() before the process "burst" execution and release() immediately after.

**Testing approach**: Checked for ConcurrentModificationException during high-concurrency runs.

**Time spent**: 3 hours.

---

### Entry 4 - [May 1, 6:00 AM]
**What I implemented**: Task 4: Documentation and final testing.

**Challenges encountered**: Verifying that the semaphore actually limits access to one process at a time.

**How I solved it**: Added print statements to track which thread held the permit.

**Testing approach**: Ran the simulation 10 times to check for consistency.

**Time spent**: 1.5 hour.

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:
1-Global Counters: The contextSwitchCount++ is not atomic. When multiple threads execute read-modify-write simultaneously, updates are lost (e.g., two threads read 5, both increment to 6 and save, losing one count).
2-Execution Log: The ArrayList.add() method is not thread-safe. Concurrent access can cause a ConcurrentModificationException or result in null entries in the list because the internal array resize logic is interrupted.

[Your answer here - 4-6 sentences with code examples]

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:
A ReentrantLock is a mutual exclusion (mutex) mechanism where only the thread that locked it can unlock it. I used it for the counters and the log because they require exclusive ownership. A Semaphore maintains a set of permits. I used a Binary Semaphore (1 permit) for the CPU access. While similar to a lock, a semaphore is more suitable here as a signaling mechanism to control access to a limited hardware resource (the simulated CPU).

[Your answer here - explain your implementation choices]

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:
1-Prevention Technique 1 (Lock Ordering): Always acquiring locks in the same order.
2-Prevention Technique 2 (Timed/Non-blocking attempts): Using tryLock().
In my code, I prevented deadlocks by using try-finally blocks to ensure every lock is released regardless of errors and by keeping critical sections small and non-nested.

[Your answer here - reference try-finally blocks, lock ordering, etc.]

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:
I chose fine-grained locking (separate locks) for the counters and the log, but a single lock for the three related counters. Specifically, because contextSwitchCount and totalWaitingTime are often updated together, one lock for counters is efficient (Coarse-grained for counters). However, having a separate lock for the executionLog allows one thread to update counters while another thread is still writing to the log, increasing concurrency. Fine-grained locking is better for independent resources as it reduces "lock contention," allowing more threads to work simultaneously.

[Your answer here - explain coarse-grained vs fine-grained locking, independence of counters, concurrency implications. Show understanding of when to use each approach. 5-8 sentences expected.]

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: 
contextSwitchCount, completedProcessCount, totalWaitingTime

**Why they need protection**: 
To prevent lost updates during concurrent increments.

**Synchronization mechanism used**: 
ReentrantLock

**Code snippet**:
```java
counterLock.lock();
try {
    contextSwitchCount++;
} finally {
    counterLock.unlock();
}
```

**Justification**: 
Ensures atomicity of the increment operation.
---

### Critical Section #2: Execution Log

**What resource**: 
ArrayList<String> executionLog

**Why it needs protection**: 
ArrayList is not thread-safe; concurrent adds cause data corruption.

**Synchronization mechanism used**: 
ReentrantLock
**Code snippet**:
```java
logLock.lock();
try {
    executionLog.add(logEntry);
} finally {
    logLock.unlock();
}
```

**Justification**: 
Prevents ConcurrentModificationException
---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 
To simulate a single-core CPU that only processes one task at a time.

**Number of permits and why**: 
1 permit (Binary Semaphore), because there is only one CPU.
**Where implemented**: 
Surrounding the process "burst" simulation.
**Code snippet**:
```java
cpuSemaphore.acquire();
try {
    // Simulate burst execution
} finally {
    cpuSemaphore.release();
}
```

**Effect on program behavior**: 
Ensures that even with multiple threads, the CPU usage remains sequential.
---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: Ran the program 5 times using java SchedulerSimulationSync
```bash
# Commands used (run the program at least 5 times)
```

**Results**: 
Total Waiting Time and Completion counts were identical in all 5 runs.
**Why synchronization is necessary**: 
Without it, the "Total Waiting Time" would vary because threads would overwrite each other's updates.

**Conclusion**: 
The locks effectively stabilized the output.
---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: 
Presence of ConcurrentModificationException
**Results**: 
0 exceptions thrown over 10 trials.
**What this proves**: 
The logLock successfully protected the ArrayList.
---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: 
Total processes completed = 10 (based on input).
**Actual values**: 
10 
**Analysis**: 
The synchronization did not interfere with the logic, only the safety.
---

### Test 4: Different Scenarios
**Scenario tested**: [e.g., different time quantum, more processes, etc.]

**Purpose**: 

**Results**: 

**What I learned**: 

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

I learned that multithreading is powerful but dangerous without control. The most important concept was the Critical Section—identifying exactly where shared data is modified. I also realized that try-finally is not optional; it is a safety requirement to prevent a program from hanging forever (Deadlock). Seeing a ConcurrentModificationException disappear after adding a lock was a "lightbulb moment" for me regarding thread safety.

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 
Banking Systems: To prevent two people from withdrawing money from the same account at the exact same millisecond and resulting in a negative balance.

**Example 2**: 
E-commerce Inventory: To ensure that if only one item is left in stock, only one customer can successfully "checkout" even if ten people click at once.
---

### How I would explain synchronization to others:

Imagine a single-person bathroom (the resource) in a busy cafe. Even if ten people (threads) want to use it, the lock on the door (Mutex) ensures only one person is inside. Everyone else waits in line. If there was no lock, multiple people would try to walk in at once, causing a "race condition" and total chaos!

---

## Part 6: GitHub Repository Information

**Repository URL**: 

**Number of commits**: 

**Commit messages**: 
1. 
2. 
3. 
4. 

---

## Summary

**Total time spent on assignment**: 
8 hours
**Key takeaways**: 
1. Thread safety
2. Mutex vs Semaphore
3. Atomic operations

**Most challenging aspect**: 
Debugging why the semaphore was causing a delay (it was working correctly, just limiting the speed!).
**What I'm most proud of**: 
Successfully implementing the try-finally pattern to ensure a robust, crash-proof simulation.
---

**End of Documentation**
