
# Install from NuGet
-Microsoft.EntityFrameworkCore
-Microsoft.EntityFrameworkCore.SqlServer

## Model Binding
If you want to read the data from JSON/Text you need to add [from body] without it will not read the product.
Since the the method name is post it can be called from URL: /api/product with type POST (Help of POST man for test)
```sh
    [HttpPost("")]
    public IActionResult Post(Product product)
    {
        return Ok();
    }
```
should be
```sh
	[HttpPost("")]
    public IActionResult Post([FromBody]Product product)
    {
        return Ok();
    }
```

## Middlewares
in startup we used a middleware, they are injected with app.
```sh
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {            
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        // Routing mechanism as Default 
        app.UseMvc(config => 
		{ 
			//mvc middleware
			config.MapRoute("DefaultRoute", "api/{controller}/{action}"); 
		
		});         
    }
```

> Control if the person who request has proper authentication or not.

### Custom Middleware
We are creating a custom middleware to check the authentication
for that we need to rules first Invoke and

we need http context to read the header data 
Request delegate will provide to move between middleware

```sh
public class AuthenticationMiddleware
{
    private readonly RequestDelegate _next;

    public AuthenticationMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    public async Task Invoke(HttpContext context)
    {
        var authHeader = context.Request.Headers["Authorization"];

        // Pass to next middleware
        await _next(context);
    }
}

```
If you send a request header via postman and debug on the await _next you can see the middle ware first read the header
Donot forget to add the middleware in the startup before the test
```sh
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
            
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    // authentication middleware
    app.UseMiddleware<AuthenticationMiddleware>();

    // Routing mechanism as Default
    app.UseMvc(config =>
    {
        // Routing middleware
        config.MapRoute("DefaultRoute", "api/{controller}/{action}");
    });
    app.UseMvc();
}
```

