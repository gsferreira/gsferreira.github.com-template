---
layout: post
title: Azure WebJobs dependency injection with Unity 
category: Azure
---

In the early days of **Azure WebJobs** use **Dependency Injection** was a little bit tricky because the Triggered methods were only static methods.

Since the WebJobs SDK 1.0.1 you are no longer limited to static methods and that can change how you approach Dependency Injection on WebJobs.

In this blog post I will show how you can use **[Unity](https://github.com/unitycontainer/unity)** to inject dependencies into your WebJobs.

<!--excerpt-->

##Setup dependency injection

*In this post I'm assuming you already know how to set up a [WebJob](https://azure.microsoft.com/en-gb/documentation/articles/websites-dotnet-webjobs-sdk-get-started/).*

The first step is to install the Unity Package in your project.

	Install-Package Unity


Then, we will need to implement the **IJobActivator** Interface, that accepts a Unity container. Here, we will provide the capabilitie of resolve the Instances using the Unity container.

Let's name the class as **"UnityJobActivator"** and fill it with the following code:


	public class UnityJobActivator : IJobActivator
	{
		private readonly IUnityContainer _container;
		public UnityJobActivator(IUnityContainer container)
		{
			_container = container;
		}
		public T CreateInstance<T>()
		{
			return _container.Resolve<T>();
		}
	}



As you can see, the **UnityJobActivator** receives in the constructor the Unity Container. Now let's setup the container.

*I like to use a Unity Configuration Class to organize my Unity configurations.*

	public class UnityConfig
	{
		private static Lazy<IUnityContainer> container = new Lazy<IUnityContainer>(() =>
		{
			var container = new UnityContainer();
			RegisterTypes(container);
			return container;
		});
		public static IUnityContainer GetConfiguredContainer()
		{
			return container.Value;
		}
		public static void RegisterTypes(IUnityContainer container)
		{
			container.RegisterType<IMailService, MailService>(new HierarchicalLifetimeManager());
		}
	}

Now that we have the container defined, we can specify the **JobActivator** that the WebJob should use. 

It can be accomplished using the Job Host Configuration (this is done in *Program.cs* in the main method), as you can see in the following code:

	JobHostConfiguration config = new JobHostConfiguration()
	{
		JobActivator = new UnityJobActivator(UnityConfig.GetConfiguredContainer())
	};
	var host = new JobHost(config);
	host.RunAndBlock();


Finally, open your functions, transform them in non static methods and receive the parameter of the type you want in the constructor.

	public class Functions
	{
		private readonly IMailService _mailService;
		public Functions(IMailService mailService)
		{
			_mailService = mailService;
		}
		
		public void ProcessQueueMessage([QueueTrigger("queue")] string message, TextWriter log)
		{
			_mailService.Send(message);

			log.WriteLine(message);
		}
	}


I hope that this helps you. 