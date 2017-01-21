# Overview
Critical thinking is more important than rules, but rules give you more time to think about the more important 
things.

These are some guidelines for a Unity C# project that produce coherent and maintainable code.  Many guidelines are personal taste,
the important part is uniformity.

## Style

- UsePascalCase ForYour Namespaces, Classes, Enums, Variables, And Methods. Two letter names can be all caps (eg. UI.Drawing).
- useCamelCase for properties, events, instanceVariables, constants, and methodParameters
- Interfaces begin with an upper-cased I (```IExtendableInterface```)
- Brackets on a new line (Allman-style).
- Name things so their usage, scope, and type are unambiguous, whenever possible.
- Booleans are always named in the form of a question (```isOn```, ```canBeHighlighted```)
- Method names are clear in whether they read data (```GetX```), write data (```UpdateY```), or have a specific purpose
that cannot be inferred from their signature (```ApplyValueInternal```, ```DebugSetValues```).  Don't write methods that 
couple unrelated things together, break them up into multiple methods.
- Events use the suffix Event (```forceAppliedEvent```)
- Event handlers use the prefix On (```OnForceApplied()```)
- Enums should be singular (```enum StateType```) but enum flags should be plural (```enum StateFlags```)
- Use whitespace to separate blocks of related code.  Negative vertical space adds meaning and flow, and makes
spotting methods or lines that form a logical unit of work easier.
- Avoid nesting complex logic in a single line.  Break up work into multiple lines and put intermediate
results into named variables or methods.
- Define one class/enum per file, with the same name as the file.  The exception would be for protected/private 
classes or enums, in that case place them above or at the top of the primary class.
- Lines should not extend too wide, they should fit in a side-by-side editor or vertical display without
requiring to scroll (recommend 110 characters).
- Pre-processor macros should not be indented, and symbols should be in macro-case (```#if FOO_BAR```).
- At the start of the project choose tabs or spaces, not both.
- Use ```<summary>``` tags when possible as they appear in intellisense popups.
- Document ALL interface methods.  There's no immediate source code to clarify questions about their implementation.
- When modifying library code from an external source, follow it's style as closely as possible rather than 
these guidelines, and write comments on what you changed.

### Class order of definitions
- inner classes/structs/enum definitions
- events, variables, and constants
- properties
- methods


### Class order of access type
This would be the order within a single category above.
- public 
- public static
- protected
- protected static
- private
- private static

### Sample Style

```CSharp
public class MyClass : IComponent
{
    private float ratio;
    
    public int MyProperty
    {
        get;
        private set;
    }

    public int GetRatioClamped()
    {
        // For really really long lines of comments, we should not let them extend off horizontally, they
        // should be broken into multiple lines.
        int clamped = Mathf.Clamp01(ratio);

        return clamped;
    }
}
```

## Syntax

- Always use brackets for every conditional statement (if, else if, else).  No one/two liners.

```CSharp
// WRONG
if(isOn) DoSomething();

// WRONG
if(isOff) 
    DoSomethingElse();

// RIGHT
if(isActive)
{
    DoTheRightThing();
}
```

- Do not use the ternary operator with nested expressions, only trivial cases (only pre-defined symbols or constants).
Same with the null coalescing operator (??), the null case can use a trivial constructor or factory method/property.
Single line expressions do not work well with line debuggers.  Prefer if-else in the vast majority of situations.

```CSharp
int number = isDone ? foo : bar;
string myString = inputString ?? string.Empty;
```

- Avoid optional parameters.

```CSharp
// BAD
public void ApplyForce(float force, int extraForce = 0, ForceType = ForceType.BasicForce, 
    bool isImmediate = false)
```

  Optional parameters are error prone when refactoring and are a code smell of a poorly defined method.  
  Break your methods up into overloads and bubble down to a single implementation.  Apply all arguments 
  explicitly in the invokation of the method, possibly define defaults with a constant variable.

- However, do use optional parameter syntax to make method calls more explicit.  Applying this with booleans
is highly preferred.

```CSharp
// UNCLEAR
UpdateState(StateType.NewState, true);

// CLEAR
UpdateState(StateType.NewState, isDeferredUpdate: true);
```

- Avoid using return in the middle of a method.   The return statement should be singular and on the last line.  
- Does not apply to co-routine yield returns.
- One other exception is for handling error cases at the top of a method, which can clarify expectations and reduce 
cyclomatic complexity.  Syntax restrictions are relaxed in this specific case.  Make it clear that the early return 
is breaking control flow, as the general assumption is that a method will execute fully before returning.

```CSharp
// Early outs can also use single line if statements, since control flow will be broken anyway.
// EARLY OUT! //
if(inputValue == -1) { return null; }
```

- For complex switch statements that introduce scoped variables, use the following syntax.  ALWAYS handle the default case.

```CSharp
switch(type)
{
    case StateType.Start:
    {
        int temporaryValue = x + y;
        toReturn = temporaryValue * 4f;
    } break;
    case StateType.Next:
        ...
    default:
    {
        Debug.LogError("Unknown StateType: " + type);
    } break;
}
```

- Unless there is an explicit pre-condition that a type passed into a variable is not null, you must 
check the variable before using it.  Any input could be any value that the compiler allows it to 
be, whether null, negative, out of range, etc.

```CSharp
public void MyMethod(string value, MonoBehaviour otherClass)
{
    if(value != null && otherClass != null)
    {
        ...
    }
}
```

- List any assumptions a method's caller must make as pre-conditions in the method summary.

```CSharp
/// <summary>
/// Calculates the foobar of the parameters.
/// Pre-condition: numProducts cannot be 0.
/// </summary>
public int GetFooBar(int totalBalance, int numProducts)
{
    // If numProducts is 0, we have a divide by 0 error.
    return totalBalance / numProducts;
}
```

- Use Allman style for lambda expressions.

```CSharp
doSomething(callback: () =>
{
    // Lambda function in here.
});
'''

- Don't modify input parameters of methods when calculating a result, use temporary variables instead.
- Using var liberally is fine (```var keyword = "developer";```)

## Best Practices

- Minimize singleton usage.
- Minimize public static access.
- Don't throw exceptions.  Handle error cases.
- Consider alternatives to inheritance when possible.  Do not make inheritance trees more than 3 levels deep,
prefer single level inheritance of concrete classes.
- Avoid deeply nested loops.  If you have three or more levels of nesting, consider breaking the innermost 
loop(s) into their own methods.
- If a method is pure functional ie has no side effects, make it static.  A static method provides a
compile time guarantee that it won't touch member variables and is easier to reason about.
- Prefer clear code over liberal commenting.  Code always describes the exact behavior that will be executed, 
but comments have no compiler and can easily go out of date during refactoring.  DO comment when needed, but
consider if variable names and algorithm structure can inform the reader better first.
- Don't use magic numbers, put them into constants or named variables.
- Don't put unnecessary logic in an Update loop, prefer event driven logic or co-routines.
- Update() is a general purpose name for a general purpose method.  It's best to break it up into one or more
clearly named methods that are contextual to the particular MonoBehaviour.
- Don't use MonoBehaviour constructors, and don't call constructors on classes that may spawn MonoBehaviours 
on constructor defined member variables.

```CSharp
// BAD
public class MyScript : MonoBehaviour
{
    public OtherClass member = new OtherClass();
}

public class OtherClass
{
    GameObject.Instantiate(Prefab);
}
```
    
- Don't do equality checks on floating point numbers, check against a range.
- Don't use any MonoBehaviour method signature ex. ```Update()``` unless you intend to implement that method so 
Unity invokes it.  Even in classes that don't extend MonoBehaviours.
