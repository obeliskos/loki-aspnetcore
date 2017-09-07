# loki-aspnetcore
asp.net core translation layer for interfacing with loki-nodeservice

# Overview
This library contains generic reusable c# abstractions for interfacing with :
- LokiDatabase
- LokiCollections
- Query Chaining and Transforms
- loki-nodeservice statistics and collection information

# Nuget
This library has been published as [loki-aspnetcore on nuget](https://www.nuget.org/packages/loki-aspnetcore/) so applications wishing
to use this library should add a reference in their csproj file to : 
```xml
    <PackageReference Include="loki-aspnetcore" Version="1.0.0" />
```

# Usage

> Note that in order to use this library, you will need an npm library (loki-nodeservice) which we demonstrate more fully in our [loki-aspnetcore-example](https://github.com/obeliskos/loki-aspnetcore-example) repository.

Generally you will need to bootstrap logic into each Controller for interfacing with loki-aspnetcore and its node.js library
via a Controller constructor such as :
```csharp
        private LokiDatabaseConfiguration _demoServiceConfiguration;
        
        public UserController(IHostingEnvironment env)

        {
            _env = env;

            _demoServiceConfiguration = new LokiDatabaseConfiguration(
                "./node_modules/loki-nodeservice/lokiservice.js", 
                env.ContentRootPath.Replace("\\", "/") + "/nodesvcs/demo1-service.init.js", 
                "./dbinstances/demo-1.db"
            );            
        }
        
```
 
That somewhat ugly bit of path resolution should suffice for abstraction of the loki-nodeservice node module as well as your 
custom service initializer (also demonstrated in our [loki-aspnetcore-example](https://github.com/obeliskos/loki-aspnetcore-example)).
Your service initializer will be called by the loki-nodeservice for defining/creating/spinning up instanc(es) of the databases you initialize in 
your initializer. loki-nodeservice, in turn, will be called by nodeservices, which in turn, will be called by asp.net nodeservices, 
which is called by your controller.  The above definition of a global LokiDatabaseConfiguration object will contain most of the 
pathing 'glue' needed for that to work.

Having established that configuration object and your c# concrete classes, you might have controller actions which query a 
collection such as this view action : 
```csharp
        public async Task<IActionResult> Index([FromServices] INodeServices nodeServices)
        {
            LokiDatabase db = new LokiDatabase(nodeServices, _demoServiceConfiguration);

            List<User> result = await db.Find<User>("users");

            return View(result);
        }
```

Note the asp.net nodeservices dependency injection and our configuration object are passed to resolve a LokiDatabase instance 
which we can use to query.  The creation of the LokiDatabase is for establishing configuration only for subsequent method invocations 
to be able to resolve how to address the node service. 

# Detailed Example
_Please see our for an example on how you_
_might use this library, along with loki-nodeservice npm package and asp.net nodeservices to completely host, query, and_
_obtain statistic on lokijs database(s)._


