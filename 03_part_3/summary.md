# C# Advanced Topics

In the 3rd part of the c# series some more advanced topics are covered.

# Generics

In older versions of c# it was for example not possible to create a list of a generic type. As a workaround a 'ObjectList' could be used, but this comes with a performance penalty. C# now lets us use generics instead. lets say we created a class called GenericList.

```c#
public class GenericList<T>
{
    public void Add(T value) {}
    public T this[int index] {}
}
```

In the main program we can create different lists and specify the type of class we store when we create the list. 

```c#
namespace Generics
{
    class Program
    {
        static void Main(string[] args)
        {
            var numbers = new GenericList<int>();
            var books = new GenericList<Book>();
        }
    }
}
```

Usually we don't create generic's ourselves since the most useful ones are already implemented. System.Collections.Generic contains a lot of useful classes. 

```c#
Systems.Collections.Generic
```

## Creating Generics

For example to create a dictionary:

```c#
public class GenericDictionary<TKey, TValue>
{
    public void Add(TKey key, TValue value) {}
}
```

## Constraints

Without constraints generics can't always work. For example a generic is normally considered an object. For example a Max function for integers cannot be put into a generic class without constraining the object to be comparable. 

```c#
public class Utilities<T>
{
    public int Max(int a, int b) // this wont work!
    {
        return a > b ? a : b;
    }
}
```

There are multiple constraints:

- IComparable - a type that can be compared
- struct - a value type
- class - a reference type
- Product - a certain class and it's subclasses
- new() - an object with a constructor, for example if you need to be able to instantiate an object in the function.

Using the constraint IComparable allows us to write a generic max function.

```c#
public class Utilities<T> where T : IComparable
{
    public T Max(T a, T b)
    {
        return a.CompareTo(b) > 0 ? a : b;
    }
}
```

# Delegates

A delegate is a reference (pointer) to a function. This can be used to make frameworks more extensible and flexible. For example when delegates are used the library can be extended by the end user without needing to change or recompile the original library. In the course a class with photo filters is used as an example. Delegates are defined like this:

**PhotoProcessor class** responsible for applying filters

```c#
public class PhotoProcessor
{
    // declaration of the delegate
    public delegate void PhotoFilterHandler(Photo photo);
    
    public void Process(string path, PhotoFilterHandler filterHandler)
    {
        var photo = Photo.load(path);
        filterHandler(photo) // this process method does not know what filter will be applied
    }
}
```

The currently available filters are defined in a seperate class **PhotoFilters**. In the main program the delegate will be used like this:

1. Create an instance of the PhotoProcessor class
2. Create an instance of the current PhotoFilters class
3. Assign filters to the filterHandler delegate
4. Apply processor.Process();

Adding other filters can be achieved using **+=** operator on the filterHandler delegate. And new filters can be created and added on the spot without adjusting anything in the filter library, as long as it confirms with the signature of the delegate. For example adding a custom filter to remove red eyes could be added.

```c#
class Program
{
    static void Main(string[] args)
    {
        var processor = new PhotoProcessor();
        var filters = new PhotoFilters();
        PhotoProcessor.PhotoFilterHandler filterHandler = filters.ApplyBrightness;
        filterHandler += filters.ApplyContrast;
        filterHandler += filters.ApplyRemoveRedEye; // non library filter
        
        processor.Process("photo.jpg", filterHandler)
    }
    
    static void ApplyRemoveRedEye(Photo photo) {}
}
```

## Built in delegates

Instead of declaring a delegate function like in the previous example it is also possible to use the built in delegate methods. **Action** and **Func**. Action is used for a delegate that returns void, func can be used to return a value.

```c#
public class PhotoProcessor
{  
    public void Process(string path, Action<Photo> filterHandler)
    {
        var photo = Photo.load(path);
        filterHandler(photo) // this process method does not know what filter will be applied
    }
}
```

```c#
class Program
{
    static void Main(string[] args)
    {
        var processor = new PhotoProcessor();
        var filters = new PhotoFilters();
        Action<Photo> filterHandler = filters.ApplyBrightness;
        filterHandler += filters.ApplyContrast;
        filterHandler += filters.ApplyRemoveRedEye; // non library filter
        
        processor.Process("photo.jpg", filterHandler)
    }
    
    static void ApplyRemoveRedEye(Photo photo) {}
}
```

