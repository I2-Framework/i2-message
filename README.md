# I2 Framework Messaging and EventBus Module
Welcome to **i2-message**, a Java messaging and event bus module. Send POJO (Plain Old Java Object) messages and events with ease. This module is completely standalone but is part of the greater *I2 Framework*.

## Status
* Local domain and bus dispatch complete
* Currently, working on remote multi-domain messaging
**=> Check out the** [Full Status Blog](https://github.com/I2-Framework/i2-message/wiki/Status-Blog)

## Module Dependencies
* None

## Getting started
Messages are POJOs (Plain Old Java Objects) which you send using a `MessageBus` to subscribers  who receive these messages. There is very little to no setup or tear down.

First we setup a subscriber which is any method in our example object that is annotated with `@Subscribe` annotation. This method will be called and our POJO message object will be *injected*. By convention, the first not-annotated method parameter is expected to be the message parameter. The class type of the *message* parater determines the types messages to be received.

In our example we have a simple POJO class called `Message`:
```java
public class Message {
	public final String message;
	public Message(String message) { this.message = message; }
}
```
We are going to send a message using this POJO class to a message subscriber in our `Example1` class:
```java
@Subscribe
public void myMessageHandler(Message message) {
	System.out.println(message.message);
}
```
The key points are the following. First, the method is annotated with `@Subscribe` annotation which marks this method as a message receiver. Secondly, the first parameter `Message message` determines the type of messages this subscriber is will receive. The parameter type `Message` or any of its superclasses will be used to determine which messages are delivered to this subscriber method.

> **Note:** you also can register explicit *subscribers* implementating a handler interface `bus.subscribe(Message.class, new Subscriber() {...})` or more compactly using *lambda* expression or *method reference* `bus.subscribe(Message.class, System.out::println)`

The subscription itself occures at some control point which in our example is the `Example1.main()` method. 
```java
final static MessageBus messageBus = MessageBus.getMessageBus("org.i2.messagebus.tutorials.tutorial1");

public static void main(String[] args) {

	Example1 example1 = new Example1();

	messageBus.subscribe(example1);

	messageBus.post(new Message("Hello World"));

	messageBus.unsubscribe(example1);

}
```
First create a message bus:
```java
final static MessageBus messageBus = MessageBus.getMessageBus("org.i2.messagebus.tutorials.tutorial1");
```
in a very similar way that java `java.util.Logger` is setup with a name that is used to identify a particular instance of an `MessageBus`. If `MessageBus` does not exist, it will be created and cached and returned next time.

Next we create our `Example1` object, which also contains the `Example1.myMessageHandler(Message message)` subscriber. 
```java
Example1 example1 = new Example1();
```
Then we subscribe our method with `MessageBus.subscribe(Object subsriberContainer)`, which will reflectively look for methods annotated with `@Subscribe` annotation. We can define as many such subscribers as needed in a single object. Each subscriber will be registered with the `MessageBus` as a receiver of messages of certain types. 

```java
messageBus.subscribe(example1);
```
Our `Example1.myMessageHandler(Message message)` method is now subscribed and will receive all messages of type `Message`.

Now we are ready to send an actual message:
```java
messageBus.post(new Message("Hello World"));
```
We `post` our message on the bus and our suscriber will receive it. 

Please note, that `MessageBus.subscribe()` method returns a `Subscription` result which we can use to manager our subscription but we elect to use an even simpler approach was to unsubscribe, since we don't want to keep any state.
```java
messageBus.unsubscribe(example1);
```
which will unsubscribe all previous subscribers of `example` object. Technically we don't have to unsubscribe anything since cleanup happens implicitely when the instance of the object subscribers are part of is garbage collected. However it is cleaner to excplicitely unsubscribe and release any held resources. Internally the module uses java `Reference` and releases sources and subscribers when they are no longer in use. 

### More on subscribers
The `MessageBus` module relies on message and parameter injection when invoking subscribers of a particular type or types of messages. The `MessageBus` can inject the following builtin types when posting a message to a subscriber.

* `@Source T source` when messages are posted from a particular source, the source object of a particular type will also be injected. Only messages posted from a source that matches the parameter type will be received. 
* `@Source Optional<T> source` an optional source in either case where a message is posted form a particular source or general broadcast
* `MessageInfo` interface which contains more information about the message just recevied
* `MessageBus` injects the message bus which posted the message
* `MessageDomain` injects the current message domain, `LocalMessageDomain` by default

Here is a short example of both:
```java
@Subscribe
public void handler2(Message message, @Source Example1 source) {}
```
where the source of the message must be specified and it must come from `Example1` type and
```java
@Subscribe
public void handler3(Message message, MessageInfo info, @Source Optional<Example1> source) {
```
Where the source is optional, meaning it did not have to be specified when posting a message, but if it was, it will be supplied. Also a `MessageInfo` object was injected which provides more information about the message and even allows the subscriber to *veto* further propagation of the message. 

### General reference injection
When combined with `org.i2.inject` module, any bound reference object can be injected into the subscriber method. However,  without the inclusion of the inject module, just the builtin injections will occur for the described builtin types such as the message, `MessageInfo` data and any message *source* object if present.

# Domains and name space

The `MessageBus` instances are allocated in a hierarchal domain space using static call `MessageBus.getMessageBus(String name)`. The default domain is called the `LocalMessageDomain` where all of the communication happens locally. Following the `java.util.logging` name space pattern, the `MessageBus` module allocates buses out of the current local doamin. The convention is to use dot-separated hierarchal names or java package names, but just like with logging framework, any name can be used.

Further more, multiple local domains can be created for segragation of name space and provide scope access to message buses. This allows applications to secure their messages so outside actors do not intercept private messages. Access Control can be used to secure domains.

Other domain types also are defined (in other module extensions) for remote message sending and asynchronous communication between multiple domains, etc.

