# Blazor Stopwatch With Recurop-library
A stopwatch/timer component created with Blazor and Recurop library (for .NET Timer based functions).

See the live demo at https://tdmr87.github.io/BlazorStopwatchWithRecurop/

Check out the Recurop library: https://github.com/TDMR87/Recurop

![Screenshot_1](https://user-images.githubusercontent.com/44962475/124162144-db15d700-daa6-11eb-99d2-b8160b361a47.png)

***

All of the .NET System.Threading.Timer based functionality is achieved by using the Recurop .NET Standard library (available on NuGet). Recurop abstracts the functions that are executed by the Timer into an object through which the execution can be easily controlled (paused, resumed etc). Recurop also handles the thread safety of the actions, enables assigning handlerss to exceptions thrown by the actions, and offers bindable properties that expose the current state of the action (executing, paused, idle, cancelled etc.)

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
***
Inject Recurop's manager class inside a Blazor component. 
```C#
@using Recurop
@inject RecurringOperationsManager Recurop
```
***
Declare and instantiate a RecurringOperation object, which represents the timed function and it's state. You can assign the object the action to execute and event handlers for status changes and possible exceptions. In the code below, the function "IncrementTimer" is assigned as the action to execute.
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
***
The injected manager class is responsible for controlling the RecurringOperation objects. Start a recurring operation by calling StartRecurring() and specifying the operation to start and the execution interval.
```C#
void StartButton_Clicked()
{
    Recurop.StartRecurring(_timerOperation, TimeSpan.FromSeconds(1));
}
```
***
You can pause the recurring execution by simply calling the manager class's PauseRecurring() method.
```C#
void PauseButton_Clicked()
{
    Recurop.PauseRecurring(_timerOperation);
}
```
***
You can resume the paused operation by calling ResumeRecurring().
```C#
void ResumeButton_Clicked()
{
    Recurop.ResumeRecurring(_timerOperation);
}
```
***
If you want to stop or cancel a recurring operation, you can call CancelRecurring(). This will dispose the underlying timer object.
```C#
void CancelButton_Clicked()
{
    Recurop.CancelRecurring(_timerOperation);
}
```
***
The RecurringOperation object has multiple properties which can be used to poll the status of the operation.
```C#
_timerOperation.CanBeStarted;
_timerOperation.IsInitialized;
_timerOperation.IsRecurring;
_timerOperation.IsNotRecurring;
_timerOperation.IsIdle;
_timerOperation.IsPaused;
_timerOperation.LastRunStart;
_timerOperation.LastRunFinish;
_timerOperation.Status;
_timerOperation.IsExecuting;
```
***
All of the properties implement INotifyPropertyChanged, in order to easily bind them to UI elements. The UI elements will therebay react immediately to state changes of the operation. For example, you can disable/enable buttons according to the operation's state.
```C#
<button class="btn btn-sm btn-primary" disabled=@(_timerOperation.CanBeStarted ? false : true) @onclick="StartTimer">Start</button>
<button class="btn btn-sm btn-primary" disabled=@(_timerOperation.IsRecurring ? false : true)>Pause</button>
<button class="btn btn-sm btn-primary" disabled=@(_timerOperation.IsPaused ? false : true)>Resume</button>
<button class="btn btn-sm btn-danger" disabled=@(_timerOperation.IsRecurring ? false : true)>Stop</button>
<button class="btn btn-sm btn-primary" disabled=@(_timerOperation.IsRecurring ? false : true)
``` 
