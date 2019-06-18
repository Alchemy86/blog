---
title: 'Function Programming in C#'
date: 2019-06-18 14:26:09
draft: false
tags: [ ".NET", ".NET Core", "functional" ]
categories: [ ".NET" ]
images: []
banner: ""
description: ""
---

## Functional Programming
Functional programming is a style that treats computation as the evaluation of
mathematical functions and avoids changing-state and mutable data.

## Immutable Types
This is an object whos state cannot be modified after it is created.

Mutable
```csharp
public class Rectangle
{
	 public int Length {get;set;}
	 public int Height {get;set;}

	 public void Grow(int length, int height)
	 {
	 Length += length;
	 Height += height;
	 }
}
Rectangle r = new Rectangle();
r.Length = 5;
r.Height = 10;
r.Grow(10, 10);
// r.Length is 15, r.Height is 20, same
instance of r
```

Immutable
```csharp
public class ImmutableRectangle
{
	 int Length { get; }
	 int Height { get; }
	
	 public ImmutableRectangle(int length,
	 int height)
	 {
		 Length = length;
		 Height = height;
	 }
	 public ImmutableRectangle Grow(int length, int height) => new ImmutableRectangle(Length + length, Height + height);
}
ImmutableRectangle r = new
ImmutableRectangle(5, 10);

r = r.Grow(10, 10);
// r.Length is 15, r.Height is 20, is a new
instance of r
```

## Expressions vs Statements
Statments define an actiona nd are executed for thier side-effect.
Expressions produce a result without mutating state.

```csharp
Statement
public static string GetSalutation(int hour) {
 string salutation; // placeholder value
 if (hour < 12)
 salutation = "Good Morning";
 else
 salutation = "Good Afternoon";
 return salutation; // return mutated variable
}
Expression
public static string GetSalutation(int hour) =>
 hour < 12 ? "Good Morning" : "Good Afternoon";
```

## [Value Tuples](https://blogs.msdn.microsoft.com/mazhou/2017/05/26/c-7-series-part-1-value-tuples/)
Tuple is a more efficient and more productive lightweight syntax to define a data structure that carries more than one value. 
**Requires NuGet Package System.ValueTuple**

* Represent data without DTO classes
* Lower memory footprint than a class
* Return multiple values from methods without the need for out variables

```csharp
(double lat, double lng) GetCoordinates(string
query)
{
	//DO search query ...
	return (lat: 47.6450905056185,
	lng: 122.130835641356);
}
var pos = GetCoordinates("15700 NE 39th St,
Redmond, WA");
pos.lat; //47.6450905056185
pos.lng; //122.130835641356
```

## Func Delegates

Func Delegates encapsulate a method. When declaring a Func, input and output parameters are specified as T1-T16, and TResult.

* Func<TResult> – matches a method that takes no arguments, and returns value of type TResult.
* Func<T, TResult> – matches a method that takes an argument of type T, and returns value of type TResult.
* Func<T1, T2, TResult> – matches a method that takes arguments of type T1 and T2, and returns value of type TResult.
* Func<T1, T2, …, TResult> – and so on up to 16 arguments, and returns value of type TResult.

```csharp
Func<int, int> addOne = n => n +1;
Func<int, int, int> addNums = (x,y) => x + y;
Func<int, bool> isZero = n => n == 0;

Console.WriteLine(addOne(5)); // 6
Console.WriteLine(isZero(addNums(-5,5))); //
True

int[] a = {0,1,0,3,4,0};
Console.WriteLine(a.Count(isZero)); // 3
```

## Higher Order Functions / Functions as Data

Higher-order function is a function taking one or more function parameters as input, or returning a function as output. The other functions are called first-order functions. (Again, in C#, the term function and the term method are identical.) C# supports higher-order function from the beginning, since a C# function can use almost anything as its input/output, except:

Static types, like System.Convert, System.Math, etc., because there cannot be a value (instance) of a static type.
Special types in .NET framework, like System.Void.
A first-order function can take some data value as input and output:

```csharp
method signature
int IEnumerable.Count<T>(Func<T, Bool>
predicate)
Source code for Count()
int count = 0;
 foreach (TSource element in source)
 {
 checked // overflow exception check
 {
 if (predicate(element)) //
func<T,Bool> invoked
 {
 count++;
 }
 }
 }
return count;
usage
bool[] bools = { false, true, false, false };

int f = bools.Count(bln => bln == false); //
out = 3
int t = bools.Count(bln => bln == true); // out
= 1
```

## Method Chaining (~Pipelines)

Since C# lacks a Pipeline syntax, pipelines in C# are created with design patterns that allow for methods to chain. 
The result of the method chain should produce the desired value and type.

```csharp
string str = new StringBuilder()
 .Append("Hello ")
 .Append("World ")
 .ToString()
 .TrimEnd()
 .ToUpper();
```

## Extension Methods

Extension methods are a great way to extend method chains and add functionality to a class.

```csharp
// Extends the StringBuilder class to accept a predicate
public static StringBuilder AppendWhen( this StringBuilder sb, string value, bool predicate) =>
 predicate ? sb.Append(value) : sb;

string htmlButton = new StringBuilder()
 .Append("<button")
 .AppendWhen(" disabled", isDisabled)
 .Append(">Click me</button>")
 .ToString();
```

## Yield

Using yield to define an iterator removes the need for an explicit extra class (the class that holds the state for an enumeration.
You consume an iterator method by using a foreach statement or LINQ query.
Yield is the basis for many LINQ methods.

```csharp
// Without Yield
public static IEnumerable<int>
GreaterThan(int[] arr, int gt) {
 List<int> temp = new List<int>();
 foreach (int n in arr) {
 if (n > gt) temp.Add(n);
}
 return temp;
}
// With Yield
public static IEnumerable<int>
GreaterThan(int[] arr, int gt) {
 foreach (int n in arr) {
 if (n > gt) yield return n;
 }
}
```

## LINQ

The gateway to functional programming in C#. LINQ makes short work of most imperative programming routines that work on arrays and collections

