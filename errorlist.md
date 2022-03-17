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
