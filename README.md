# i2-message-bus
Java POJO messaging and event bus module

## Getting started
Messages are POJOs (Plain Old Java Objects) which you send using a `MessageBus` to subscribers  who receive these messages. There is virtually no setup or tear down.

First we setup a subscriber which is any method in our example object that is annotated with `@Subscribe` annotation. This method will be called and our POJO message object will be *injected*. By convention, the first not-annotated method parameter is expected to be the message parameter. The class type of the first parater determines which messages are to be received.

In our example we have a simple POJO class called `Message`:
```java
public class Message {
	public final String message;
	public Message(String message) { this.message = message; }
}
```
We are going to send a message using this POJO class to a message subscriber in our `Main.class` example1 class:
```java
@Subscribe
public void myMessageHandler(Message message) {
	System.out.println(message.message);
}
```
The key points are the following. First, the method is annotated with `@Subscriber` annotation which marks this method as a message receiver. Second, the first parameter `Message message` determines the type of messages this method is will receive. The parameter type `Message.class` or any of its superclasses will be used to determine which messages are delivered to this subscriber method.

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

Next we create our `Example1` object, which also contains the `Example1.myMessageHandler(Message message)` subscriber method. 
```java
Example1 example1 = new Example1();
```
Then we subscribe our method with `MessageBus.subscribe(Object subsriberContainer)` method, which will use reflection to look for methods which are annotated with `@Subscribe` annotation. We can define as many such methods as we want in a single object and each one will be registered with the `MessageBus` as a receiver of messages of certain types. 
```java
messageBus.subscribe(example1);
```
Our `Example1.myMessageHandler(Message message)` method is not registered and will receive all messages of type `Message`.

Now we are ready to send a message:
```java
messageBus.post(new Message("Hello World"));
```
We `MessageBus.post(Object pojoMessage)` message on the bus and all subscribers will receive it. 

The `MessageBus.subscribe()` method actually returns a `Subscription` result which we can use to unsubscribe our subscriber using `Subscription.removeSubscription()` call, but we elect to use an even simpler was to unsubscribe since we don't want to keep any state.
```java
messageBus.unsubscribe(example1);
```
which will unsubscribe all previous subscribers of `example` object. Technically we don't have to do that as all subscribers are implicitely unsubscribed when the instance of the object they are part of is garbage collected. However it is cleaner to excplicitely unsubscribe and release any held resources.

### More on subscribers


