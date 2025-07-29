# Homework (Simulation)

This program, `process-run.py`, allows you to see how process states change as programs run and either use the CPU (e.g., perform an add instruction) or do I/O (e.g., send a request to a disk and wait for it to complete). See the README for details.

## Questions

### 1. Run process-run.py with the following flags: `-l 5:100,5:100`

What should the CPU utilization be (e.g., the percent of time the CPU is in use?) Why do you know this? Use the `-c` and `-p` flags to see if you were right.

```
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1          
  2        RUN:cpu         READY             1          
  3        RUN:cpu         READY             1          
  4        RUN:cpu         READY             1          
  5        RUN:cpu         READY             1          
  6           DONE       RUN:cpu             1          
  7           DONE       RUN:cpu             1          
  8           DONE       RUN:cpu             1          
  9           DONE       RUN:cpu             1          
 10           DONE       RUN:cpu             1          

Stats: Total Time 10
Stats: CPU Busy 10 (100.00%)
Stats: IO Busy  0 (0.00%)
```

**Answer:** The CPU utilization is 100% since it is busy for all 10 units and there is not a point where a process waits for I/O.

---

### 2. Now run with these flags: `./process-run.py -l 4:100,1:0`

```
Time        PID: 0        PID: 1           CPU           IOs
  1        RUN:cpu         READY             1          
  2        RUN:cpu         READY             1          
  3        RUN:cpu         READY             1          
  4        RUN:cpu         READY             1          
  5           DONE        RUN:io             1          
  6           DONE       BLOCKED                           1
  7           DONE       BLOCKED                           1
  8           DONE       BLOCKED                           1
  9           DONE       BLOCKED                           1
 10           DONE       BLOCKED                           1
 11*          DONE   RUN:io_done             1          

Stats: Total Time 11
Stats: CPU Busy 6 (54.55%)
Stats: IO Busy  5 (45.45%)
```

**Answer:** Process 0 runs its 4 CPU instructions and then process 1 starts its I/O which takes 1 second for the initialization. The I/O process takes by default 5 units and 1 unit to complete. In total 11 units.

---

### 3. Switch the order of the processes: `-l 1:0,4:100`

What happens now? Does switching the order matter? Why? (As always, use `-c` and `-p` to see if you were right)

```
Time        PID: 0        PID: 1           CPU           IOs
  1         RUN:io         READY             1          
  2        BLOCKED       RUN:cpu             1             1
  3        BLOCKED       RUN:cpu             1             1
  4        BLOCKED       RUN:cpu             1             1
  5        BLOCKED       RUN:cpu             1             1
  6        BLOCKED          DONE                           1
  7*   RUN:io_done          DONE             1          

Stats: Total Time 7
Stats: CPU Busy 6 (85.71%)
Stats: IO Busy  5 (71.43%)
```

**Answer:** Here process 0 starts its I/O and immediately blocks which takes 1 unit. The scheduler switches to process 1 which runs 4 units while I/O of process is happening in the background. The overlap reduced the total time: 1 + 5 + 1 = 7.

---

### 4. Exploring the `-S` flag with `SWITCH_ON_END`

We'll now explore some of the other flags. One important flag is `-S`, which determines how the system reacts when a process issues an I/O. With the flag set to `SWITCH_ON_END`, the system will NOT switch to another process while one is doing I/O, instead waiting until the process is completely finished. What happens when you run the following two processes (`-l 1:0,4:100 -c -S SWITCH_ON_END`), one doing I/O and the other doing CPU work?

**Answer:** The process will not be switched even when it is blocked for I/O, the result will be the same as that of question 2.

---

### 5. Using `SWITCH_ON_IO` behavior

Now, run the same processes, but with the switching behavior set to switch to another process whenever one is WAITING for I/O (`-l 1:0,4:100 -c -S SWITCH_ON_IO`). What happens now? Use `-c` and `-p` to confirm that you are right.

**Answer:** Opposite to the previous question, the result should be identical to that of question 3.

---

### 6. I/O completion behavior with `IO_RUN_LATER`

One other important behavior is what to do when an I/O completes. With `-I IO_RUN_LATER`, when an I/O completes, the process that issued it is not necessarily run right away; rather, whatever was running at the time keeps running. What happens when you run this combination of processes? (`./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -c -p -I IO_RUN_LATER`) Are system resources being effectively utilized?

**Answer:** This effectively starves the I/O bound processes.

---

### 7. I/O completion behavior with `IO_RUN_IMMEDIATE`

Now run the same processes, but with `-I IO_RUN_IMMEDIATE` set, which immediately runs the process that issued the I/O. How does this behavior differ? Why might running a process that just completed an I/O again be a good idea?

**Answer:** I/O-bound processes follow I/O → CPU → I/O. By running it right after its first I/O it can quickly perform its short CPU task and move on to the next I/O.

---

### 8. Random process generation

Now run with some randomly generated processes using flags `-s 1 -l 3:50,3:50` or `-s 2 -l 3:50,3:50` or `-s 3 -l 3:50,3:50`. See if you can predict how the trace will turn out. What happens when you use the flag `-I IO_RUN_IMMEDIATE` versus that flag `-I IO_RUN_LATER`? What happens when you use the flag `-S SWITCH_ON_IO` versus `-S SWITCH_ON_END`?

## Flag Behavior Analysis

### `-I IO_RUN_IMMEDIATE` vs. `-I IO_RUN_LATER`

- **`IO_RUN_IMMEDIATE`**: Prioritizes system **responsiveness** 🏃. A process that finishes an I/O gets to run right away, even if it interrupts another job. This is best for interactive applications.

- **`IO_RUN_LATER`**: Prioritizes **CPU throughput**. The currently running process finishes its work first. This is less responsive and can make I/O-bound processes wait.

### `-S SWITCH_ON_IO` vs. `-S SWITCH_ON_END`

This flag determines when the system switches between processes.

- **`SWITCH_ON_IO`**: Enables efficient **multiprogramming** ⚙️. It switches to another job when the current one blocks for I/O. This allows the CPU and I/O devices to work in parallel, maximizing resource use.

- **`SWITCH_ON_END`**: Forces job **serialization** ⏸️. It waits for each process to finish completely before starting the next. This is very inefficient if any I/O is involved, as it leads to significant idle CPU time.