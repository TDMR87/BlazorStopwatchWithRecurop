# Blazor Stopwatch With Recurop-library
A stopwatch/timer component created with Blazor and Recurop library (for .NET Timer based functions).

See the live demo at https://tdmr87.github.io/BlazorStopwatchWithRecurop/

Check out the Recurop library: https://github.com/TDMR87/Recurop

![Screenshot_1](https://user-images.githubusercontent.com/44962475/124162144-db15d700-daa6-11eb-99d2-b8160b361a47.png)

***

All the .NET System.Threading.Timer based functionality is achieved by using the Recurop .NET Standard library (available on NuGet).

***

Register Recurop in the DI container.

```C#
using Recurop;

public class Program
{
    public static async Task Main(string[] args)
    {
        var builder = WebAssemblyHostBuilder.CreateDefault(args);
        builder.RootComponents.Add<App>("#app");

        // Registers Recurop's manager class as a singleton to the DI container.
        builder.Services.AddRecurop();

        await builder.Build().RunAsync();
    }
}
```

Inject Recurop's manager class inside a Blazor component. 
```C#
@using Recurop
@inject RecurringOperationsManager Recurop
```

Recurop abstracts a function that executes in certain intervals into a single object (RecurringOperation type). Declare and instantiate a RecurringOperation object, which represents the timed function and it's state. You can assign the object the action to execute and event handlers for status changes and possible exceptions. Properties of the RecurringOperation object implement INotifyPropertyChanged, so you can easily bind the properties into your UI elements.
```C#
@code {
    RecurringOperation _timerOperation;

    protected override void OnInitialized()
    {
        _timerOperation = new("timer");
        _timerOperation.Operation = IncrementTimer;
        _timerOperation.StatusChanged += TimerOperationStatusChanged;
        _timerOperation.OperationFaulted += LogError;
    }
}
```
