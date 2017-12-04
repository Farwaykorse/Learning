<!-------------------------------------------------------------><a id="top"></a>
# C++ :: Exceptions
<!----------------------------------------------------------------------------->

<!-- introduction -->

##### TOC #####
[Basics](#basics)
[Open Questions](#questions)

<!----------------------------------------------------------><a id="basics"></a>
## Basic 'Rules`
<!----------------------------------------------------------------------------->
- Don't use exceptions for control-flow
  Only use exceptions for *exceptional* cases.
  <!-- add clear reasoning, stack-unwinding, and ...-->
- Throw by Value; Catch by (const) reference
  <!-- example code -->
  <!-- reasons -->


<!-------------------------------------------------------><a id="questions"></a>
## Open Questions
<!----------------------------------------------------------------------------->
- How to find what exceptions can propagate to where?
- Lippincoth functions
  - Lippincott functions: (Jason Turner, C++ weekly) https://youtu.be/-amJL3AyADI
  - How?
  - When to (not) use these?
    Note: use, especially with templated code.
- noexcept
    - When, where and how to (not) use these
    - Risks?
- formatting strategie
- Where not to throw exceptions
- rethrowing
- using catch(...) // catch all
- prefenting performance hits?
- exception categories and when to use them
    - std::... when to use which?
    - custom exceptions
- logging
- when to use exceptions / assert / static_assert ?



Sources:
- [CppReference: Error handling](http://en.cppreference.com/w/cpp/error)
- http://www.acodersjourney.com/2016/08/top-15-c-exception-handling-mistakes-avoid/



-----------
[top](#top)
