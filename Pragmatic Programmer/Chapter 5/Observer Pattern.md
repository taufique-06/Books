# The Observer Pattern

It is a behavioural design pattern used when you want one object (the subject) to notify multiple other objects (the observers) whenever its state changes - without the subject needing to know anything about these observers.
In simple terms

`When something changes, let other knows automatically.`

For Instance:

Think of a YouTube channel (subject) and subscribers (observers).
Whenever the channel uploads a new video, all subscribers get a notification.

## Components of the Observer Pattern

1. Subject (Publisher)
- It maintains a list of observers
- Has methods to Add/Remove observes
- Notifies when sth changes

2. Observer (Subscriber)
- Defines an interface with an Update() or similar
- Gets called by the Subject when sth happens

3. Concrete Implementations
- The actual subject and observers that implement the logic


## C# Implementation

### Define the Interfaces

IObserver – all observers will implement this:

```csharp
public interface IObserver
{
    void Update(float temperature);
}
```

ISubject – for managing subscribers:

```csharp
public interface ISubject
{
    void RegisterObserver(IObserver observer);
    void RemoveObserver(IObserver observer);
    void NotifyObservers();
}
```

### Implement the Subject

```csharp
public class WeatherStation : ISubject
{
    private List<IObserver> _observers = new();
    private float _temperature;

    public void RegisterObserver(IObserver observer)
    {
        _observers.Add(observer);
    }

    public void RemoveObserver(IObserver observer)
    {
        _observers.Remove(observer);
    }

    public void NotifyObservers()
    {
        foreach (var observer in _observers)
        {
            observer.Update(_temperature);
        }
    }

    public void SetTemperature(float temperature)
    {
        Console.WriteLine($"\n[WeatherStation] Temperature changed to {temperature}°C");
        _temperature = temperature;
        NotifyObservers();
    }
}

```

### Create Observer Classes

```csharp
public class PhoneDisplay : IObserver
{
    private readonly string _owner;

    public PhoneDisplay(string owner)
    {
        _owner = owner;
    }

    public void Update(float temperature)
    {
        Console.WriteLine($"[PhoneDisplay - {_owner}] The temperature is now {temperature}°C");
    }
}

```

However, it introduces coupling because each of the observers has to register with the observable.
In addition, because in the typical implementation the callbacks are handled inline by the observable, synchronously, it can introduce performance bottlenecks.

This is solved by the next strategy: Publish/Subscribe