# Real World Akka

This project aims to collect a set of recommendations and best practices for implementing actor systems using Akka.
Whilst most examples may be in Scala, the concepts, practices and recommendations should apply to Java too. 

# Designing Actor Systems

Background reading and videos:

 * [Designing Actor Based Applications](http://www.nljug.org/jfall/session/akka-in-practice-designing-actor-based-application/34/)
 * [A Crash Course in Failure](http://web.archive.org/web/20090430014122/http://nplus1.org/articles/a-crash-course-in-failure/)


## The One Rule

Test first. It's easy. Your interface is `tell` and you give it messages. It's that simple. Implement your actors
through their tests, following the "Test. Code. Refactor." mantra. You will strugle to reason about your actors if you can't
document the actor's input/output through tests and the larger system gets the harder it will be to reason.

## The Error Kernel

> Push higher-risk work to the leaf nodes, leaving state and longer living actors as parents

The Error Kernel pattern is taken from Erlang's 


## Patterns

### Behavioural

#### Cameo 

#### Anonymous Worker

#### "If In Doubt, Push It Out"

> Instead of letting actors grow, let common behaviour or longer-running tasks be delegated to workers

Actors can quickly grow in behaviour and responsibilities. This in turn can increase the risk of failure which can, and will, impact all children and clients. 
If you find yourself squinting whilst reasoning about the actor, consider pushing out common behaviour to an shortly-lived "worker". This worker is given `sender()` 
and all the other parameters it needs at creation time, and is then left to complete/wait/do whatever to achieve the task, stopping itself once complete. The added benefit
is that you keep your actors small and encourage `K.I.S.S` through reduced responsibilities and behaviour that's testable and reusable.

### Structural / Creational

#### Companion Object

In Scala, one can define a companion object that can be used as a factory for creation of instances of the class in
question. In Akka this can be used for two purposes:

	1. Simplifying the creation of `Props` for the actor
	2. Grouping the messages (case classes) used for communicating with the actor: both input and output

Consider this actor:

	class CounterActor(start: Int, initialSubscribers: List[ActorRef]) extends Actor {

		var current: Int = start

		def receive: Receive = normal(initialSubscribers)

		def normal(subs: List[ActorRef]): Receive = {
			case i: Int 					=> from = from + i 
			case Freeze 					=> context.become(frozen)
			case s@Subscribe(a)		=> context.become(normal(subs + a))
			case DistributeValue 	=> subscribers.foreach(who => who ! Value(from))
		}

		def frozen: Receive = {
			case Unfreeze => context.become(normal)
			// Ignore everything else
		}
	}

`CounterActor` accepts 5 different message types: `Int`, `Freeze`, `DistributeValue`, `Subscribe` and `Unfreeze`, none of which are
externally documented. Furthermore the creation of the Actor requires two inputs, which in most cases will be default
starting values -- zero and empty respectively -- thus burdening the client/user. Instead, we could use a
companion object to hide this creation detail and document the different message types:

	object CounterActor {
		case object Freeze
		case object DistributeValue
		case object Unfreeze
		case class Subscribe(actor: ActorRef)

		def props(): Props = {
			Props(new CounterActor(0, List.empty[ActorRef]))
		}
	}

So instead of:

	context.actorOf(Props(new CounterActor(0, List.empty[ActorRef])))

you just write:

	context.actorOf(CounterActor.props())
	
and not only save on the typing, but avoid having to scour receive blocks to determine the message types as they're
clearly documented in the companion object. Of course this is even easier considering you've tested first...

# Testing Actor Systems

Background reading and videos:

 * [Akka Documentation on Testing](http://doc.akka.io/docs/akka/snapshot/scala/testing.html)
 * [Testing Actors and FSMs by Roland Kuhn](https://skillsmatter.com/skillscasts/3235-actors-fsm-akka)

## The One Rule

Yes, this bit's repeated. It's that important: 

> Test first. It's easy. Your interface is `tell` and you give it messages. It's that simple. Implement your actors
> through their tests, following the "Test. Code. Refactor." mantra. You will strugle to reason about your actors if you can't
> document the actor's input/output through tests and the larger system gets the harder it will be to reason.

## Failures

[![Creative Commons License](https://i.creativecommons.org/l/by-nc-sa/4.0/80x15.png)](http://creativecommons.org/licenses/by-nc-sa/4.0/)  <span xmlns:dct="http://purl.org/dc/terms/" href="http://purl.org/dc/dcmitype/Text" property="dct:title" rel="dct:type">Real World Akka</span> by [Alex Collins](http://github.com/atc-/RealWorldAkka) is licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-nc-sa/4.0/). Based on a work at [http://github.com/atc-/RealWorldAkka](http://github.com/atc-/RealWorldAkka).
