[TOC]

# Create a web API with ASP.NET Core

## Resources

- [Tutrial: Create a web API with ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-web-api?view=aspnetcore-6.0)
- [GitHub: first-web-api TodoApi](https://github.com/dotnet/AspNetCore.Docs/tree/main/aspnetcore/tutorials/first-web-api/samples/6.0)



## Tools

- .NET 6 SDK
- IDE: Visual Studio 2022
- [dotnet-aspnet-codegenerator](https://github.com/dotnet/AspNetCore.Docs/blob/main/aspnetcore/fundamentals/tools/dotnet-aspnet-codegenerator.md)
- API Test: Postman



Suggest to use Visual Studio 2022 on Windows to create a new .NET Web API project.

- Visual Studio for Mac currently only support .NET 5 and .NET 3.1
- VS Code requires manual run commands to install NuGet packages

## APIs



| API                          | Description             | Request body | Response body        | HTTP Status code |
| ---------------------------- | ----------------------- | ------------ | -------------------- | ---------------- |
| `GET /api/todoitems`         | Get all to-do items     | None         | Array of to-do items | `200 OK`         |
| `GET /api/todoitems/{id}`    | Get an item by ID       | None         | To-do item           | `200 OK`         |
| `POST /api/todoitems`        | Add a new item          | To-do item   | To-do item           | `201 Created`    |
| `PUT /api/todoitems/{id}`    | Update an existing item | To-do item   | None                 | `204 No Content` |
| `DELETE /api/todoitems/{id}` | Delete an item          | None         | None                 | `204 No Content` |



Notes:

- `204 No Content` means success no response body;



## Create a Web API project

Notes: Create a normal Web API with controllers (not minimal web API).



```bash
# Create a Web API project
dotnet new webapi -o TodoApi

# Add EntityFrameworkCore.InMemory package
cd TodoApi
dotnet add package Microsoft.EntityFrameworkCore.InMemory --prerelease

# Trust the HTTPS development certificate 
dotnet dev-certs https --trust

```



## Test the project

Run `Ctrl` + `F5` to run the app, and choose `.NET Core` at first time.

Access `https://localhost:{port}/swagger` to open the API list, and choose the API, and click "Try it out" and then click "Execute".

Or access the demo `GET /WeatherForecast` directly by `https://localhost:{port}/WeatherForecast`



Notes:

- The above `{port}` is a random port.



## TodoItems demo

### Update the launchUrl

In `*Properties\launchSettings.json`, update `launchUrl` from `swagger` to `api/todoitems`



Notes:

- The luanchUrl is the default url when launch the app and auto open browser, but this features looks only work perfect in Visual Studio 2022 on Windows

### Add a model class

Add `Models/TodoItem.cs`:

```c#
namespace TodoApi.Models
{
    public class TodoItem
    {
        public long Id { get; set; }
        public string? Name { get; set; }
        public bool IsComplete { get; set; }
    }
}
```



### Add a database context

Add `Models/TodoContext.cs`:

```c#
using Microsoft.EntityFrameworkCore;
using System.Diagnostics.CodeAnalysis;

namespace TodoApi.Models
{
    public class TodoContext : DbContext
    {
        public TodoContext(DbContextOptions<TodoContext> options)
            : base(options)
        {
        }

        public DbSet<TodoItem> TodoItems { get; set; } = null!;
    }
}
```



### Register the database context

Update `Program.cs` with the following code.



Import packages at the begining of the file:

```c#
using Microsoft.EntityFrameworkCore;
using TodoApi.Models;
```



Register the database context after `builder.Services.AddControllers()`:

```bash
builder.Services.AddDbContext<TodoContext>(opt =>
    opt.UseInMemoryDatabase("TodoList"));
```



### Scaffold a controller



```bash
# Add required NuGet packages
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design --prerelease
dotnet add package Microsoft.EntityFrameworkCore.Design --prerelease
dotnet add package Microsoft.EntityFrameworkCore.SqlServer --prerelease

# Install the scaffolding engine `dotnet-aspnet-codegenerator`
dotnet tool install -g dotnet-aspnet-codegenerator --version 6.0.1

# Scaffold the `TodoItemsController`
# -name : Name of the controller
# -async : Async controller actions
# -api : REST API
# -m : Modle class
# -dc : DbContext class
# -outDir: output folder
dotnet aspnet-codegenerator controller \
  -name TodoItemsController \
  -async \
  -api \
  -m TodoItem \
  -dc TodoContext \
  -outDir Controllers

```



View generated `Controllers/TodoitemsControllers.cs`



#### Root API URI of Controller

The root API URI of `TodoItemsController` is `api/todoitems`

```c#
[Route("api/[controller]")]
[ApiController]
public class TodoItemsController : ControllerBase
{
  // ...
}
```



#### POST - Add a new item

```c#
// POST: api/TodoItems
// To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
[HttpPost]
public async Task<ActionResult<TodoItem>> PostTodoItem(TodoItem todoItem)
{
  _context.TodoItems.Add(todoItem);
  await _context.SaveChangesAsync();

  return CreatedAtAction("GetTodoItem", new { id = todoItem.Id }, todoItem);
}
```



Use Postman to test `POST https://localhost:{port}/api/todoitems`.

Request body with JSON format is the new created item:

```json
{
  "id": 1,
  "name": "walk dog",
  "isComplete": true
}
```



Or access `https://localhost:{port}/swagger` to use Swagger UI to test APIs.



#### GET - Get all to-do items

```c#
// GET: api/TodoItems
[HttpGet]
public async Task<ActionResult<IEnumerable<TodoItem>>> GetTodoItems()
{
  return await _context.TodoItems.ToListAsync();
}
```



Use Postman to test `GET https://localhost:{port}/api/todoitems`.

No request body.

Response body with JSON format is an array of items:

```json
[
    {
        "id": 1,
        "name": "walk dog",
        "isComplete": true
    }
]
```



Or access `https://localhost:{port}/swagger` to use Swagger UI to test APIs.



#### GET - Get an item by ID

```c#
// GET: api/TodoItems/5
[HttpGet("{id}")]
public async Task<ActionResult<TodoItem>> GetTodoItem(long id)
{
  var todoItem = await _context.TodoItems.FindAsync(id);

  if (todoItem == null)
  {
    return NotFound();
  }

  return todoItem;
}
```



Use Postman to test `GET https://localhost:{port}/api/todoitems/1`.

No request body.

Response body with JSON format is an item:

```json
{
    "id": 1,
    "name": "walk dog",
    "isComplete": true
}
```



Or access `https://localhost:{port}/swagger` to use Swagger UI to test APIs.



#### PUT - Update an existing item

```c#
// PUT: api/TodoItems/5
// To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
[HttpPut("{id}")]
public async Task<IActionResult> PutTodoItem(long id, TodoItem todoItem)
{
  if (id != todoItem.Id)
  {
    return BadRequest();
  }

  _context.Entry(todoItem).State = EntityState.Modified;

  try
  {
    await _context.SaveChangesAsync();
  }
  catch (DbUpdateConcurrencyException)
  {
    if (!TodoItemExists(id))
    {
      return NotFound();
    }
    else
    {
      throw;
    }
  }

  return NoContent();
}
```



Use Postman to test `PUT https://localhost:{port}/api/todoitems/1`.

Request body with JSON format:

```json
{
    "id": 1,
    "name": "feed fish",
    "isComplete": true
}
```



No response body and HTTP Status code is `204 No Content`.

Or access `https://localhost:{port}/swagger` to use Swagger UI to test APIs.



#### DELETE - Delete an item

```c#
// DELETE: api/TodoItems/5
[HttpDelete("{id}")]
public async Task<IActionResult> DeleteTodoItem(long id)
{
  var todoItem = await _context.TodoItems.FindAsync(id);
  if (todoItem == null)
  {
    return NotFound();
  }

  _context.TodoItems.Remove(todoItem);
  await _context.SaveChangesAsync();

  return NoContent();
}
```



Use Postman to test `DELETE https://localhost:{port}/api/todoitems/1`.

No response body and HTTP Status code is `204 No Content`.

Or access `https://localhost:{port}/swagger` to use Swagger UI to test APIs.



### Prevent over-posting

Use Data Transfer Object (DTO) to [prevent over-posting](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-web-api?view=aspnetcore-6.0&tabs=visual-studio-code#prevent-over-posting).


## Depoy to OpenShift

```bash
# Create openshift namespace
oc new-project will-dotnet-demo

# Deploy application
oc new-app dotnet:6.0-ubi8~https://github.com/xdevops-caj-dotnet/TodoApi.git --name todoitems

# Check build logs
oc logs -f bc/todoitems

# Expose service
oc get svc
oc create route edge --service todoitems

# Check route of the WebApp
oc get route


```

Access `https://todoitems-will-dotnet-demo.<CLUSTER_DOMAIN>/api/todoitems` to test APIs.

References:
- <https://github.com/xdevops-caj-dotnet/myWebApp>


## Logging

If you have cluster admin right and haven't installed OpenShift Logging, follow [Install OpenShift Logging](https://docs.openshift.com/container-platform/4.9/logging/cluster-logging-deploying.html) to Install OpenShift logging:
1. Install OpenShift Elasticsearch Operator
2. Install Red Hat OpenShift Logging Operator
3. Create OpenShift Logging Instance
4. Define Kibana index patterns: 
    - Index pattern: `app-*`
    - Filter field: `@timestamp`


### Kibana Discover

Open Kibana from OpenShift shortcut, and open "Discover".

Perform API test, and search related logs:

```sql
kubernetes.namespace_name="will-dotnet-demo" AND kubernetes.container_name=todoitems AND message=Entity
```


Discover features:
- User query string to search
  - Search matched
  - Search error response or status
- Add filter fields in the search
  - Include matches
  - Add matches
- Customize search
  - New search
  - Save search
  - Open saved search
- Customize result fields (default is `time` and `Source`)
  - Add fields from left avaiable fields (e.g select `message` field)
- Customize time range (defualt is last 15 minutes)
- Autorefresh
  - Enable autorefresh logs and set autorefresh interval
  - Disable autorefresh
- Check log document (details)
  - Expand the log record to see the log fields with table and json format.
- Check log context
  - Expand the log record, and click "View surrounding documents"


References:
- [Kibana | Using Discover](https://www.elastic.co/guide/en/kibana/6.8/tutorial-sample-discover.html)
- [Kibana | Discover](https://www.elastic.co/guide/en/kibana/6.8/discover.html)
- [Apache Lucene - Query Parser Syntax](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html)
- [Kibana Query String Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/query-dsl-query-string-query.html#query-string-syntax)
- [Kibana Query Language Enahancements](https://www.elastic.co/guide/en/kibana/6.8/kuery-query.html)