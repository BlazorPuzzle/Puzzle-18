# Blazor Puzzle #18

## Where's the Config?

YouTube Video: https://youtu.be/QdPp_HClHn4

BlazorPuzzle Home Page: https://blazorpuzzle.com

#### The Challenge:

This is a Blazor Web App running on .NET 8 with the Interactive render mode set to "WebAssembly" and the Interactivity location set to "Global". 

A WelcomeMessage value is set in appsettings.json on the server. How can we simply show it in a Razor page?

#### The Solution:

The problem is that we need to show something on the client that isn't on the client at all. It's on the server.

On the server, we're implementing a minimal API Endpoint in the server's *Program.cs*.

Add this on before the line `app.Run()`:

```c#
app.MapGet("/config", () => builder.Configuration["WelcomeMessage"]);
```

This creates a public endpoint called "config" that returns the welcome message right from the config file.

Now on the client, we first need to inject an HttpClient with the base address set to our server project.

In the client's *Program.cs*, add the following on line 5:

```c#
builder.Services.AddScoped(sp => new HttpClient
    { BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) });
```

Now we can change *Home.razor* to load and display the message:

*Home.razor*:

```c#
@page "/"
@inject HttpClient httpClient

<PageTitle>Home</PageTitle>

<h1>@welcomeMessage</h1>

<p>
    This is a Blazor Web App running on .NET 8 with the Interactive render mode set to "WebAssembly" and the Interactivity location set to "Global".
</p>

<p>
    A WelcomeMessage value is set in appsettings.json on the server.
</p>

<p>
    How can we simply show it here in a Razor page?
</p>


@code {

    string welcomeMessage = string.Empty;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            welcomeMessage = await httpClient.GetStringAsync("config");
            StateHasChanged();
        }
    }
}
```

If you run it now you will get an error because the HttpClient is not configured on the server. Remember that the app runs server-side first during pre-rendering.

To fix this, add the following to the server's *Program.cs* file just before the line `var app = builder.Build();`:

```c#
builder.Services.AddScoped<HttpClient>();
```

The base address is irrelevant because it will never be called from the server.

Boom!

