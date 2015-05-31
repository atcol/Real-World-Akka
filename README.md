# Real World Akka

This project aims to collect a set of recommendations and best practices for implementing actor systems using Akka.
Whilst most examples may be in Scala, the concepts, practices and recommendations should apply to Java too. 

# Designing Actor Systems

## The One Rule

Test first. It's easy. Your interface is `tell` and you give it messages. It's that simple. Implement your actors
through their tests, following the "Test. Code. Refactor." mantra. You will fail to reason correctly if you can't
document the actor's input/output through tests and the larger system gets the harder it will be to reason.

## Patterns

### Cameo 

### Anonymous Worker

### "If In Doubt, Push It Out"

# Testing Actor Systems

## The One Rule

Yes, this bit's repeated. It's that important: 

> Test first. It's easy. Your interface is `tell` and you give it messages. It's that simple. Implement your actors
> through their tests, following the "Test. Code. Refactor." mantra. You will fail to reason correctly if you can't
> document the actor's input/output through tests and the larger system gets the harder it will be to reason.
