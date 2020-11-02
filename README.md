# Create a web API with ASP.NET Core

Create a RESTful service with ASP.NET Core that supports Create, Read, Update, Delete (CRUD) operations.

This module explores creating a cross-platform RESTful service using ASP.NET Core web API with .NET Core and C#. An in-memory database will be created to persist the products data. Entity Framework (EF) Core will be used as the Object-Relational Mapper (O/RM) for reading and managing products data.

This module uses the [.NET Core CLI](https://docs.microsoft.com/en-us/dotnet/core/tools/) to demonstrate web API development. After completing this module, you can apply its concepts using a development environment like Visual Studio (Windows), Visual Studio for Mac (macOS), or Visual Studio Code (Windows, Linux, & macOS).

- Use the .NET Core CLI to create an ASP.NET Core web API project.
- Create an in-memory database for persisting products.
- Add CRUD action methods to the web API.
- Test the web API actions from the command shell.

# Create a web API project

The .NET Core CLI is the simplest way to create an ASP.NET Core web API. The CLI is pre-installed in the Azure Cloud Shell environment used in this unit.

use the .NET Core CLI to create a web API within the Cloud Shell command shell to the right. You'll also gain an understanding of the resulting project.

## Set up development environment

```bash
. <(wget -q -O - https://aka.ms/build-web-api-net-core-setup)
```

The preceding command installs a specific version of the .NET Core SDK in the Azure Cloud Shell environment.

# Create a web API project

## Scaffold and explore a web API project

1. Run the following .NET Core CLI command in the command shell:

    ```.NET Core CLI
    dotnet new webapi -o aspnet-learn/src/ContosoPets.Api
    ```

    The preceding command uses an ASP.NET Core project template, aliased as webapi, to scaffold a C#-based starter web API project. The _aspnet-learn/src/ContosoPets.Api_ directory structure is created, which contains an ASP.NET Core project targeting .NET Core. The project name matches the ContosoPets.Api directory name.

2. Run the following command in the command shell:

    ```Bash
    cd ./aspnet-learn/src/ContosoPets.Api
    ```

    The current directory changes to the newly created ContosoPets.Api directory.

3. Run the following command in the command shell:

    ```Bash
    code .
    ```

    The _ContosoPets.Api_ project directory opens in the [Azure Cloud Shell editor](https://docs.microsoft.com/en-us/azure/cloud-shell/using-cloud-shell-editor).

4. Examine the following files and directories:

    TABLE 1
    | Name | Description |
    |------|-------------|
    | Controllers/ | 	Contains classes with public methods exposed as HTTP endpoints. |
    | Program.cs | 	Contains a Main method—the app's managed entry point. |
    | ContosoPets.Api.csproj | 	Contains configuration metadata for the project. |
    | Startup.cs | 	Configures services and the app's HTTP request pipeline. |

## Build and test

1. Run the following command to build the app:

    ```Bash
    dotnet build
    ```

    The build succeeds with no warnings. If the build fails, check the output for troubleshooting information.

2. Run the following .NET Core CLI command in the command shell:

    ```Bash
    dotnet ./bin/Debug/netcoreapp3.1/ContosoPets.Api.dll \
        > ContosoPets.Api.log &
    ```

    The preceding command:

    - Hosts the web API with ASP.NET Core's Kestrel web server.
    - Displays the background task's process ID.

    .NET Core emits logging information and blocks command shell input. The command shell needs to be usable to test the running app. Therefore, the dotnet run output is redirected to a ContosoPets.Api.log text file. Additionally, the & runs the app as a background task to unblock command shell input.

    The web API is hosted at both `http://localhost:5000` and `https://localhost:5001`. This module uses the secure URL beginning with `https`.

    > Important:
    > Check _ContosoPets.Api.log_ if you encounter any unexpected behavior. If the build fails or other errors occur, the log file's information helps troubleshoot. If you make code changes, run `kill $(pidof dotnet)` to stop all .NET Core apps before attempting to run again.

3. Send an HTTP GET request to the web API:

    ```Bash
    curl -k -s https://localhost:5001/weatherforecast | jq
    ```

    Note: [curl](https://curl.haxx.se/) is a cross-platform command-line tool for testing web APIs and other HTTP endpoints.

    The preceding command uses:

    - HTTPS to send a request to the web API running on port 5001 of localhost. The `WeatherForecastController` class' parameterless `Get` action method handles the request.
    - The `-k` option to indicate that `curl` should allow insecure server connections when using HTTPS. The .NET Core SDK includes an HTTPS development certificate for testing. By default, curl rejects secure connections using this certificate.
    - The `-s` option to suppress all output except the JSON payload. The JSON is sent to the `jq` command-line JSON processor for improved display.

    The following represents an excerpt of the JSON that is returned:

    ```JSON
    [
        {
            "date": "2020-05-19T21:18:21.0596135+00:00",
            "temperatureC": 18,
            "temperatureF": 64,
            "summary": "Bracing"
        },
        {
            "date": "2020-05-20T21:18:21.0599683+00:00",
            "temperatureC": 27,
            "temperatureF": 80,
            "summary": "Hot"
        },
        // ...
    ]
    ```

4. Stop all processes produced by the .NET Core app:

    ```Bash
    kill $(pidof dotnet)
    ```

    Now that the web API has been created, let's modify it to meet the retailer's needs.

# Add a data store

A type of class called a Model is needed to represent a dog toy in inventory. The Model must include the properties of a product and is used to pass data in the web API. The Model is also used to persist dog toys in a data store. In this unit, that data store will be created as an [in-memory EF Core database](https://docs.microsoft.com/en-us/ef/core/providers/in-memory/?tabs=dotnet-core-cli).

An in-memory database is used in this unit for simplicity. Choose a different data store for production environments, such as SQL Server or Azure SQL Database.

> Important:
If the Cloud Shell session ever times out or disconnects, reconnect and run the following command after reconnecting to set the working directory to ~/aspnet-learn/src/ContosoPets.Api and launch the editor:

```Bash
cd ~/aspnet-learn/src/ContosoPets.Api && code .
```

1. Run the following command:

    ```.NET Core CLI
    dotnet add package Microsoft.EntityFrameworkCore.InMemory
    ```

    The preceding command:

    - Adds the specified NuGet package reference to the project.
    - Downloads the specified NuGet package and its dependencies.

    The `Microsoft.EntityFrameworkCore.InMemory` package is required to use EF Core in-memory databases.

2. Run the following command:

    ```Bash
    mkdir Models && touch $_/Product.cs
    ```

    A _Models_ directory is created in the project root with an empty _Product.cs_ file. The directory name _Models_ is a convention. The directory name comes from the __Model__-View-Controller architecture used by the web API.

3. Update file explorer by clicking the editor's refresh icon.

4. Add the following code to _Models/Product.cs_ to define a product. Save your changes.

    ```C#
    using System.ComponentModel.DataAnnotations;

    namespace ContosoPets.Api.Models
    {
        public class Product
        {
            public long Id { get; set; }

            [Required]
            public string Name { get; set; }

            [Required]
            [Range(minimum: 0.01, maximum: (double) decimal.MaxValue)]
            public decimal Price { get; set; }
        }
    }
    ```

    The `Name` and `Price` properties are marked as required to ensure values are provided when creating a `Product` object. Additionally, the `Price` property enforces minimum and maximum values.

5. Run the following command:

    ```Bash
    mkdir Data && touch $_/ContosoPetsContext.cs $_/SeedData.cs
    ```

    A _Data_ directory is created in the project root with empty _ContosoPetsContext.cs_ and _SeedData.cs_ files.

6. Refresh file explorer, and add the following code to _Data/ContosoPetsContext.cs_. Save your changes.

    ```C#
    using Microsoft.EntityFrameworkCore;
    using ContosoPets.Api.Models;

    namespace ContosoPets.Api.Data
    {
        public class ContosoPetsContext : DbContext
        {
            public ContosoPetsContext(DbContextOptions<ContosoPetsContext> options)
                : base(options)
            {
            }

            public DbSet<Product> Products { get; set; }
        }
    }
    ```

    The preceding code creates a Contoso Pets-specific implementation of an EF Core `DbContext` object. The `ContosoPetsContext` class provides access to an in-memory database, as configured in the next step.

7. Add the following highlighted code to the _Startup.cs_ file's `ConfigureServices` method. Save your changes.

    ```C#
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddDbContext<ContosoPetsContext>(options =>
            options.UseInMemoryDatabase("ContosoPets"));
        services.AddControllers();
    }
    ```

    The preceding code:

    - Registers the custom `DbContext` class, named `ContosoPetsContext`, with ASP.NET Core's [dependency injection](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-3.1) system.
    - Defines an in-memory database named _ContosoPets_.

8. Add the following code to the top of _Startup.cs_. Save your changes

    ```C#
    using Microsoft.EntityFrameworkCore;
    using ContosoPets.Api.Data;
    ```

9. Add the following code to Data/SeedData.cs. Save your changes.

    ```C#
    using System;
    using System.Linq;
    using Microsoft.EntityFrameworkCore;
    using Microsoft.Extensions.DependencyInjection;
    using ContosoPets.Api.Models;

    namespace ContosoPets.Api.Data
    {
        public static class SeedData
        {
            public static void Initialize(ContosoPetsContext context)
            {
                if (!context.Products.Any())
                {
                    context.Products.AddRange(
                        new Product
                        {
                            Name = "Squeaky Bone",
                            Price = 20.99m,
                        },
                        new Product
                        {
                            Name = "Knotted Rope",
                            Price = 12.99m,
                        }
                    );

                    context.SaveChanges();
                }
            }
        }
    }
    ```

    The preceding code defines a static `SeedData` class. The class's `Initialize` method seeds the in-memory database with two dog toys.

10. Replace the code in _Program.cs_ with the following code. Save your changes.

    ```C#
    using System;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.Extensions.Hosting;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Logging;
    using ContosoPets.Api.Data;

    namespace ContosoPets.Api
    {
        public class Program
        {
            public static void Main(string[] args)
            {
                var host = CreateHostBuilder(args).Build();
                SeedDatabase(host);
                host.Run();
            }

            public static IHostBuilder CreateHostBuilder(string[] args) =>
                Host.CreateDefaultBuilder(args)
                    .ConfigureWebHostDefaults(webBuilder =>
                    {
                        webBuilder.UseStartup<Startup>();
                    });

            private static void SeedDatabase(IHost host)
            {
                var scopeFactory = host.Services.GetRequiredService<IServiceScopeFactory>();
                using var scope = scopeFactory.CreateScope();
                var context = scope.ServiceProvider.GetRequiredService<ContosoPetsContext>();

                if (context.Database.EnsureCreated())
                {
                    try
                    {
                        SeedData.Initialize(context);
                    }
                    catch (Exception ex)
                    {
                        var logger = scope.ServiceProvider.GetRequiredService<ILogger<Program>>();
                        logger.LogError(ex, "A database seeding error occurred.");
                    }
                }
            }
        }
    }
    ```

    The `Program.Main` method is the first code to execute when the app starts. With the preceding changes, seeding of the in-memory database is triggered via a call to `SeedData.Initialize`.

     > Important: This database seeding strategy isn't recommended in a production environment. Consider seeding during database deployment instead.

11. Run the following command to build the app:

    ```.NET Core CLI
    dotnet build
    ```

    The build succeeds with no warnings. If the build fails, check the output for troubleshooting information.

    The `Product` Model and `ContosoPetsContext` class will be used by the controller created in the next unit.

## Add a controller

A _Controller_ is a public class with one or more public methods known as _actions_. By convention, a Controller class is placed in the project root's _Controllers_ directory. The actions are exposed as callable HTTP endpoints inside the web API controller.

1. Run the following command:

    ```bash
    touch ./Controllers/ProductsController.cs
    ```

    An empty class file named _ProductsController.cs_ is created in the _Controllers_ directory. The directory name _Controllers_ is a convention. The directory name comes from the Model-View-__Controller__ architecture used by the web API.

    > Note: By convention, controller class names are suffixed with _Controller_.

2. Refresh file explorer, and add the following code to _Controllers/ProductsController.cs_. Save your changes.

    ```C#
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.EntityFrameworkCore;
    using ContosoPets.Api.Data;
    using ContosoPets.Api.Models;

    namespace ContosoPets.Api.Controllers
    {
        [ApiController]
        [Route("[controller]")]
        public class ProductsController : ControllerBase
        {
            private readonly ContosoPetsContext _context;

            public ProductsController(ContosoPetsContext context)
            {
                _context = context;
            }

            [HttpGet]
            public ActionResult<List<Product>> GetAll() =>
                _context.Products.ToList();

            // GET by ID action

            // POST action

            // PUT action

            // DELETE action
        }
    }
    ```

    This class derives from `ControllerBase`, the base class for an MVC controller without web UI support. The following attributes define its behavior:

    - `[Route]` defines the routing pattern `[controller]`. The `[controller]` token is replaced by the controller's name (case-insensitive, without the _Controller_ suffix), so requests to `https://localhost:5001/products` are handled by this controller.
    - `[ApiController]` adds behaviors that make it easier to build web APIs. Some behaviors include [parameter source inference](https://docs.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-3.1#binding-source-parameter-inference), [attribute routing as a requirement](https://docs.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-3.1#binding-source-parameter-inference) , and [model validation error handling enhancements](https://docs.microsoft.com/en-us/aspnet/core/web-api/?view=aspnetcore-3.1#automatic-http-400-responses).

    Within the class definition:

    - [Constructor injection](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/dependency-injection?view=aspnetcore-3.1#constructor-injection) provides an instance of `ContosoPetsContext` to the controller.
    - An HTTP GET action named `GetAll` is created for retrieving all products.

    > Note: The route may contain static strings, as in `api/[controller]`. In that example, a request to `https://localhost:5001/api/products` would be handled by this controller.

3. Run the following command to build the app:

    ```bash
    dotnet build --no-restore
    ```

    The `--no-restore` option is included because no NuGet packages were added since the last build. The build process bypasses restoration of NuGet packages and succeeds with no warnings. If the build fails, check the output for troubleshooting information.

4. Start the web API by running the following command:

    ```bash
    dotnet ./bin/Debug/netcoreapp3.1/ContosoPets.Api.dll \
    > ContosoPets.Api.log &
    ```

5. Run the following command:

    ```bash
    curl -k -s https://localhost:5001/products | jq
    ```

    The following JSON is returned:

    ```JSON
    [
        {
            "id": 1,
            "name": "Squeaky Bone",
            "price": 20.99
        },
        {
            "id": 2,
            "name": "Knotted Rope",
            "price": 12.99
        }
    ]
    ```

6. Stop all processes produced by the .NET Core app:

    ```bash
    kill $(pidof dotnet)
    ```

Actions to support CRUD operations on products data will be added to `ProductsController` in the next unit.

# Implement CRUD operations

When the retailer's storefront UI is built, it should display all products in the inventory. To fulfill such a requirement, an action responding to an HTTP GET action verb is needed.

The following table depicts the relationship between HTTP action verbs, CRUD operations, and ASP.NET Core attributes. For example, an HTTP PUT action verb is most often used to support an update operation. Such an action is annotated with the `[HttpPut]` attribute.

| HTTP action verb | CRUD operation | ASP.NET Core attribute |
|------------------|----------------|------------------------|
| POST | Create | `[HttpPost]`    |
| GET | Read | `[HttpGet]`     |
| PUT | Update | `[HttpPut]`     |
| DELETE | Delete | `[HttpDelete]`  |

In addition to the action verbs in the preceding table, a web API in ASP.NET Core supports HEAD, OPTIONS, and PATCH.

The following sections demonstrate how to support each of these four actions in the web API.

# Retrieve a product

Replace the `// GET by ID action` comment in _Controllers/ProductsController.cs_ with the following code:

```C#
[HttpGet("{id}")]
public async Task<ActionResult<Product>> GetById(long id)
{
    var product = await _context.Products.FindAsync(id);

    if (product == null)
    {
        return NotFound();
    }

    return product;
}
```

The preceding action:

- Responds only to the HTTP GET verb, as denoted by the `[HttpGet]` attribute.
- Requires that the `id` value is included in the URL segment after `products/`. Remember, the `/products` pattern was defined by the controller-level [Route] attribute.
- Queries the database for a product matching the provided id parameter.

Each `ActionResult` used in the preceding action is mapped to the corresponding HTTP status code in the following table.

| ASP.NET Core action result | HTTP status code | Description |
|----------------------------|------------------|-------------|
| Ok is implied | 200 | A product matching the provided id parameter exists in the database. The product is included in the response body in the media type as defined in the Accept HTTP request header (JSON by default). |
| NotFound | 404 | A product matching the provided id parameter doesn't exist in the database. |

# Add a product

Replace the `// POST action` comment in _Controllers/ProductsController.cs_ with the following code:

```C#
[HttpPost]
public async Task<IActionResult> Create(Product product)
{
    _context.Products.Add(product);
    await _context.SaveChangesAsync();

    return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
}
```

The preceding action:

- Responds only to the HTTP POST verb, as denoted by the `[HttpPost]` attribute.
- Inserts the request body's `Product` object into the database.

> Note: Because the controller is annotated with the `[ApiController]` attribute, it's implied that the `product` parameter will be found in the request body.

The first parameter in the `CreatedAtAction` method call represents an action name. The `nameof` keyword is used to avoid hard-coding the action name. `CreatedAtAction` uses the action name to generate a `Location` HTTP response header with a URL to the newly created product.

Each `ActionResult` used in the preceding action is mapped to the corresponding HTTP status code in the following table.

| ASP.NET Core action result | HTTP status code | Description |
|----------------------------|------------------|-------------|
| `CreatedAtAction` | 201 | The product was added to the database. The product is included in the response body in the media type as defined in the `Accept` HTTP request header (JSON by default). |
| `BadRequest` is implied | 400 | The request body's Product object is invalid.|


# Modify a product

Replace the `// PUT action` comment in _Controllers/ProductsController.cs_ with the following code:

```C#
[HttpPut("{id}")]
public async Task<IActionResult> Update(long id, Product product)
{
    if (id != product.Id)
    {
        return BadRequest();
    }

    _context.Entry(product).State = EntityState.Modified;
    await _context.SaveChangesAsync();

    return NoContent();
}
```

The preceding action:

- Responds only to the HTTP PUT verb, as denoted by the [HttpPut] attribute.

- Returns `IActionResult` because the `ActionResult` return type isn't known until runtime. The `BadRequest` and `NoContent` methods return `BadRequestResult` and `NoContentResult` types, respectively.

- Requires that the `id` value is included in the URL segment after `products/`.

- Updates the `Name` and `Price` properties of the product. The following code instructs EF Core to mark all of the `Product` entity's properties as modified:

    ```C#
    _context.Entry(product).State = EntityState.Modified;
    ```

    It's a more maintainable alternative to individual property assignments that replaces the following hypothetical code:

    ```C#
    product.Name = productIn.Name;
    product.Price = productIn.Price;
    ```

> Note: Because the controller is annotated with the `[ApiController]` attribute, it's implied that the `product` parameter will be found in the request body.

Each `ActionResult` used in the preceding action is mapped to the corresponding HTTP status code in the following table.

| ASP.NET Core action result | HTTP status code | Description |
|----------------------------|------------------|-------------|
| `NoContent` | 204 | The product was updated in the database.
| `BadRequest` | 400 | The request body's Id value doesn't match the route's | id value. |
| `BadRequest` is implied | 400 | The request body's Product object is invalid. |

# Remove a product

Replace the `// DELETE action` comment in `Controllers/ProductsController.cs` with the following code:

```C#
[HttpDelete("{id}")]
public async Task<IActionResult> Delete(long id)
{
    var product = await _context.Products.FindAsync(id);

    if (product == null)
    {
        return NotFound();
    }

    _context.Products.Remove(product);
    await _context.SaveChangesAsync();

    return NoContent();
}
```

The preceding action:

- Responds only to the HTTP DELETE verb, as denoted by the `[HttpDelete]` attribute.
- Requires that `id` is included in the URL path.
- Queries the database for a product matching the provided id parameter.

Each `ActionResult` used in the preceding action is mapped to the corresponding HTTP status code in the following table.

| ASP.NET Core action result | HTTP status code | Description |
|----------------------------|------------------|-------------|
| NoContent | 204 | The product was deleted from the database. |
| NotFound | 404 | A product matching the provided id parameter doesn't exist in the database. |

# Build and run

1. Run the following command to build the app:

    ```C#
    dotnet build --no-restore
    ```

    The `--no-restore` option is included because no NuGet packages were added since the last build. The build process bypasses restoration of NuGet packages and succeeds with no warnings. If the build fails, check the output for troubleshooting information.

2. Start the web API by running the following command:

    ```bash 
    dotnet ./bin/Debug/netcoreapp3.1/ContosoPets.Api.dll \
        > ContosoPets.Api.log &
    ```

    The web API is running and is ready for testing via curl.

> Important: Don't forget to check +ContosoPets.Api.log_ for troubleshooting information, if required.

# Test web API actions

Now that the CRUD actions have been added to the web API, it's time to test them.

1. Send an invalid HTTP POST request to the web API:

    ```bash
    curl -i -k \
        -H "Content-Type: application/json" \
        -d "{\"name\":\"Plush Squirrel\",\"price\":0.00}" \
        https://localhost:5001/products
    ```

    In the preceding command:

    - `-i` displays the HTTP response headers.
    - `-d` implies an HTTP POST operation and defines the request body.
    - `-H` indicates that the request body is in JSON format. The header's value overrides the default content type of `application/x-www-form-urlencoded`.

    The command returns an HTTP 400 status code because the controller's `[ApiController]` attribute triggers Model validation on the request body. MVC's Model binder attempts to convert the request's `-d` JSON to a Product object. Model validation fails because the request's `Price` value is less than the minimum value of 0.01.

    ```text

    HTTP/1.1 400 Bad Request
    Date: Mon, 18 May 2020 21:04:05 GMT
    Content-Type: application/problem+json; charset=utf-8
    Server: Kestrel
    Transfer-Encoding: chunked

    {
    "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
    "title": "One or more validation errors occurred.",
    "status": 400,
    "traceId": "|34d8eb25-4f589a8a1ec12a3f.",
    "errors": {
        "Price": [
        "The field Price must be between 0.01 and 7.922816251426434E+28."
        ]
    }
    }

    ```

2. Send a valid HTTP POST request to the web API:

    ```bash
    curl -i -k \
        -H "Content-Type: application/json" \
        -d "{\"name\":\"Plush Squirrel\",\"price\":12.99}" \
        https://localhost:5001/products
    ```

    The following text represents the response:

    ```text
    HTTP/1.1 201 Created
    Date: Mon, 18 May 2020 21:07:17 GMT
    Content-Type: application/json; charset=utf-8
    Server: Kestrel
    Transfer-Encoding: chunked
    Location: https://localhost:5001/products/3

    {"id":3,"name":"Plush Squirrel","price":12.99}
    ```

    Successful creation of the product results in:

    - An HTTP 201 status code
    - A `Location` response header with a URL to retrieve the newly created product
    - A JSON representation of the newly created product

3. Send an HTTP GET request to the web API:

    ```bash
    curl -k -s https://localhost:5001/products/3 | jq
    ```

    The following output is displayed, proving that the new product was persisted to the in-memory database:

    ```JSON
    {
        "id": 3,
        "name": "Plush Squirrel",
        "price": 12.99
    }
    ```

4. Send an HTTP PUT request to the web API:

    ```bash
    curl -i -k \
        -X PUT \
        -H "Content-Type: application/json" \
        -d "{\"id\":2,\"name\":\"Knotted Rope\",\"price\":14.99}" \
        https://localhost:5001/products/2
    ```

    The preceding command changes the price from 12.99 to 14.99. The retailer has decided to increase the price of the Knotted Rope product.

    The following text represents the response:

    ```text
    HTTP/1.1 204 No Content
    Date: Mon, 18 May 2020 21:08:48 GMT
    Server: Kestrel
    ```

5. Send an HTTP DELETE request to the web API:

    ```bash
    curl -i -k -X DELETE https://localhost:5001/products/1
    ```

    The preceding command deletes the product from the in-memory database. The supplier for the Squeaky Bone product has filed for bankruptcy. The product can no longer be ordered and the retailer has no inventory remaining.

    The following text represents the response:

    ```text
    HTTP/1.1 204 No Content
    Date: Mon, 18 May 2020 21:09:29 GMT
    Server: Kestrel
    ```

6. Send an HTTP GET request to the web API:

    ```bash
    curl -k -s https://localhost:5001/products | jq
    ```

    The updated inventory is displayed:

    ```JSON
    [
        {
            "id": 2,
            "name": "Knotted Rope",
            "price": 14.99
        },
        {
            "id": 3,
            "name": "Plush Squirrel",
            "price": 12.99
        }
    ]
    ```

# Summary

## Download project files

Use the following command to package and download your project files:

```bash
pushd ~/aspnet-learn/src && \
    zip -r ~/aspnet-learn.zip . && \
    popd && \
    download ~/aspnet-learn.zip
```

# Learn more with a Channel 9 video series

- [.NET Core 101](https://channel9.msdn.com/Series/NET-Core-101/?WT.mc_id=Educationaldotnet-c9-scottha)
- [ASP.NET Core 101](https://channel9.msdn.com/Series/ASPNET-Core-101/?WT.mc_id=Educationaspnet-c9-niner)
