## List of Visual Studio errors

[[TOC]]

## /permissive- with templates
A lot of problems come from relaxed rules the Visual Studio compiler has with templates. Templates have subtle language ambiguities that have been discovered and corrected over time. To resolve these ambiguities the compiler needs additional information by the programmer. These requirements have been added to the C++ language standard over time.

### C2760: syntax error: unexpected token 'identifier', expected ';' <br>C7510: 'const_iterator': use of dependent type name must be prefixed with 'typename'
#### Description:
This line of code inside a template definition causes a C2760 in combination with a C7510.

   renderer_map_t::const_iterator i=m_objects.find(a.get_type());
#### Solution:
Add the keyword `typename` before the identifier in question.

   `typename renderer_map_t::const_iterator i=m_objects.find(a.get_type());

#### Bonus:
This particular instance should read

   auto i=m_objects.find(a.get_type());

#### What is the problem:
Old code:
```cpp
template <typename TargetType>
class renderer
{
   typedef std::map<prop_t, std::pair<render_object_t, tstring> > renderer_map_t;

   bool find_render_object(const const_accessor &a, std::pair<render_object_t, tstring> &rc)
   {
      renderer_map_t::const_iterator i=m_objects.find(a.get_type());
```

After:
```cpp
template <typename TargetType>
class renderer
{
   typedef std::map<prop_t, std::pair<render_object_t, tstring> > renderer_map_t;

   bool find_render_object(const const_accessor &a, std::pair<render_object_t, tstring> &rc)
   {
      *typename* renderer_map_t::const_iterator i=m_objects.find(a.get_type());
```


### C5043 (warning): 'litwindow::odbc::sqlreturn::do_log_errors': exception specification does not match previous declaration
#### Description:
The declaration of the method differs from the definition.

Here the declaration specifies a `throws()` Exception Specifications, but the implementation does not.
Old code:
```cpp
.h
struct sqlreturn
{
   bool do_log_errors const throws();
};

.cpp
bool sqlreturn::do_log_errors() const
{
}
```
 In `/permissive` mode the compiler simply accepts that and assumes `throws()`, even if it is missing from the declaration. In `/permissive-` mode this is no longer accepted.

 #### Solution:
 Add `throws()` to the declaration
 
   bool sqlreturn::do_log_errors() const throws()

or get rid of `throws()` altogether. Exception Specifications where a thing pre 2000, but are now considered bad practice. See: https://stackoverflow.com/questions/1055387/throw-keyword-in-functions-signature

### error C2440: '\<function-style-cast\>': cannot convert from 'initializer list' to '_litwindow\::odbc\::sqldiag_'
> ...litwindow\libs\odbc\lwodbc.cpp(202): error C2440: '\<function-style-cast\>': cannot convert from 'initializer list' to 'litwindow::odbc::sqldiag'<br>
> ...\litwindow\libs\odbc\lwodbc.cpp(202): note: No constructor could take the source type, or constructor overload resolution was ambiguous

#### Description:
The code is trying to create an object of type `sqldiag`, but there is no constructor that accepts the `initializer list` (the parameters).

```cpp
.h
struct sqldiag {
   sqldiag(char p_state[5], SQLINTEGER p_native_error, const string &p_msg);
};

.cpp
   sqldiag value("LWODB", err_logic_error, "<<Logic error: Requested diagnostics record does not exist!>>");
```
What seems suprising is that this code compiles just fine with `/permissive-`. What is going on?

sqldiag has a constructor that accepts an array of 5 chars ```char p_state[5]```. The code tries to call it with a null terminated string literal "LWODB".

#### Solution:
Change the constructor from `char p_state[5]` to a `const char p_state[5]`.

String literals are actually of type `const char *`, not `char *`. This makes sense, since string literals are not supposed to be modified. But earlier versions of the compiler or the `/permissive` switch allow this anyway.

Now, with `/permissive-` this is no longer allowed. Thus the compiler sees a string literal (`const char *`), looks for a constructor but does not find one. The existing constructor wants a `char *`, which no longer matches, causing the error "cannot convert ...  No constructor could take the source type..."


```cpp
struct sqldiag {
   sqldiag(const char p_state[5], SQLINTEGER p_native_error, const string &p_msg);
};
```
### error C2664 cannot convert argument / A non-const reference may only be bound to an lvalue
> ...\litwindow\libs\odbc\table.cpp(24): error C2664: 'void litwindow::odbc::table::init(litwindow::odbc::connection &)': cannot convert argument 1 from 'litwindow::odbc::shared_connection' to 'litwindow::odbc::shared_connection &'<br>
> ...\litwindow\libs\odbc\table.cpp(24): note: A non-const reference may only be bound to an lvalue
```cpp
inline shared_connection LWODBC_API default_connection()
{
   return named_connection(tstring());
}

void table::init(shared_connection &s);

table::table()
{
    init(default_connection()); // C2664 ... note: A non-const reference may only be bound to an lvalue
}

```

#### Description:
The code is trying to pass or assign a temporary variable to a parameter or other variable that is declared as an lvalue (reference).

Here, `default_connection()` returns a newly created object - a temporary - which is then passed to `init`. `init` takes a reference to the parameter, which makes it possible for `init` to modify the value, which would be discarded. And worse, `init` might decide to keep the reference and attempt to modify the value later on - when the temporary object no longer exists.

#### Solution:
Add a const: change the parameter from `shared_connection &s` to `const shared_connection &s`. Temporary objects are allowed to be passed to const references.
```cpp
void table::init(shared_connection &s);
```

#### Details:
Previously the Visual Studio compiler allowed this:
```cpp
bool consume(int how_much, int &from_what_inout)
{
   bool rc = consume <= from_what_inout;
   if (rc)
       from_what_inout -= howmuch;
   return rc;
}

void main()
{
   int start = 10;
   cout << consume(1, start) << endl;       // Okay, returns true.
   cout << consume(1, start + 1) << endl;   // Also would return true, but no longer compiles, error C2664
}
```
The idea here is that the function `consume`
1. operates on the `from_what_inout` parameter
2. returns true/false if the operation was successful
3. "returns" the "remainder" of the operation by modifying the reference "from_what_inout"

So after consume, the "from_what_inout" contains whatever remains after the op.

The problem with this is that it is easy to shoot yourself into the foot (#footgun). See (https://stackoverflow.com/questions/6967927/non-const-reference-may-only-be-bound-to-an-lvalue) and many others for more info.
