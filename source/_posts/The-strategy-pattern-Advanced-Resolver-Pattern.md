---
title: The strategy pattern - Advanced (Resolver Pattern)
date: 2019-06-18 16:18:32
draft: false
tags: [ ".NET", ".NET Core", "design patterns" ]
categories: [ ".NET" ]
description: "Design Patterns"
---

## The Strategy Pattern
This pattern is a simpler design pattern - it allows you to:

1. Encapsulate a family of related algorithms
2. Allow them to vary
3. Allow a class to maintain a single purpose and implementation
4. Seperate the calculation from the delivery of its results

By doing this we can maintain standard principles, leaving the code open to extension but closed to modification.

Simple Example
```csharp
/// <summary>
    /// The Strategy abstract class, which defines an interface common to all supported strategy algorithms.
    /// </summary>
    abstract class CookStrategy
    {
        public abstract void Cook(string food);
    }

    /// <summary>
    /// A Concrete Strategy class
    /// </summary>
    class Grilling : CookStrategy
    {
        public override void Cook(string food)
        {
            Console.WriteLine("\nCooking " + food + " by grilling it.");
        }
    }

    /// <summary>
    /// A Concrete Strategy class
    /// </summary>
    class OvenBaking : CookStrategy
    {
        public override void Cook(string food)
        {
            Console.WriteLine("\nCooking " + food + " by oven baking it.");
        }
    }

    /// <summary>
    /// A Concrete Strategy class
    /// </summary>
    class DeepFrying : CookStrategy
    {
        public override void Cook(string food)
        {
            Console.WriteLine("\nCooking " + food + " by deep frying it");
        }
    }

    /// <summary>
    /// The Context class, which maintains a reference to the chosen Strategy.
    /// </summary>
    class CookingMethod
    {
        private string Food;
        private CookStrategy _cookStrategy;

        public void SetCookStrategy(CookStrategy cookStrategy)
        {
            this._cookStrategy = cookStrategy;
        }

        public void SetFood(string name)
        {
            Food = name;
        }

        public void Cook()
        {
            _cookStrategy.Cook(Food);
            Console.WriteLine();
        }
    }
```

Simple Execution
```csharp

CookingMethod cookMethod = new CookingMethod();

Console.WriteLine("What food would you like to cook?");
var food = Console.ReadLine();
cookMethod.SetFood(food);

Console.WriteLine("What cooking strategy would you like to use (1-3)?");
int input = int.Parse(Console.ReadKey().KeyChar.ToString());

switch(input)
{
    case 1:
        cookMethod.SetCookStrategy(new Grilling());
        cookMethod.Cook();
        break;

    case 2:
        cookMethod.SetCookStrategy(new OvenBaking());
        cookMethod.Cook();
        break;

    case 3:
        cookMethod.SetCookStrategy(new DeepFrying());
        cookMethod.Cook();
        break;

    default:
        Console.WriteLine("Invalid Selection!");
        break;
}
Console.ReadKey();
```

## The limmitation
For this and almost every single example I can find in existance its basically a way to register your types and apply them.
The switch is essentially moved but calling through and doing what you need to do is still in the wrong place and you need to know exaclty the method you want to use!! We need something more dynamic, where the stratergy knows what it is applied to and based on the information provided by the food in this instance, would be chosen for the execution of that dish!

## The Resolver Pattern
This is where the seperation comes in, as I cannot find an example that follows this I will coin it was the resolver pattern.
Each 'Resolver' is registered with the application for dependancy injection, like anything else and its very existance now allows it to be used and identifed by the data processing it.

Lets visit the food example again to see how this would be done in practice

We start with a service, its job? Using all available 'resolvers' it will process the data it is given (Food dishes)

```csharp
public interface IDishResolutionService
{
    Task ResolveDishesAsync(
        IEnumerable<FoodDish> recordData);
}
```

Its implementation will have one job, identify the required / available resolver and execute it.
The resolvers themselves have thier own interface to work from, one method to execute, one to identify what meal type it applies to

```csharp
public interface IFoodCookingResolver
{
    bool AppliesTo(string control);

    Task CookFoodAsync(IDbConnection conn, FoodDish dish);
}
```

The service implementaion will take all available resolvers via DI

```csharp
public DishResolutionService(
    IConnectionFactory databaseConnectionFactory,
    IEnumerable<IFoodCookingResolver> dishResolvers)
{
    _satabaseConnectionFactory = databaseConnectionFactory ??
        throw new ArgumentNullException(nameof(databaseConnectionFactory));

    _dishResolvers = dishResolvers ??
        Enumerable.Empty<IFoodCookingResolver>();
}
```

These are registered with anything else, heres the fancy way in an MVC app:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.TryAddScoped<IDishResolutionService>(ctx =>
    {
        return new DishResolutionService(ctx.GetService<IConnectionFactory>(), GetResolvers(ctx));
    });
}

private IEnumerable<IFoodCookingResolver> GetResolvers(IServiceProvider ctx)
{
    yield return new FishResolver(ctx.GetService<IConnectionFactory>());
    yield return new SingleMealResolver(ctx.GetService<IConnectionFactory>());
    yield return new FamilyBBQResolver(ctx.GetService<IConnectionFactory>());
    yield return new DefaultResolver();
}
```

Each new resolver will simply be registered in the project and this will allow it to be picked up in the service that executes the food cooking based on data within the food item itself, leaving the original code untouched and including a default resolver to go to if none apply to it specifically.

## Example Resolver



```csharp
public class FishResolver : IFoodCookingResolver
{
    private readonly IDbConnection _dbConnection;

    public FishResolver(IDbConnection conn)
    {
        _dbConnection = conn ??
            throw new ArgumentNullException(nameof(conn));
    }

    public bool AppliesTo(string control) => control == FoodType.OvernFood.UppercaseName();

    public async Task CookFoodAsync(FoodDish meal)
    {
        await _dbConnection.QueryAsync("select 'example, im not doing any data stuff here'");

        console.writeline($"Cooking that in the oven - as I oven cooks things like {meal.Name}");
    }
}
```

As with this example, all resolvers state what they apply to, so when the service seeks the required resolver, it identifies itself and has contained logic for what to do with this type of food.


```csharp
public async Task ResolveControlsAsync(
            IEnumerable<RecordData> recordData)
    {
        var groups = recordData
            .SelectMany(x => x.Controls, (x, meta) => (x.Ruid, meta))
            .GroupBy(x => x.meta.ControlType);

        var tasks = new List<Task>();
        using (var conn = _databaseConnectionFactory.GetConnection(DatabaseType.HR))
        {
            foreach (var item in groups)
            {
                tasks.Add(
                    _controlDataResolvers
                    .FirstOr(x => x.AppliesTo(item.Key), () => new DefaultResolver())
                    .CookFoodAsync(conn, item.ToList())
                );
            }

            var results = await Task.WhenAll(tasks);
            return results.SelectMany(x => x);
        }
    }
```

In the example above the service groups by the type of meal and sends them all off to be handled in the resolver togther - one at a time or as a group is fine depending on what your doing.
So the service checks the available resolvers, picks the one for the data it has, executes the resolver against the object and completes.
The core setup need no longer be changed, but the resolvers can be added to, registered and the system keeps working as it did, default resolvers are just a backup here, where say the oven is used by default unless a specific resolver says otherwise.

This allows the resolvers to differ 100% in what they do, what services they use etc. Keeping the flow of logic but seperation of type.