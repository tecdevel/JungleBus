# JungleBus
Transactional service bus built on top of Amazon Web Services.  
[![Build status](https://ci.appveyor.com/api/projects/status/ccy923lmviskyfww/branch/master?svg=true)](https://ci.appveyor.com/project/rfleming71/junglebus/branch/development)  
Since version 3.0, the bus is built upon the JungleQueue library.

# Creating the Bus
Creates a service bus that can send and recieve messages
```C#
var bus = BusBuilder.Create("Development")
	.WithObjectBuilder(new JungleQueue.StructureMap.StructureMapObjectBuilder())
	.UsingJsonSerialization()
	.PublishingMessages(typeof(TestMessage).Assembly.ExportedTypes, RegionEndpoint.USEast1)
	.SetInputQueue("Test_Queue1", RegionEndpoint.USEast1)
	.SetSqsPollWaitTime(14)
	.UsingEventHandlersFromEntryAssembly()
	.SetNumberOfPollingInstances(2)
	.CreateStartableBus();

bus.StartReceiving();
bus.CreateSendBus().Publish(new TestMessage());
```

#Example message handler
```C#
public class Handler2 : IHandleMessage<TestMessage>
{
	private readonly IBus _bus;
	private readonly ILog _log;
	public Handler2(IBus bus, ILog log)
	{
		_bus = bus;
		_log = log;
	}

	public void Handle(TestMessage message)
	{
		_log.Info("Starting message Handler 2");
		_bus.Publish(new TestMessage2()); // Publish event to the world
		_bus.PublishLocal(new TestMessage3())); // Publish event to self
		_log.Info("Finished message Handler 2");
	}
}
```

# Other bus types	
## Send Only Bus
Creates a service bus that can only send messages
```C#
var busBuilder = BusBuilder.Create()
	.WithStructureMapObjectBuilder()
	.UsingJsonSerialization()
	.PublishingMessages(typeof(TestMessage).Assembly.ExportedTypes, RegionEndpoint.USEast1)
	.CreateSendOnlyBusFactory();
busBuilder().Publish(new TestMessage());
```

## Receive Only Bus
Creates a service bus that can only receive messages. This bus can send messages to itself.
```C#
var bus = BusBuilder.Create()
	.WithObjectBuilder(new JungleQueue.StructureMap.StructureMapObjectBuilder())
	.UsingJsonSerialization()
	.SetInputQueue("Test_Queue1", RegionEndpoint.USEast1)
	.SetSqsPollWaitTime(14)
	.UsingEventHandlersFromEntryAssembly()
	.SetNumberOfPollingInstances(2)
	.PublishingLocalEventsOnly()
	.CreateStartableBus();
bus.StartReceiving();
```
