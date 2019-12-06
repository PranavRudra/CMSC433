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
