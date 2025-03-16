# Why Generic Inputs and Specific Outputs Create Better Code

## Arguments for Generic Types and Greater Flexibility

-   A method that performs a service can work with any vehicle. To ensure flexibility, we accept the most generic type that makes sense:
```csharp
public static void Service(Vehicle vehicle)
{
    // Service implementation
}
```
-   This makes sense because the service is not limited to a specific type of vehicle. By accepting a generic type (`Vehicle`), we allow the method to work with all subtypes: cars, trucks, or any future types of vehicles we might add.
-   However, for specific methods, such as `CheckBrakes`, which applies only to cars, we use the specific type `Car`:
```csharp
public static void CheckBrakes(Car car)
{
    // Logic for checking brakes
}
```
-   Here, we use the specific type `Car` because this functionality does not apply to trucks or other vehicles.

## Return Values with Specific Types for Greater Clarity

-   A method that returns a vehicle ready for service:
```csharp
public static Vehicle GetVehicleForService()
{
    // Returns any vehicle
    return new Car();
}
```
-   While this is valid, a generic return type (`Vehicle`) can be limiting. If we know the method always returns a car, we should use a more specific type:
```csharp
public static Car GetCarForService()
{
    // Returns a specific vehicle
    return new Car();
}
```
-   The advantage of a specific return type is that method users can immediately access functions specific to that type without casting. For example:
```csharp
Car car = GetCarForService();
car.StartEngine();
```
-   If we use the generic type `Vehicle`, users would have to cast the object manually:
```csharp
Vehicle vehicle = GetVehicleForService();
(vehicle as Car)?.StartEngine();
```

## “Possible” and “Honest” Methods

-   It is important to understand what “most generic” or “most specific” means. It does not always mean choosing `object` or the lowest common class. For example:
```csharp
public static void Process(object obj)
{
    if (obj is Vehicle vehicle)
    {
        // Process vehicle
    }
}
```
-   Although this technically fulfills the rule of genericity, this method represents leaky abstraction. The need to understand the method’s inner logic makes it less useful. Instead, the method should be honest:
```csharp
public static void Process(Vehicle vehicle)
{
    // Process vehicle
}
```
-   Honesty in a method means that its signature clearly communicates what it does and what it works with, without hiding implementation details.

## Broader and Deeper Implications

-   The rule “generic for arguments, specific for returns” reminds us of deeper principles:
    -   **Flexibility in input:** The more open a method is to various types, the more useful it is across a broader spectrum of cases.
    -   **Clarity in output:** The more precise the return value, the easier it is for users to work with it.

## For the End

-   The rule of generic arguments and specific return values helps us design methods that are both adaptable and clear. By practicing this rule, we ensure that our code remains understandable, scalable, and usable.
-   Just as every car is designed for a specific purpose, every method should be designed with clarity and intent in mind.