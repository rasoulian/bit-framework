# Bit Web API

Web API is a powerful library to develop rest api in .NET world. If you're not familiar with that, you can [get start here](https://www.asp.net/web-api).

Using bit you'll get more benefits from web api. This includes following:

1. We've configured web api on top of [owin](http://owin.org). Owin stands for "Open web interface for .NET" and allows your code to work almost anywhere. We've developed extra codes to make sure your app works well on following workloads:
    - Traditional ASP.NET/IIS on Windows servers + Azure web/app services
    - ASP.NET Core/Kestrel on Windows & Linux Servers + Azure web/app services
    - Self host Windows services + Azure web jobs
2. We've configured web api on top of [asp.net core/owin request branching](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware). By default, all middlewares such as WebApi handle all incoming requests, but each request has one destination, for example signalr, web api, static file etc. By branching web api will handle api requests only which results into better performance.
3. We've developed extensive logging infrastructure in bit framework. It logs everything for you in your app, including web api traces. We've tons of log stores including but not limited to Windows Event Logs, Application Insights, Sql Server etc.
4. We've configured headers like [X-Content-Type-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options), [X-CorrelationId](http://theburningmonk.com/2015/05/a-consistent-approach-to-track-correlation-ids-through-microservices/) etc to improve logging, security etc.
5. You can protect your web api with bit identity server, a modern single sign on server based on open id/oauth 2.0

## Getting started

