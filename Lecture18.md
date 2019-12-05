# MPI

## Functions

- exchanges data between processes
- provides reliable communication
- works in distributed or shared memory systems

## Model

- communicator manages communications
- group contains communicating processes
- context provides communication channel

## Communications

- synchronous: acknowledgement received from user
- buffered: immediate
- standard: as soon as framework sends
- ready: immediate, but framework drops if receiver isn't ready to receive