## Delegates vs Interfaces

Use a delegate when (according the msdn guidelines):

- An event pattern is used (the built in event is also delegate under the hood)
- The caller does not need access to other properties or methods on the object implementing the method.

# Lamdba Expressions

A Lambda expression is an anonymous method. No acces modifier, no name, no return statement. The syntax is very similar to JavaScript. Example Code:

```c#
class Program {
    static void Main(string[] args) 
    {
        Func<int, int> Square = number => number * number; // the func way
        var result = Square(5);
    }
    
    static int Square(int number) // the old way
    {
        return number*number;
    }  
}
```

But a more useful case is when a **predicate** function is expected. For example the functions provided for collections.

```c#
class Program {
    static void Main(string[] args) 
    {
        var books = new BookRepository().GetBooks();
        var cheapBooks = books.findAll(Under10Dollars); // calling a function for the predicate
        var expensiveBooks = Books.findAll(b => b.Price > 10); // lambda expression
    }
    
    static bool Under10Dollars(Book book)
    {
        return book.Price < 10;
    }  
}
```

# Events

Events are used so that a class can send a signal that a certain event has happened. In another part of the program Listeners can be added to this event. This allows for example a mail service to send an email when an order is complete. Under the hood this is all implemented as a delegate function.

Steps:

1. declare an event using `public event Eventhandler<>`
2. Create a `protected virtual void OnEvent(Class obj)`
3. Raise the event by calling the `OnEvent()` in a class method
4. Create an event handler in a seperate class
5. Add the event handler to the list of pointers that will be excetuted

```c#
public class VideoEncoder
{
    // Step 1
    public event EventHandler VideoEncoding // if no aditional arguments are needed
    public event EventHandler<VideoEventArgs> VideoEncoded; // for sending aditional arguments
    
    // Step 2
    protected virtual void OnVideoEncoding()
    {
        if ( VideoEncoding != null ) // make sure there are listeners added in step 5!
            VideoEncoding(this, EventArgs.Empty)
    }
    
    protected virtual void OnVideoEncoded(Video video) {
        if (VideoEncoded != null ) 
            VideoEncoded(this, new VideoEventArgs(){Video video}); // fire the event
    }
    
    // Step 3. Note that usually for readability the methods from step 2 are better
    // placed below this method.
    public void Encode(Video video) {
        OnVideoEncoding();
        // encode the video
        OnVideoEncoded(video); 
    }
    

}
```

Another class can implement a function to match this. For example a message service.

```c#
public class MessageService 
{    
    // step 4. EventHandler with matching signature
	public void OnVideoEncoded(object source, EventArgs e)
    {
        // ...
    }
}
```

In the main program we tie these together:

```c#
static void Main(string[] args) 
{
    var video = new Video() { Title = "Video 1" };
    var videoEncoder = new VideoEncoder(); // publisher
    var messageService = new MessageService(); // subscriber
    // Step 5
    videoEncoder.VideoEncoded += messageService.OnVideoEncoded; // adding the listener
    videoEncoder.Encode(video) // executing the method that will fire the event
}
```



# Extension Methods

Extension methods allow us to add methods to an existing class without changing it's source code or creating a new class that inherits from it. This is implemented using `public static class` and `public static` member functions that as first argument have `this Class cls` where Class is the class the extension is for. For example to expand the string class: (Nb, a string is immutable so the function has to return a new string).

```c#
public static class StringExtensions
{
    public static string NewMethod(this String str) {}
}
```

In real world scenario's it is often better to change the source code or derive from the class, because when the original class gets the same method (with the same name and signature) it will override the custom extension method. But .NET does provide a lot of extension methods that are useful. For example the method's provided by LINQ.

# LINQ

LINQ stands for Language Integrated Query and provides the capability to query objects in a sql style manner. It allows queries on Objects (like collections), databases, XML, ADO.NET datasets. LINQ methods can be chained.

Commonly Used Linq Methods:

- `Where()`
- `Orderby()` and `OrderByDesc()`
- `Select`
- `Single()` and `SingleOrDefault()`
- `First()` and `FirstOrDefault()`, `Last()` and `LastOrDefault()`
- `Skip()` and `Take()` which are useful for pagination
- Aggregate functions: `Count()`, `Max()`, `Min()`, `Sum()` 

