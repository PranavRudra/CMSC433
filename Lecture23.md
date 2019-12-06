# Remote Method Invocation (RMI)

## Core

- enables you to make a normal function call, but it have it be executed on another machine
- arguments/return values will be marshalled/unmarshalled for transport across network
  - marshalling: converting object into sequence of bytes (serializing)
  - unmarshalling: converting sequence of bytes into object (deserializing)

## Registries

- object registry: name server that relates remotely accessible objects with (unique) names
  - each server (JVM) has an object registry on the same host computer
- client that wishes to access a remote object can do so by giving the object name to a registry
- server that wishes to make an object available for RMI must register it with its object registry

## Stubs

- proxy that the client uses to initiate remote method invocations
  - when client queries object registry for an object, stub is returned
  - stub matches the same interface as the actual remote object
  - stub handles marshalling, unmarshalling...