Run following command after you installed [git for windows](https://git-scm.com/download/win) (You can use any git clinet)
```shell
git clone https://github.com/bit-foundation/bit-framework.git
```

Then open Samples/WebApiSamples/WebApiSamples.sln and go to the 1SimpleWebApi project.

There are several classes there. Program and ValuesController are get copied from [this microsoft docs article](https://docs.microsoft.com/en-us/aspnet/web-api/overview/hosting-aspnet-web-api/use-owin-to-self-host-web-api). It's a good idea to read that article first.

For now, let's ignore the third class: "AppStartup".

Press F5, you'll see the result in your default browser. This project starts a self host server with power of [Microsoft.Owin.SelfHost](https://www.nuget.org/packages/Microsoft.Owin.SelfHost/), which allows you to self host bit based apps under windows services / console apps.


What's AppStartup class anyway? It configures your app. You'll understand its parts while you're reading docs, but for now let's focus on following codes only:

```csharp
dependencyManager.RegisterDefaultWebApiConfiguration();

dependencyManager.RegisterWebApiMiddleware(webApiDependencyManager =>
{
    webApiDependencyManager.RegisterWebApiMiddlewareUsingDefaultConfiguration();

    webApiDependencyManager.RegisterGlobalWebApiCustomizerUsing(httpConfiguration =>
    {
        // You've access to web api's http configuration here.
    });
});
```

That code configures web api into your app using the default configuration. Default configuration is all about security, performance, logging etc.

Bit is an extensible framework developed based on best practices. We've extensively used dependency injection in our code base and you can customize default behaviors and conventions based on your requirements.

In following samples, you can find out how to customize web api in bit, but feel free to [drops us an issue in github](https://github.com/bit-foundation/bit-framework/issues), ask a question on [stackoverflow.com](http://stackoverflow.com/questions/tagged/bit-framework) or use disqus comments below if you can't find what you want in these samples.


### Web API - Swagger (Open-API) configuration sample

Swagger is the World's Most Popular API Tooling. by using this real world sample, you can find out how to customize web api in bit projects.

Read [first part of "Swagger and ASP.NET Web API"](http://wmpratt.com/swagger-and-asp-net-web-api-part-1/)

Differences between our sample (2WebApiSwagger project) and that article:

1- There is no App_Start folder in bit projects by default. It's not needed.


2- In following code:

```csharp
GlobalConfiguration.Configuration
  .EnableSwagger(c => c.SingleApiVersion("v1", "A title for your API"))
  .EnableSwaggerUi();
```

[GlobalConfiguration](https://msdn.microsoft.com/en-us/library/system.web.http.globalconfiguration.aspx) uses ASP.NET pipeline directly and it does not work on ASP.NET Core. But bit's config works on both ASP.NET & ASP.NET Core.


3- Instead of adding Swashbuckle package, you've to install Swashbuckle.Core package. Swashbuckle package relies on ASP.NET pipeline and it does not work on ASP.NET Core.


4- In following code:
```csharp
c.IncludeXmlComments(string.Format(@"{0}\bin\SwaggerDemoApi.XML",           
                           System.AppDomain.CurrentDomain.BaseDirectory));
```
We've used #DefaultPathProvider.Current.GetCurrentAppPath()# instead of #System.AppDomain.CurrentDomain.BaseDirectory# which works fine on both ASP.NET/ASP.NET Core. We also use Path.Combine which works fine on linux servers instead of string.Format and "\" character usage.


5- We've following code which has no equivalent in the article codes:
```csharp
c.ApplyDefaultApiConfig(httpConfiguration);
```
As you see in the article, you open swgger ui by opening http://localhost:51609/swagger/ but in bit's sample, you open http://localhost:9000/api/swagger/. You open /swagger under /api. This is a magic of owin/asp.net core's request branching. Calling method "ApplyDefaultApiConfig" describs that magic to swagger. It also performs a bunch of other recommneded swagger configs for you too.

EnableBitSwaggerUI provides better UX for Swagger UI. As an example, it simplifies your login experience. It also stores your token, so you don't have to login everytime you open swagger.

So run the second sample and you're good to go (-:

### Web API file upload sample

There is a [question](https://stackoverflow.com/questions/10320232/how-to-accept-a-file-post) on stackoverflow.com about web api file upload.
The important thing you've to notice is "You don't have to use System.Web.dll classes in bit world, even when you're hosting your app on traditional asp.net


By removing usages of that dll, you're going to make sure that your code works well on both asp.net & asp.net core. So drop using #HttpContext.Current and all other members of System.Web.dll#. Note that if you install Bit.CodeAnalyzer nuget package, we warn you about this automatically.


Web API Attribute routing works fine in bit projects, but instead of [Route("api/file-manager/upload")] or [RoutePrefix("api/file-manager")], you've to write [Route("file-manager/upload")] or [RoutePrefix("file-manager)], this means **you should not write /api in your attribute routings**.


Remember to use async/await and CancellationToken in your Web API codes at it improves your app overall scalability and performance. Using CancellationToken, bit stops processing requests when user/operator cancels her request. (By closing the browser/mobile app or clicking on cancel button provided by you).

Open 3rd sample. It contains upload methods using Web API attribute routing and uses async/await & CancellationToken.

### Web API - Configuration on traditional "ASP.NET"

In 4th project (4WebApiAspNetHost), you'll find a bit web api project hosted on ASP.NET/IIS.

##### Differences between this project and previews projects:

1- Instead of [Microsoft.Owin.SelfHost] nuget package, we've installed [Microsoft.Owin.Host.SystemWeb]. Using that, you can host bit server side apps on top of ASP.NET/IIS. All codes you've developed are the same (We've copied codes from 2WebApiSwagger project in fact).

### Web API - Configuration on "ASP.NET Core / Full .NET Framework"

##### Differences between this project and first project:

1- Instead of [Microsoft.Owin.SelfHost] nuget package, we've installed [Bit.OwinCore] nuget package. Using Bit.OwinCore, you can host your app on top of asp.net core. ASP.NET core apps can be hosted almost anywhere.

Web API configuration and web api codes are all the same. (-:

You can easily start using ASP.NET Core 2.0 by installing Bit.OwinCore.AspNetCore2Upgrade nuget package. No code change is required.

### Web API - Configuration on "ASP.NET Core / .NET Core"

##### Differences between this project and first project:

1- As like as ASP.NET Core with full .NET framework, we've [Bit.OwinCore] instead of [Microsoft.Owin.SelfHost]

Web API configuration and web api codes are all the same. (-:

### Web API - Dependency Injection samples:

Bit's dependency injection covers you anywhere, from web api to signalr, background job workers, etc. As you see in AppStartup class of samples, there is a dependencyManager variable in both ASP.NET & ASP.NET Core projects. Classes you register/add are accessible using constructor and property injection anywhere you need them. Let's take a look at  [sample](https://github.com/bit-foundation/bit-framework/tree/master/Samples/WebApiSamples/7WebApiDependencyInjection)

As you see in that project, there is an IEmailService interface and DefaultEmailService class. We've registered that as following:

```csharp
dependencyManager.Register<IEmailService, DefaultEmailService>();
```

It uses [Autofac](https://autofac.org/) by default. Support for other IOC containers is going to be added soon.

You can also specify life cycle by calling .Register like following:

```csharp
dependencyManager.Register<IEmailService, DefaultEmailService>(lifeCycle: DependencyLifeCycle.PerScopeInstance);
```

It accepts three life cycles: PerScopeInstance, SingleInstance & Transient. PerScopeInstance creates a new instance of your class for every web request, every background job start, etc. But SingleInstance creates one instance and uses that anywhere. Transient creates a new instance everytime you resolve a service. Classes which are registered using PerScopeInstance have access to current user, and some classes like Entity framework's db context and repositories should be registered using PerScopeInstance.

You've also other Register methods like RegisterGeneric, RegisterInstance, RegisterTypes and RegisterUsing, you're welcomed to use those method if you're a DI ninja ;D

You can also cast dependency manager to IAutofacDependencyManager, and after that, you can access [ContainerBuilder](http://docs.autofac.org/en/latest/register/registration.html) of autofac.

```csharp
ContainerBuilder autofacContainerBuilder = ((IAutofacDependencyManager)dependencyManager).GetContainerBuidler();
```

You also have access to [IServiceCollection](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection) too.

If you've got a complex scenario, simply drops us an [issue on github](https://github.com/bit-foundation/bit-framework/issues) or ask a question on [stackoverflow](https://stackoverflow.com/questions/tagged/bit-framework).

### Exception handling:

Consider the following code:

```csharp
public void ValidateCustomer(Customer customer)
{
    if (customer.FirstName == customer.LastName))
        throw new Exception("Customer's first name and last name might not be identical"); // Sample rule.
}
public async Task SaveCustomer(Customer customer, CancellationToken cancellationToken)
{
    ValidateCustomer(customer);
    // Save customer using entity framework's db context for example.
}
```

When you try to save a customer, there might be two types of exceptions: "Known" and "Unknown" exceptions.

It is known that a customer with identical first name and last name would not be saved to the database, but the client does not except sql exception because your database server is down!

Every **unknown exception** results into following response:

```
StatusCode: 500
ReasonPharse: "UnKnownError"
Message: "UnKnownError"
```

**All exceptions are unknown**, except following:

DomainLogicException, ResourceNotFoundException, BadRequestException

So, let's rewrite the last example again:

```csharp
public void ValidateCustomer(Customer customer)
{
    if (customer.FirstName == customer.LastName))
        throw new DomainLogicException("Customer's first name and last name might not be identical"); // Sample rule.
}
```
The response would be:

```
StatusCode: 500
ReasonPharse: "KnownError"
Message: "Customer's first name and last name might not be identical"
```

Bad request exception is as like as DomainLogicException, but it results in a response with (400-BadRequest) status code. The status code of ResourceNotFoundException would be 404

Let's take a look at another example:

```csharp
public async Task UpdateCustomer(int customerId, string newName)
{
    // Customer customer = try to find customer in database first...
    if (customer == null)
        throw new ResourceNotFoundException($"Customer with Id {customerId} was not found");
}
```

Notes:

1- Every response has a header called X-CorrelationId (RequestId). When we log exceptions for you, it has a X-CorrelationId, so you can associate a request/response to logs.

2- When your app is in debug mode, exceptions details are written into responses. So if "DebugModel" is set to "true" in environments.json, you see exception details, no matter the exception is known or not, but if it is set to "false", then you see "UnKnownException" for unknown exceptions and exception's message for known exceptions.

Pro tip: If you prefer to create new "Known" exception types, [take a look at following question in stackoverflow.com](https://stackoverflow.com/a/46202377/2720104)

### Logging:

Everything is being logged in memory, and there are some log stores such as Windows event logs, Visual Studio's console etc to make them permanent.

Log contains lots of important data such as user info, machine info, app info etc.

To add event log stores you can use following code:

```csharp
dependencyManager.RegisterDefaultLogger(typeof(WindowsEventsLogStore).GetTypeInfo(), typeof(DebugLogStore).GetTypeInfo()); // You can add as many as event log stores you want.
```