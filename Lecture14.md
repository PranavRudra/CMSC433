# Actors

## Distributed Systems

- host: computer in a distributed environment
- port: communication channel used by hosts to exchange messages
- network: a system consisting of hosts and the equipment used to connect them

## Actor Model

- multiprocess programming paradigm
- no shared memory
- no assumptions about distributed/non-distributed
- all communication occurs through interprocess communication (IPC)

## Actors

- actor is an independent sequential (i.e. single-threaded) computation
- each actor has a mailbox that it extracts messages from to process
- actors communicate by sending each other messages

## Message Passing

- asynchronous: senders don't know when messages are received
- at most once delivery: every message sent is eventually received at most once (no duplication)
- locally first-in-first-out: messages sent by one actor to another are received in the order sent (excepting lost messages)

## Akka

- `ActorSystem` provides actor execution (e.g. threads) and message-passing infrastructure

```java
ActorSystem actorSystem = ActorSystem.create("Message_Printer");
Props props = Props.create(MessagePrinterActor.class);
ActorRef messagePrinterActor = actorSystem.actorOf(props, "Message_Printer_Actor");

public class MessagePrinterActor extends AbstractActor {
    @Override
    public Receive createReceive() {
        // specifies how to deal with received messages
        return receiveBuilder()
            .match(Double.class, n -> { sender().tell(d.isNaN() ? 0 : d, self()); })
            .match(Integer.class, i -> { sender().tell(i * 10, self()); });
            .build();
            // tell() is fire and forget, non-blocking
    }
}

public class MessageAcknowledgerActorextends AbstractActor{
    ...
    public void onReceive(Object msg) throws Exception {
        if (msg instanceof String) {
            ActorRef sender = getSender();
            String payload = (String) msg;
            System.out.printf("Message is:  %s%n", payload);
            sender.tell(payload + " message received", sender);
        }
    }
}

// kills all actors
actorSystem.terminate();
```

```java
public class PongActor extends AbstractLoggingActor {
    @Override
    public Receive createReceive() {
        return receiveBuilder()
            .match(String.class, this::onString)
            .build();
    }
    
    public void onString(String msg) throws Exception {
        if (msg instanceof String) {
            String payload = (String) msg;
            if (payload.equals("stop")) { 
                // game over
                System.out.println(getSelf().path().name() +  ": OK");
            } else if (payload.equals("start")) {
                // getSelf.path().name() returns runtime name of actor
                System.out.println(getSelf().path().name() +  ": Let's do it.");
                getSender().tell("go", getSelf());
            }else { 
                // next stroke
                System.out.println("Pong");
                getSender().tell("go", getSelf());
            }
        }
    }
}

public class PingActor extends AbstractActor {

    private int numHitsLeft;
    private ActorRef partner;
    
    public PingActor(int numHits) {
        this.numHitsLeft = numHits;
    }
    
    @Override public Receive createReceive() {
        return receiveBuilder()
            .match(ActorRef.class, this::onActorRef)
            .match(String.class, this::onString)
            .build();
    }
}

public class PingPong{

    public static void main(String[] args) {
        ActorSystem actorSystem = ActorSystem.create("Ping_Pong");
        // 5 is the argument to the PingActor constructor
        Props pingProps = Props.create(PingActor.class, 5);
        Props pongProps = Props.create(PongActor.class);
        ActorRef pingNode = actorSystem.actorOf(pingProps, "Ping_Node");
        ActorRefpongNode = actorSystem.actorOf(pongProps, "Pong_Node");
        pingNode.tell(pongNode, null);
        actorSystem.terminate();
    }
}
```

## Messages

- must be immutable
- must be serializable

## Supervision

- akka actors have a parent and can have children if they create them using `ActorContext`
- guardians are `/` (root guardian), `/system` (system guardian), `/user` (guardian actor)
  - `/system` and `/user` are children of root
  - actors created w/ `ActorContext` are children of `/user`
 
```java
getContext().parent();      // gives the parent of the calling actor
getSelf().path().name();    // gives the name of the calling actor
```

### Failure

- when actors fail, a system message is sent to their parent in a special queue used only for parent-child communication
- options
  - resume: `createReceive()` is re-invoked (message being processed is lost but not queued messages)
  - stop: kills the actor after the finishing processing of the message it's currently working on
    - kill is propagated to all of the actor's descendants as well
  - restart the failed actor
  - escalate: throw the same exception as child and hand off responsibility to parent

### Strategies

- `SupervisorStrategy`: determines how failures of children will be handled (get by executing `.supervisorStrategy()`) 
  - `AllForOneStrategy`: if one child fails, apply the strategy to all children (subclass of `SupervisorStrategy`)
  - `OneForOneStrategy`: apply supervision strategy only to the failed child (subclass of `SupervisorStrategy`)

## Memory Model

- if one actor sends a message to another, pending writes are guaranteed to be visible after the message is received
- pending writes after an actor reads a message are visible when the actor reads the next message
- only comes into play when actors are sharing memory (unusual since they're often distributed)

