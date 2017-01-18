# Overview
Critical thinking is more important than rules, but rules give you more time to think about the more important 
things.

These are some guidelines for a Unity C# project that produce coherent and maintainable code.  Many guidelines are personal taste,
the important part is uniformity.

## Style

- UseCamelCase ForYour ClassesVariablesAndFunctions
- Brackets on a new line.
- Classes, public properties, and functions and function parameters begin with an upper cased letter (```MyClass```)
- Member variables and protected/private properties begin with a lower cased letter (```myVariable```)
- Constant members use ```regularVariableSyntax```.
- Interfaces begin with an upper-cased I (```IExtendableInterface```)
- Name things so their usage, scope, and type are not ambiguous, whenever possible.
- Booleans are always named in the form of a question (```isOn```, ```canBeHighlighted```)
- Function names are clear in whether they read data (```GetX```), write data (```UpdateY```), or have a specific purpose
that cannot be inferred from their signature (```ApplyValueInternal```, ```DebugSetValues```).  As a best practice, don't 
write functions that couple unrelated things together, break them up into multiple functions.
- Events use the suffix Event (```ForceAppliedEvent```)
- Event handlers use the prefix On (```OnForceApplied```)
- Enums should be singular (```enum StateType```) but enum flags should be plural (```enum StateFlags```)
- Use whitespace to separate blocks of related code.  Negative vertical space adds meaning and flow, and makes
spotting functions or lines that form a logical unit of work easier.
- Avoid nesting complex logic in a single line.  Break up work into multiple lines and put intermediate
results into named variables.
- Define one class/enum per file, with the same name as the file.  The exception would be for classes only 
used by the primary class, in that case place them above or at the top of the primary class.
- Lines should not extend too wide, they should fit in a side-by-side editor or vertical display without
requiring to scroll (recommend 110 characters).
- Pre-processor macros should not be indented, and symbols should be in macro-case (```#if FOO_BAR```).
- At the start of the project choose tabs or spaces, not both.
- Use ```<summary>``` tags when possible as they appear in intellisense popups.
- When modifying library code from an external source, follow it's style as closely as possible rather than 
these guidelines, and write comments on what you changed.

### General class order of definitions
- private inner classes/structs/enum definitions
- public variables and constants
- protected variables
- private variables
- public functions
- public static functions
- protected functions
- private functions
- private static functions

### Sample Style

```CSharp
public class MyClass : IComponent
{
    private float ratio;

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

- Do not use ternary operators with nested expressions, only pre-defined symbols or constants.

```CSharp
// BAD
int number = isDone ? x + y - 3 * scalar : 3 / scalar;

// GOOD
int number = isDone ? doneResult : temporaryResult;

// BEST
// Use a real if-else statement instead of the ternary operator.
int number;
if(isDone)
{
  number = x + y - 3 * scalar;
}
else
{
  number = 3 / scalar;
}
```

- Avoid optional parameters.

```CSharp
// BAD
public void ApplyForce(float force, int extraForce = 0, ForceType = ForceType.BasicForce, 
    bool isImmediate = false)
```

  Optional parameters are error prone when refactoring and are a code smell of a poorly defined function.  
  Break your function up into overloads and bubble down to a single implementation.  Apply all arguments 
  explicitly in the invokation of the function, possibly define defaults with a constant variable.

- However, do use optional parameter syntax to make function calls more explicit.  Applying this with booleans
is highly preferred.

```CSharp
// UNCLEAR
UpdateState(StateType.NewState, true);

// CLEAR
UpdateState(StateType.NewState, isDeferredUpdate: true);
```

- Avoid using return in the middle of a function.   The return statement should be singular and on the last 
line.  There are exceptions, most typically when handling error cases of input parameters at the top of a 
function.  If you choose to do an early return, make it isolated and clear that it is breaking control flow,
as the general assumption is that a function will execute fully before returning.

```CSharp
// EARLY OUT ! //
if(inputValue == null)
{
    return 0;
}
```

Does not apply to co-routine yield returns.

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

- Unless there is an explicit/implicit precondition that a type passed into a variable is not null, you must 
null check the variable before using it.

```CSharp
public void MyFunction(string value, MonoBehaviour otherClass)
{
    if(value != null && otherClass != null)
    {
        ...
    }
}
```

- Don't modify input parameters of functions when calculating a result, use temporary variables instead.
- Using var liberally is fine (```var keyword = "developer";```)

## Best Practices

- Minimize singleton usage.
- Minimize public static access.
- Don't throw exceptions.  Handle error cases.
- Consider alternatives to inheritance when possible.  Do not make inheritance trees more than 3 levels deep,
prefer single level inheritance of concrete classes.
- Avoid deeply nested loops.  If you have three or more levels of nesting, consider breaking the innermost 
loop(s) into their own functions.
- If a function is pure functional ie has no side effects, make it static.  A static function provides a
compile time guarantee that it won't touch member variables and is easier to reason about.
- Prefer clear code over liberal commenting.  Code always describes the exact behavior that will be executed, 
but comments have no compiler and can easily go out of date during refactoring.  DO comment when needed, but
consider if variable names and algorithm structure can inform the reader better first.
- Don't use magic numbers, put them into constants or named variables.
- Make no assumptions about function parameters.  Any input could be any value that the compiler allows it to 
be, whether null, negative, out of range, etc.  Unless pre-conditions are explicitly listed.
- List any assumptions a function's caller must make as pre-conditions in the function summary.

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
- Don't use any MonoBehaviour function signature ex. ```Update()``` unless you intend to implement that function so 
Unity invokes it.  Even in classes that don't extend MonoBehaviours.
