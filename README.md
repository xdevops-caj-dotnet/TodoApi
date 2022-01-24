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


# Set resources requests and limits
oc set resources deployment todoitems -n will-dotnet-demo \
  --limits=cpu=200m,memory=100Mi \
  --requests=cpu=100m,memory=100Mi
```

Access `https://todoitems-will-dotnet-demo.<CLUSTER_DOMAIN>/api/todoitems` to test APIs.

References:
- <https://github.com/xdevops-caj-dotnet/myWebApp>


## Logging

### Use Serilog in .NET 6

Use [Serilog](https://github.com/serilog/serilog) in .NET 6 for application logging for structured log data.

Add Serilog packages:

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
```

A `Program.cs` [example](https://github.com/datalust/dotnet6-serilog-example/blob/dev/Program.cs)

A `appsettings.json` [example](https://github.com/datalust/dotnet6-serilog-example/blob/dev/appsettings.json)

In `TodoItemsControllers.cs`, use Serilog for API logging.

```c#
using Serilog;

// other codes...
Log.Information("Retrive all todo items...");
Log.Information("Retrieved an item: {@item}", todoItem);
Log.Information("Update an item {@item} by {id}", todoItem, id);
// other logging...

```


References:

- [Serilog](https://github.com/serilog/serilog)
- [Serilog | Getting Started](https://github.com/serilog/serilog/wiki/Getting-Started)
- [Serilog | Structured Data](https://github.com/serilog/serilog/wiki/Structured-Data)
- [net 6.0 Serilog example](https://github.com/datalust/dotnet6-serilog-example)
- [Setting up Serilog in .NET 6](https://blog.datalust.co/using-serilog-in-net-6/)

### Install OpenShift Logging
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
  - Expand the log record, and click "View surrounding documents" (e.g. try to add duplciate items)


References:
- [Kibana | Using Discover](https://www.elastic.co/guide/en/kibana/6.8/tutorial-sample-discover.html)
- [Kibana | Discover](https://www.elastic.co/guide/en/kibana/6.8/discover.html)
- [Apache Lucene - Query Parser Syntax](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html)
- [Kibana Query String Syntax](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/query-dsl-query-string-query.html#query-string-syntax)
- [Kibana Query Language Enahancements](https://www.elastic.co/guide/en/kibana/6.8/kuery-query.html)


Besides search matched logs, you also can use Kibana Dashboard to analyze logs visually.

References:
- [Kibana Visualize](https://www.elastic.co/guide/en/kibana/6.8/visualize.html)
- [Kibana Dashboard](https://www.elastic.co/guide/en/kibana/6.8/dashboard.html)
- [THE TOP 24 KIBANA DASHBOARDS & VISUALISATIONS](https://logit.io/blog/post/the-top-kibana-dashboards-and-visualisations)





## Monitoring



References:

- [Grafana Operator](https://github.com/grafana-operator/grafana-operator)

- [Grafana Operator | Grafana Instance](https://github.com/grafana-operator/grafana-operator/blob/master/documentation/deploy_grafana.md)

- [Custom Grafana dashboards for Red Hat OpenShift Container Platform 4](https://www.redhat.com/en/blog/custom-grafana-dashboards-red-hat-openshift-container-platform-4)

- [Grafana Helm charts example](https://github.com/petbattle/pet-battle-infra/tree/main/grafana)

- [Build efficient Grafana dashboards from the built-in Prometheus of OpenShift Container Platform 4.3](https://developer.ibm.com/tutorials/custom-grafana-dashboards-from-pre-configured-prometheus-ocp-43-datasource/)

  



### Load Testing

Set API root URI:

```bash
export API_ROOT_URI="https://$(oc get route/todoitems -n will-dotnet-demo --template='{{.spec.host}}')"
```



Add some items:

```bash
# Add some items
curl -k -X POST -H "Content-Type: application/json" -d '{"id": 1,"name": "one","isComplete": true}' $API_ROOT_URI/api/todoitems
curl -k -X POST -H "Content-Type: application/json" -d '{"id": 2,"name": "two","isComplete": true}' $API_ROOT_URI/api/todoitems
curl -k -X POST -H "Content-Type: application/json" -d '{"id": 3,"name": "three","isComplete": true}' $API_ROOT_URI/api/todoitems
curl -k -X POST -H "Content-Type: application/json" -d '{"id": 4,"name": "four","isComplete": true}' $API_ROOT_URI/api/todoitems
curl -k -X POST -H "Content-Type: application/json" -d '{"id": 5,"name": "five","isComplete": true}' $API_ROOT_URI/api/todoitems

# Verify added items
curl -sk $API_ROOT_URI/api/todoitems | jq
```



Use [hey](https://github.com/rakyll/hey) to execute a simple load testing.

```bash
hey -t 30 -c 100 -n 100000 \
  -H "Content-Type: application/json" \
  -m GET $API_ROOT_URI/api/todoitems 

```



In a new terminal, use [stern](https://github.com/wercker/stern) to run `stern todoitems` to watch logs, or run `oc logs -f <pod>` to watch logs.



### Observe

**Observe OCP platform metrics:**

Refer to [Reviewing monitoring dashboards](https://docs.openshift.com/container-platform/4.9/monitoring/reviewing-monitoring-dashboards.html) , Switch to Admin view, and open "Observe / Dashboard" to monitor OCP platform metrics.



**Observe application metrics:**

On OpenShift, choose `will-botnet-demo` project, switch to Developer view, and click "Observe" to open built-in monitoring dashboard and inspect metrics.



You also can open `todoitems` deployment, and open its pods to check Pod metrics.



### Access Grafana dashboard

You can access OpenShift built-in Grafana dashboard with `cluster-admin` rights.

```bash
oc get routes -n openshift-monitoring
```



Access Grafana dashboard via browser.



Open Dashboard / Manage, and open Default / "Kubernetes / Compute Resources / Namespaces (Pod)" dashboard.

Choose namespace as `will-dotnet-demo`.



References:

- [Accessing third-party UIs](https://docs.openshift.com/container-platform/4.9/monitoring/accessing-third-party-uis.html)



### Work with Custom dashboard



#### Install Grafana operator

Create a OpenShift namespace (e.g. `will-grafana`), and install on this namespace.



Create a new OpenShift namespace:

```bash
oc new-project will-grafana
```



On Operator Hub, search "grafana" and choose "Grafana Operator" which is community operator provided by OpenShift.

Choose "Installed Namespace" as new created OpenShift namespace.

Click "Install" button to proceed installation.



#### Create Grafana instance

On OpenShift, choose `will-grafana` namespace, and open installed "Grafana Operator".

On the operator details page, open "Grafana" tab, and then click "Create Grafana" button.

Input an Grafana instance name (e.g. `grafana`) to create a new Grafana instance.



Grafana instancy YAML example:

```yaml
apiVersion: integreatly.org/v1alpha1
kind: Grafana
metadata:
  name: grafana
  namespace: will-grafana
spec:
  config:
    auth:
      disable_login_form: false
  ingress:
    enabled: true
  dashboardLabelSelector:
    - matchExpressions:
        - key: monitoring-key
          operator: In
          values:
            - grafana

```



Default user is `admin` , the admin password as below:

```bash
oc get secret grafana-admin-credentials -o=jsonpath='{.data.GF_SECURITY_ADMIN_PASSWORD}' \
| base64 -d
```



After creating Grafana instance, it will create a service account `grafana-serviceaccount`

```bash
oc get sa -n will-grafana
```



Verify Grafana route:

```bash
oc get route -n will-grafana
```



#### Connect Prometheus to custom Grafana

Grant Grafana service account `cluster-monitoring-view` cluster role:

```bash
oc adm policy add-cluster-role-to-user cluster-monitoring-view -z grafana-serviceaccount -n will-grafana
```



Get access token to connect Prometheus:

```bash
oc serviceaccounts get-token grafana-serviceaccount -n will-grafana
```



Create GrafanaDataSource instance via YAML:

```yaml
apiVersion: integreatly.org/v1alpha1
kind: GrafanaDataSource
metadata:
  name: prometheus-grafanadatasource
  namespace: will-grafana
spec:
  datasources:
    - access: proxy
      editable: true
      isDefault: true
      jsonData:
        httpHeaderName1: 'Authorization'
        timeInterval: 5s
        tlsSkipVerify: true
      name: Prometheus
      secureJsonData:
        httpHeaderValue1: 'Bearer ${BEARER_TOKEN}'
      type: prometheus
      url: 'https://thanos-querier.openshift-monitoring.svc.cluster.local:9091'
  name: prometheus-grafanadatasource.yaml
```



Notes:

- Replace `${BEARER_TOKEN}` as bove access token.
- Replace `will-grafana` as your namespace.





#### Import custom Grafana dashboard



Open the OpenShift built-in Grafana, and open Default / "Kubernetes / Compute Resources / Namespace (Pods)", click "Share" button, and click "Export", and save the dashboard json as file or view json / copy to clipboard. Or try to find a Grafana dashboard [here](https://grafana.com/grafana/dashboards/).



Open the new custom Grafana, and open Dashboard / Manage, and then click "Import" button to import the dashboard json file or paste json via clipboard, and then save the board.



On the new custom Grafana, open the new imported dashboard and choose `will-botnet-demo` to view the dashboard.



On OpenShift, open installed Grafana operator, and open GrafanaDashboard and create a new instance, input a dashboard name, folder name and paste the daboard json or Grafana dashboard url.



```yaml
apiVersion: integreatly.org/v1alpha1
kind: GrafanaDashboard
metadata:
  name: k8s-pods-dashboard
  namespace: will-grafana
  labels:
    monitoring-key: grafana
spec:
  customFolderName: custom
  json: >-
    ... # Grafana dashboard json
```



Notes:

- The `monitoring-key: grafana` label should align with `dashboardLabelSelector` in Grafana instance



### Alerts

References:

- [Enabling monitoring for user-defined projects](https://docs.openshift.com/container-platform/4.9/monitoring/enabling-monitoring-for-user-defined-projects.html#enabling-monitoring-for-user-defined-projects)
- [Creating cluster monitoring configmap](https://docs.openshift.com/container-platform/4.9/monitoring/configuring-the-monitoring-stack.html#creating-cluster-monitoring-configmap_configuring-the-monitoring-stack)
- [Creating altering rules for user defined projects](https://docs.openshift.com/container-platform/4.9/monitoring/managing-alerts.html#creating-alerting-rules-for-user-defined-projects_managing-alerts)
- [kubernetes-mixin](https://github.com/kubernetes-monitoring/kubernetes-mixin)
- [Prometheus Alert rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
- [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator/blob/main/README.md)
- [Effective Kubernetes monitoring using Prometheus](https://nuvalence.io/blog/effective-kubernetes-monitoring-using-prometheus)

#### Enabling monitoring for user-defined projects

```bash
oc -n openshift-monitoring edit configmap cluster-monitoring-config
```

Modify `enableUserWorkload` as `true`.

If the Configmap doesn't exist, create it:

```yaml

cat << EOF > /tmp/cluster-monitoring-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |
    enableUserWorkload: true 
EOF

oc apply -f /tmp/cluster-monitoring-config.yaml -n openshift-monitoring
```



Verify to have pods running and ready on `openshift-user-workload-monitoring`:

```bash
oc -n openshift-user-workload-monitoring get pod
```



#### Create Promethues Rules for user defined project



```bash
export NS=will-dotnet-demo

cat << EOF > /tmp/prometheusrule.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: todoitems-alerts
  namespace: ${NS}
spec:
  groups:
  - name: todoitems.rules
    rules:
    - alert: TodoItemsNotAvailable
      annotations:
        message: 'TodoItems in namespace ${NS} is not available for the last 1 minutes.'
      expr: (1 - absent(kube_pod_status_ready{condition="true",namespace="${NS}"} * on(pod) group_left(label_deployment) kube_pod_labels{label_deployment="todoitems",namespace="${NS}"})) == 0
      for: 1m
      labels:
        severity: critical
    - alert: TodoItemsMaxHttpRequestTime
      annotations:
        message: 'TodoItems max http request time over last 5 min in namespace ${NS} exceeds 1.5 sec.'
      expr: max_over_time(http_server_requests_seconds_max{service="todoitems",namespace="${NS}"}[5m]) > 1.5
      labels:
        severity: warning
EOF
```



Create prometheus rules:

```bash
oc apply -f /tmp/prometheusrule.yaml -n ${NS}
```



Verify prometheus:

```bash
oc get prometheusrules todoitems-alerts -n ${NS} -o yaml
```



Verify user defined alert rules on OCP:

- Switch to Admin view, open Observe / Alerting, and open alert rules, filter source by "User" rules.
- Switch to Developer view, open Observe / Alerts to see alert rules, and click view alert rules to see rule details.



Switch to developer view, and open Observe / Alerts to check firing alerts.

- Try to set the `todoitems` deployment pod count as 0 to see if has firing alerts.
- Try to resume `todoitems` deployment pod count as 1 to see if the firing alerts disappear.



Alert states:

- Pending -> Firing -> Disappear firing when fixed / Slience the firing temporally



*Tips: User defined prometheus rules can refer to platform prometheus rules.*



### Collect application metrics



TODO: Use ServiceMonitor object to scrap application exposed metrics.





