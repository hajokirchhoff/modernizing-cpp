## List of Visual Studio errors

[[TOC]]

## /permissive- with templates
A lot of problems come from relaxed rules the Visual Studio compiler has with templates. Templates have subtle language ambiguities that have been discovered and corrected over time. To resolve these ambiguities the compiler needs additional information by the programmer. These requirements have been added to the C++ language standard over time.

### C2760: syntax error: unexpected token 'identifier', expected ';' combined with <br>C7510: 'const_iterator': use of dependent type name must be prefixed with 'typename'
#### Description:
This line of code inside a template definition causes a C2760 _in combination_ with a C7510. *Note:* C7510 has different reasons and even different wordings. This explanation here  talks about a C2760 error in conjunction with the C7510. Both need to be present.

`renderer_map_t::const_iterator i=m_objects.find(a.get_type());`
#### Solution:
Add the keyword `typename` before the identifier in question.

`typename renderer_map_t::const_iterator i=m_objects.find(a.get_type());`

#### Bonus:
This particular instance should read

`auto i=m_objects.find(a.get_type());`

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

### C7510 use of dependent template name must be prefixed with 'template'
>...\message_queue\basic_message_transport.hpp(112): error C7510: 'handshake_response': use of dependent template name must be prefixed with 'template' <br>
>...\message_queue\basic_message_transport.hpp(112): note: This diagnostic occurred in the compiler generated function '...' 

*Note:* This is a different kind of C7510 error than the one above. Here, C7510 is the sole error code and the error text talks about a *template* name as opposed to a *typename*.
#### Description:
```cpp
template <typename Protocol>
void set_protocol()
{
    m_handshake_response=boost::bind(&Protocol::handshake_response<sock_type_t, boost::asio::yield_context>, Protocol(), _1, _2, _3);
}
```

The error here is with `&Protocol::handshake_response<...>`. Even though `handshake_response` clearly seems to be a template, the compiler cannot be sure it is. See [Stackoverflow](https://stackoverflow.com/questions/610245/where-and-why-do-i-have-to-put-the-template-and-typename-keywords) for a great explanation. This is why you have to explicitly tell the compiler that it actually `is` a template.

#### Solution:
Add the keyword `template` after the \:\:, right before the actual template.
```cpp
    m_handshake_response=boost::bind(&Protocol::template handshake_response<sock_type_t, boost::asio::yield_context>, Protocol(), _1, _2, _3);
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

### C2440 'initializing': cannot convert from 'const char [18]' to 'boost\::optional\<std\::string\>'
> ...\litwindow\libs\odbc\odbc_unittest\odbc_unittest.cpp(97): error C2440: 'initializing': cannot convert from 'const char [18]' to 'boost\::optional\<std\::string\>'<br>
> ...\litwindow\libs\odbc\odbc_unittest\odbc_unittest.cpp(97): note: Constructor for class 'boost\::optional\<std\::string\>' is declared 'explicit'
#### Description:
The code is trying to pass a string literal in an initializer list where a `boost\::optional\<std\::string\>` is expected.

```cpp
struct test_struct
{
	int m_id;
	boost::optional<float> m_optional_val;
	boost::optional<std::string> m_optional_textval;
};

void main()
{
   test_struct a = {10, 3, "optional string"}; // The string literal "optional string" is no longer accepted here.
}
```
### Solution:
Change the code to:
```cpp
void main()
{
   test_struct a = {10, 3, std\::string("optional string")};
}
```

### Details:

The third entry in the initializer list is a string literal, which is passed as a `const char*`. The member variable `m_optional_textval` however is defined as an `optional\<string\>`. There are several constructors for the compiler to choose from. The standard constructors accepting a `string` or `const string &` are ruled out here, because the argument is a `const char *`. There is one constructor that accepts a type compatible with `std\::string` and initializes the `optional` by first converting the type to `std\::string` and then assigning the `string` to the optional. But this constructor is marked as `explicit`. That is basically what the error message says here: The compiler tries this constructor, but cannot use it, because it has to be called explicitly.