There are two forms of syntax, *LINQ Extension Methods* and LINQ Query Operators. The former form offering more functionality due to the use of lambda expressions. LINQ Query Operators always start with a from and always end with a select statement. Much like SQL.

```c#
var books = new Books.GetBooks(); // a list of books
// get a list of book titles of books under 10 dollars

// LINQ Query Operators.
var cheapBooks = 
    from b in books
    where b.Price < 10
    orderby b.Title;
    select b.Price;

// LINQ Extension Methods
var cheapBooks = books
    .Where(b => b.Price  < 10)
	.OrderBy(b => b.Title)
    .Select(b.Title)
```

# Nullable Types

Sometimes there is a need for a value-type to be able to be null. For example when someone does not wish to enter a value like their birthday into a database. But a default value-type does not allow this. There are two methods to declare a nullable type: `Nullable<T> name = null` or putting a question mark after a value type declaration: `int? n = null`

3 Members that are often used with a `Nullable<t>`

- `GetValueOrDefault()`
- `HasValue`
- `Value`

Using value when there is no value will throw an exception! Assigning a nullable into a regular value type is also not possible. Use `GetValueOrDefault()` Assigning the value of a nullable when it has a value or a different value when the value is null can be done using the `??` operator. Which is quite similar to the regular `?` ternary operator.  So instead of `int n2 = (n1 != null ) ? n1.GetValueOrDefault : 0;` use `int n2 = n1 ?? 0;`

```c#
class Program {
    static void Main(string[] args) 
    {
         Nullable<DateTime> date = null; // Method 1
         DateTime? date = null;			// Often used shorthand
         DateTime date2 = date.GetValueOrDefault(); // assigning to a value-type
         DateTime date3 = date ?? DateTime.Today;
    }
}
```



# Dynamics

Languages are either static or dynamic. This means that type resolution is done either at compile-time or run-time. C# started as a static language but since .NET 4 dynamic capability was introduced. So if we know that at runtime an object will have a certain method available we can compile the code. Like python the dynamic type can change at runtime as well. The downside is that it requires more testing to ensure the code won't crash at runtime. When changing types casting is sometimes required. For example from `long` to `int`. From `int` to `long` requires no casting however.

```c#
class Program {
    static void Main(string[] args) 
    {
        dynamic dyn = "abc";
        dyn.Method();
        dyn++; // this will crash the code cause at runtime "abc" is a string
        dyn = 5;
        long l = dyn // no casting required
    }
}
```





# Exception Handling

A try / catch block allows the program to handle exceptions without crashing. It is possible to have multiple catch blocks as long as they are put in order from most specific to most generic. For example the DivideByZeroException derives from the ArithmeticException class which in turn derives from the generic Exception class. 

```c#
class Program {
    static void Main(string[] args) 
    {
		try { var res = 5/0; }
    	catch (DivideByZeroException ex) {}
        catch (AritmethicException ex) {}
        catch (Exception ex) 
        {
            // Recover / throw the exception.
        }
    }
}
```

There is also a `finally` block, to allow cleanup of resources. For example file handles, network connections, database connections. But often this is replaced with `using` instead. Using a try catch block instead would be implemented like this:

```c#
class Program {
    static void Main(string[] args) 
    {
       StreamReader streamReader = null; 
        try {
            streamReader = new StreamReader(@"filepath");
            var content = streamReader.ReadToEnd();
        }
        catch {
            // ...
        }
        finally {
            if (streamReader != null) streamReader.Dispose();
        }
    }
}
```

With a using block it would look like this instead:

```c#
class Program {
    static void Main(string[] args) 
    {
        try 
        {
            using (var streamReader = new StreamReader(@"filepath")) 
            {
                // ...
            }
        }
        catch 
        {
            // ...
        }
    }
}
```

## Custom Exceptions

It is possible to create your own exception so the type of exception becomes more meaningful. 

```c#
public class CustomException : Exception 
{
    public CustomException(string message, Exception innerException)
    	: base(message, innerException)
    {
        
    }
}

try
{
    //...
    throw new Exception("Oops"); 
}
catch (Exception ex)
{
    // ...
    throw new CustomException("Error Message", ex);
}
```

In this example in the debugger the catch block will show the error message and have the "Oops" as inner-exception.

# Async/Await

Very similar to JavaScript (as explained in the codecademy course) there is a Async Await functionality. It provides asynchronous programming without the more complex use of threads or callback functions.



