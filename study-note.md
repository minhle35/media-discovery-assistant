## Check current installation of dotnet
```bash
dotnet --version
```

## Web API project

### Create a server
```bash
dotnet new webapi -n [name-server] --use-controllers
```

**Options:**
- `--use-controllers` - Create new ASP.NET Core Web API project with controllers

This command will initialize this directory structure:
```
MyApi/
├── Controllers/
│   └── WeatherForecastController.cs
├── obj/
│   ├── *.json
│   ├── *.props
│   ├── *.target
│   └── *.cache
├── Properties/
│   └── launchSettings.json
├── appsettings.Development.json
├── appsettings.json
├── MyApi.csproj
├── MyApi.http
├── Program.cs
└── WeatherForecast.cs
```

## How Routing Works

### 1. Program.cs - The Entry Point
```csharp
builder.Services.AddControllers();  // Register controller services
app.MapControllers();               // Map all controller routes
```

**FastAPI equivalent:**
```python
app = FastAPI()
app.include_router(router)
```

### 2. Controller - Where Routes Live
```csharp
[ApiController]                    // Marks this as an API controller
[Route("[controller]")]            // Base route: /WeatherForecast
public class WeatherForecastController : ControllerBase
{
    [HttpGet]                      // GET /WeatherForecast
    public IEnumerable<WeatherForecast> Get() { ... }
}
```

**FastAPI equivalent:**
```python
@app.get("/weatherforecast")
def get(): ...
```

### 3. Routing Patterns Comparison

| FastAPI | ASP.NET Core |
|---------|--------------|
| `@app.get("/")` | `[HttpGet]` |
| `@app.post("/")` | `[HttpPost]` |
| `@app.get("/{id}")` | `[HttpGet("{id}")]` |
| `@app.put("/{id}")` | `[HttpPut("{id}")]` |
| `@app.delete("/{id}")` | `[HttpDelete("{id}")]` |

---

## Request/Response Flow

```
┌─────────┐         GET /WeatherForecast         ┌─────────────┐
│  curl   │  ─────────────────────────────────►  │  ASP.NET    │
│ (client)│                                      │  Core API   │
│         │  ◄─────────────────────────────────  │  (server)   │
└─────────┘         JSON Response                └─────────────┘
```

**The flow:**
1. `curl` sends HTTP GET request to `/WeatherForecast`
2. `MapControllers()` routes it to `WeatherForecastController`
3. `[HttpGet]` matches the GET method → calls `Get()`
4. Controller returns C# objects → automatically serialized to JSON

**This is the same pattern as FastAPI:**
```python
@app.get("/weatherforecast")
def get():
    return [{"date": "...", "temperatureC": 25}]  # Auto-serialized to JSON
```
