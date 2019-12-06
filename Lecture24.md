# Scheduling

## Process Execution

- cycle of CPU (burst) execution and I/O wait

## Criteria

- CPU utilization: keep the CPU as busy as possible
- throughput: number of processes that complete execution per unit time
- turnaround time: amount of time required to execute a particular process
- waiting time: amount of time a process has been waiting in the ready queue
- response time: amount of time it takes from when a request was submitted until first response produced

## Preemptive vs. Nonpreemptive

- preemptive: processes can be removed from current processor (improves response times)
- nonpreemptive: processes run until completion or until they yield control of a processor

## Decisions

- long term: should a new job be initiated or should it be held
- medium term: should a running program be temporarily swapped out
- short term: which thread should be given CPU next, which I/O operation should be sent to disk

## Dispatcher

- gives control of CPU to process selected by the short-term scheduler
  - switches context
  - switches to user mode
  - jumps to proper location in user program to start execution

## Types

- embedded: called as a function at the end of a kernel call, runs as a part of the calling process
- autonomous: separate process, run at every quantum (scheduler and other processors alternate)

## Algorithms

- first come first serve (FCFS): processes are executed as they come in, later processes are queued
- shortest job first (SJF): processes executed in order of shortest first (by CPU burst time)
  - convoy effect: short process behind long process reduces average waiting time
- shortest time remaining first (STRF): if process with shorter CPU burst time comes in, preempt current process
- round robin (RR): each process gets a quantum of CPU time to execute, processes that don't finish are preempted
  - quantum large -> becomes first-come first-serve
  - quantum small -> context switching is too intensive

## Priorities

- SJF may cause starvation if shorter processes keep coming in before a longer process
  - aging (increasing priority of process as time progresses) is one possible solution
