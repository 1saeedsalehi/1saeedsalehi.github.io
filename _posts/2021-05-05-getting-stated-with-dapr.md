---
title: Getting started with Dapr 
author: saeed
date: 2020-05-05T00:00 +0800
categories: [Software Engineering]
tags: [dapr,microservices,miniservices,sidecar,software-architecture]
---
If you're involved in developing applications with a microservices architecture, you've likely heard of Dapr.

In simple terms, Dapr aims to simplify microservices development by focusing on core application logic, keeping the code clean, and abstracting away infrastructural concerns. Dapr introduces itself as a runtime for running applications.

The initial release of this exciting product was on February 17, 2021, and it's ready for production use.

[Introduction](https://blog.dapr.io/posts/2021/02/17/announcing-dapr-v1.0/)

### Prerequisites

- [Install Dapr CLI](https://docs.dapr.io/getting-started/install-dapr-cli/)
- [Install .NET 5 SDK](https://dotnet.microsoft.com/download/dotnet/5.0)

### Getting Started

Let's create a simple ASP.NET Core project:

```bash
mkdir WeatherForecastService
cd WeatherForecastService
dotnet new webapi
dotnet run
```

As you know, this creates a basic ASP.NET Core project. The default template usually includes a controller that returns weather data.

### Running the API with Dapr
Dapr uses the [sidecar pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar). Dapr can run our API and expose it externally.

Run the following command in your project directory:
```bash
dapr run --app-id weatherforecastservice --dapr-http-port 3500 --app-port 5001 --app-ssl -- dotnet run
```

Let's break down this command:

- `app-id`: An identifier for your application/service, used for discovery.
- `dapr-http-port`: The port that the Dapr sidecar listens on.
- `app-port`: The port that your application/service listens on.
- `app-ssl`: Specifies whether requests to the service should use SSL.
- `dotnet run`: The command to run your project.



### Advantages of Using Dapr
Before running the project with Dapr, you can access it at [https://localhost:5001/weatherforecast](https://localhost:5001/weatherforecast). However, when running with Dapr, in addition to the above, you can invoke the method through the Dapr sidecar:

http://localhost:3500/v1.0/invoke/weatherforecastservice/method/weatherforecast


![dapr](/assets/img/dapr/getting-started.jpg)


This might seem simple at first glance. However, the crucial point is that it doesn't matter what platform the service behind Dapr is running on. For example, even if it's running on `gRPC`, we can easily invoke it using Dapr's APIs.

This provides a standard way to invoke methods between different services:

```
http://localhost:<dapr-http-port>/v1.0/invoke/<app-id>/method/<method-name>
```

Importantly, this method is language and technology **agnostic**. You can directly invoke a service developed in another language using this API.

But that's not all. This standard method for invoking services enables us to achieve much more in microservices development. Many of these features are built into Dapr:

- Service discovery
- Distributed tracing
- Metrics
- Error handling
- Encryption

A significant advantage of Dapr is that you don't need to think about all these issues while writing code. Instead, you can focus on proper architecture and implementing high-quality application logic.

Libraries are available to simplify using Dapr's features, which you can easily find on NuGet.



Summary
This post provided a brief introduction to Dapr. You can find more information at [dapr.io(https://dapr.io/)].

I'd appreciate it if you shared your thoughts.