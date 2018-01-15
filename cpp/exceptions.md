<!-------------------------------------------------------------><a id="top"></a>
# C++ :: Exceptions
<!----------------------------------------------------------------------------->

<!-- introduction -->


##### TOC
- [Basic 'Rules'](#basics)  
- [Error handling](#handling)  
  - [Condition Testing (or: Where to throw exceptions?)](#conditions)
    - `static_assert`
    - `assert`
    - `[[nodiscard]]`
    - exceptions
- [Start with the `noexcept` specifier](#noexcept)  
- [What to throw](#what_to_throw)
- [The try-catch block](#try-catch)
  - How to find what exceptions can propagate to where?
  - [Catching](#catching)
  - [Lippincott functions](#lippincott)
- formatting strategie
- prefenting performance hits?
- logging
- ? add file and line to message?
- interfacing between different strategies
  occuring with libraries,  e.g. the cmath std library
  error-codes <-> exceptions
- [Open Questions](#questions)  


<!----------------------------------------------------------><a id="basics"></a>
## Basic 'Rules'
<!----------------------------------------------------------------------------->
Some general guidelines, commonly used for exception handling.
- Don't use exceptions for control-flow.  
  Throw an exception to signal that a function can't perform its assigned task.  
  Only use exceptions for *exceptional* cases.  
  Only for error handling. [Core Guidelines E.3](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#e3-use-exceptions-for-error-handling-only)
  <!-- add clear reasoning? stack-unwinding, and ...-->
- Throw by Value; Catch by (const) reference.  
  Throwing by value, because the sender will be destructed.  
  Catching by reference retains ownership (RAII) of the exception object,
  without unnecesary copying when rethrowing.  
  And thrown objects are not sliced when catched as it base class.
- Design your error-handling strategy around invariants.  
  An *invariant* must always be true.
  Used to describe a state (of an object) at a given point in the execution.
  Where possible make this part of the object construction.
  So only valid objects are constructed and any invalid variation always throws.

<!--------------------------------------------------------><a id="handling"></a>
## Error handling
<!----------------------------------------------------------------------------->
Based on:
-H. Sutter. *When and How to Use Exceptions* (2004)
[DrDobbs.com](http://www.drdobbs.com/when-and-how-to-use-exceptions/184401836)

Programming errors should be caught, where possible, by assertions.
Use exceptions for external errors, where the current code has no control over.
When:
1. precondition of a functions callees' can't be met
2. postconditions of a function can't be met
3. invariant where responsible for can't be maintained

Code that could cause an error is responsible for detecting and reporting it.
Callers are responsible for meeting preconditions of its callees.
Inside the callee not met preconditions can be dealt with using an assertion.

"The best error-handling strategies are those in which only designated major
interfaces are concerned with non-local error handling issues."
-B. Stroustrup. *The Design and Evolution of C++* (Addison-Wesley, 1994).

<!----------------------------------------------------------------------------->
#### Guarantees to give for the state of a program
The exception safety model as used in the STL.  
Before a `throw`, all modified objects must be placed in a 'valid' state.
To allow continued opperation and recovery.

**Basic**: For all operations. Ensure the program ends up in a valid state.
No resource leaks and basic invariants are maintained.  
Any allocated resource must either be deallocated or owned by an object that
handles the deallocation.  
**Strong**: Basic + The final state is either the original state (rolled back)
or the intended target state.  
Strategy: perform all the work that might emit an exception to the side and
only when you know it has succeeded, commit and modify the program state using
nonthrowing operations only. (eg. `std::swap()`)  
**Nofail**: Basic + An operation can never fail.
(Required for destructors, deallocation and swap functions.)
Any exception thrown has to be caught and handled in the function.

Every function should provide the strongest guarantee it can, without penalizing
code that doesn't need the guarantee.
And at the very least the basic quarantee needs to be ensured, anything else is
a bug.


Exceptions move error handling and recovery from the main control flow.


<!------------------------------------------------------><a id="conditions"></a>
### Condition Testing (or: Where to throw exceptions?)
<!----------------------------------------------------------------------------->
preconditions, postconditions, function testing
external influences

when to use exceptions / assert / static_assert ?
- `static_assert`
- `assert`
- `[[nodiscard]]`
  - Jason Turner. *C++17's [[nodiscard]] Attribute* (2016) [C++ Weekly](https://youtu.be/l_5PF3GQLKc)
- exceptions


<!-------------------------------------------------------><a id="noexcept"></a>
## Start with the `noexcept` specifier
<!----------------------------------------------------------------------------->
To specify if a function can throw an exception.  
Effect: If a function declared `noexcept` throws `std::terminate()` is called.

Since C++17 part of the type-system (but not the function signature).
<!-- concequences? -->

Advantages:
- Helps understanding the exception behaviour, when reading the code.
- Helps optimizers reduce possible execution paths.
- Forces you to concider exception handling early on and evaluate which
  functions can throw.

Disadvantages: 
- Overusing or forgetting to reevaluate after edits can result in a crash
  (termination) for an exception that could have been handled.
- Removing `noexcept` from a function type breaks the contract with its users.
  Although throwing an unhandled exception could result in the same problem
  anyway.

When to use:
- On everything that can't throw:
  - Functions written in C (or other language without exceptions).
  - Functions that don't throw and only call functions that don't or can't
    throw.  
    *Note*: this requires rechecking on every change.
- For functions that should not ever throw.  
  When continued operation doesn't make sense or there is no way or need to
  recover from the exception.
  *For example*: running out of memory when trying to allocation a small amount
  of memory.  
  **Comment**: When declaring a function `noexcept` that calls anything that
  might throw, make this explicit.
  Use at least `noexcept(true)` for these, to help find them later, or use a
  macro variable to control its operation.
  Add a comment to the top of the function what exceptions can be thrown and why
  termination is prefered.
- For functions that are conditionally throwing.
  The `noexcept` specifier can depend on a constant boolean expression,
  such as: `noexcept(std::is_pod_v<T>)`.  
  Use the `noexcept` operator to test a function and any operation for having a
  noexcept specifier: `noexcept(noexcept(f(a, b)))`.  
  *Note*: the `noexcept` operator can be used in unit tests with `static_assert` 
  to find accidental regressions.
- *Note*: `constexpr` functions can only throw when evaluated at run time.

Core Guidelines:  
[F.6: If your function may not throw, declare it `noexcept`](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f6-if-your-function-may-not-throw-declare-it-noexcept)  
[E.12: Use `noexcept` when exiting a function because of a throw is impossible
or unacceptable](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#e12-use-noexcept-when-exiting-a-function-because-of-a-throw-is-impossible-or-unacceptable)


<!---------------------------------------------------><a id="what_to_throw"></a>
## What to throw
<!----------------------------------------------------------------------------->
A `throw` takes an object.
This should be given by value since destructors will be called while it moves up
the call chain.
Until it meets a catch handler for the given object type.

These exception objects must represent the logical reason for the throw.
The main purpose is handeling the error, not identification of the code that
noticed the problem.

- Throw custom objects, not build-in types, to make use of the type system to
  identify the error.
  Specific errors can be caught, and a general handler for a category can catch
  by the parent type.
- Prefere throwing instances of classes derived from the std::exception class.
  This allows catching by these when exeptions cross subsystem boundaries.

Example:
````
#include <stdexcept>
struct MyException : public std::runtime_error
{
  public: MyException() : runtime_error{"MyException"} {}
};
````
The standard library exception categories:  
`<exception>`
- `std::exception` the base of all STL exception types.
  - `std::bad_exception` thrown by the C++ runtime.
`<stdexcept>`
- `std::runtime_error` conditions only detectable at run-time.
  Result of bad data from an external user. And not easily predicted.
  - `std::range_error` mathematical range error, result can't be expressed in
    the destination type.
    [*Note* 1](#throw_note_1)
  - `std::overflow_error` arithmetic overflow.
    Result is to large for the destination type.
    [*Note* 1](#throw_note_1)
  - `std::underflow_error` arithmetic underflow.
    When the result is a [subnormal (denormal)](https://en.wikipedia.org/wiki/Denormal_number)
    floating-point value.
    [*Note* 1](#throw_note_1)
- `std::logic_error` logical violations of preconditions or class invariants.
  These are bugs that should be fixed, or caught at the source.
  Or if the case, propagated up to where can be determined if it is a runttime
  error. Such that it can be handled at the source.
  - `std::invalid_argument` invalid arguments.
  - `std::length_error` exceeding maximum allowed size for an object, as
    defined by the implementation.
  - `std::out_of_range` outside *expected* range.
    While attempting to access elements with a bounds-checked method.
  - `std::domain_error` mathematical domain error.
    Input arguments are outside the domain for which the operation is defined.
`<future>`
- `std::future_error` (derived from `std::logic_error`).
`<regex>`
- `std::regex_error` (derived from `std::runtime_error`).
`<system_error>`
- `std::system_error` (derived from `std::runtime_error`).
  Associated with OS interfacing, such as the constructor of
  `std::thread`, to report a platform dependend `std::error_code`.
`<optional>`
- `std::bad_optional_access` (derived from `std::exception`)
`<variant>`
- `std::bad_variant_access` (derived from `std::exception`)
`<any>`
- `std::bad_any_cast` (derived from `std::exception`)
`<new>`
- `std::bad_alloc` (derived from `std::exception`)
  Reports failure to allocate storage.
  *Note*: can also be thrown by other exceptions.
  - `std::bad_array_new_length`

*Note* 1: exceptions are not used by standard library mathematical functions.
<a id="throw_note_1"></a>  
[CppReference, Math error handling](http://en.cppreference.com/w/cpp/numeric/math/math_errhandling)
*Note* 2: you almost always inherit from the exceptions defined in
`<stdexcept>`.  


<!-------------------------------------------------------><a id="try-catch"></a>
## The `try`-`catch` block
<!----------------------------------------------------------------------------->
There is no need to place a try-catch block arround every operation that might
fail.

Track what exceptions any called function might throw.  
<!-- TODO Tools?? -->
For now I haven't found anything to help in this besides manual checking and
documenting.

Concider a `try`-`catch` block at the top of the failing task,
where a decission is made on how to execute the task.
I.e. not where the error occures, but where a decission was made that led to
failing operation.

*Note*: everything between the `throw` and `catch` will be destructed.

<!--------------------------------------------------------><a id="catching"></a>
### Catching

**Only** catch an exception if it can be handled, or at least evaluated, in
a meaningfull way.
- If there is different strategy to achive the intended result, try it.
  But make sure it doesn't end up in an unending loop, and the alternative
  would be sensible.  
  And only if the information required for the decission is available.
- When an exception can't be handled,
  but a more specific problem description is available,
  a new more specific exception can be thrown.
  Or it can be rethrown, using `throw;`.
- What can't be handled will move further up the call-stack.
  Until it is caught, or it will terminate the program if there is no way to
  recover.


<!-- `catch (const std::range_error& e)`
the default
-->
<!-- `catch (std::range_error& e)`
if you want to edit something in e
-->
<!-- `catch (...)` Catch All
handle everything, and don't come back if you want to ensure noexcept
throw something here, but know it is un unknown error ...
-->

<!-- Rethrowing -->
<!--
  `throw;` rethrows the current exception, without slicing it.
  -->
<!--
  `throw e;` would rethrow the exception with the potentially sliced
  type as defined for the variable `error_var`.
  -->

<!-- catch all in main() only usefull if you do some logging -->


<!------------------------------------------------------><a id="lippincott"></a>
### Lippincott functions
<!----------------------------------------------------------------------------->
<!--- Named by Jon Kalb, after Lisa Lippincott, who tought him the technique.-->
- Jason Turner. *Lippincott functions* (2017), [C++ weekly, episode 91](https://youtu.be/-amJL3AyADI)
- Nicolas Guillemot.
  *Using a Lippincott Function or Centralized Exception Handling* (2013),
  [C++ Secrets](http://cppsecrets.blogspot.nl/2013/12/using-lippincott-function-for.html)
``````
foo lippincott()
{
  try
  {
    throw; // lippincott: pick up the exception.
  }
  catch (const MyException&)
  {
    // handling
  }
  catch (...)
  {
    // handling
  }
}

foo function()
{
  try
  {
    return throwing_operation();
  }
  catch (...)
  {
    return lippincoth();
  }
}
``````
- Reusable code for multiple 'similar' or templated functions.
- Error-handling code is replaced by a function-call to an external
  function.
  Moving the rarely executed error-handling code out.
  - Might improve readability of the normal code execution path.
  - Better `inline` decissions can be made for shorter functions.
    Especially since error handling should be a rare occurance for which
    inlineing is not beneficial.
- The `noecept` specifier can be added on both the calling and lippincott
  function, **if** it handles **all** errors, without rethrowing.
- The `[[noreturn]]` attribute should be used on the lippincott function,
  **if** it **always** rethrows.
  <!-- Sample
`````
[[noreturn]] foo lippincott()
{
  ...
}
`````
-->


<!-------------------------------------------------------><a id="questions"></a>
## Open Questions
<!----------------------------------------------------------------------------->
- [`noexcept`](#noexcept)
  Noexcept is part of the type-system since C++17. What are the consequences?
<!--- using noexcept(expression) with `if constexpr (..)` ?? -->
- What to do with potential errors thrown by complex algorithm implementations.
  Like for `operator==` on a `std::vector`?  
  Probably won't throw ... but if it does ... all is lost?  
  Or will it only throw `std::bad_alloc`?
  [CppReference.com lexicographical_compare](http://en.cppreference.com/w/cpp/algorithm/lexicographical_compare)


<!---------------------------------------------------------><a id="sources"></a>
## General Sources
<!----------------------------------------------------------------------------->
- [docs.microsoft, Stack unwinding](https://docs.microsoft.com/en-gb/cpp/cpp/exceptions-and-stack-unwinding-in-cpp)
- [Core Guidelines, E: Error handling](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-errors)
- [iso Cpp, exceptions FAQ](https://isocpp.org/wiki/faq/exceptions)
- [CppReference, Error handling](http://en.cppreference.com/w/cpp/error)
- [CppReference, Exceptions](http://en.cppreference.com/w/cpp/language/exceptions)
- [CppReference, Math error handling](http://en.cppreference.com/w/cpp/numeric/math/math_errhandling)


-----------
[top](#top)
