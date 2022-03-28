## /permissive-
The Visual Studio 2017 and earlier compilers where not very strict when it came to the C++ language standards.

### Enable ConformanceMode
Add `<ConformanceMode>true</ConformanceMode>` to your project.
![C++Settings](https://user-images.githubusercontent.com/16417930/158802810-2ff35e32-74b1-4c36-8de1-bd6f555e650e.png)

### Discussion
Perhaps the most problematic change for me was the "[Look up members in dependent base](https://docs.microsoft.com/en-us/cpp/build/reference/permissive-standards-conformance?msclkid=12163577ae8311ecbd52ecdaf807302c&view=msvc-170#look-up-members-in-dependent-base)" change. Consider the following example:
```cpp
template <typename B>
struct base_class
{
   using value_type = double;
   void func();
};

template <typename C>
struct derived: public base_class<C>
{
   void do_something()
   {
      value_type my_variable;
      ^^^^^^^^^^

      func();
      ^^^^^^
   }
};
```

What is just ordinary inheritance becomes more complicated, when templates are involed. The problem is that `value_type` and `func` look like they could be inherited from the `base_class`, but template specialization makes this a lot more difficult than it appears. Consider the following specialization:

```cpp
template <>
struct base_class<int>
{
};
```

It is a very primitive specialization for the data type `int`. Now consider
```cpp
void foo()
{
   derived<int> value;
   value.do_something();
}
```

`derived<int>` inherits from `base_class<int>`, which has a specialization where both, the *value_type* **and** *func* are missing. The program will not compile. But now consider this:

```cpp
using value_type = std::string;
void func()
{
   cout << "a global func";
}

void foo()
{
   derived<int> value;
   value.do_something();
}
```

This program *will* compile. `value_type` and `func` are not present in the template specialization and so not inherited. Instead they are taken from the global namespace.

But this is very difficult to see and can lead to totally unexpected situations. Imagine a more difficult template where specializations are a way to adapt or enhance the functionality. So specializations will be used frequently. Now image you have to write such a specialization and you forget to include a specific `using` directive such as the `value_type` above. It is easy to forget things like this. If the compiler goes looking for `value_type` someplace else, it might find something you did not expect. The program will compile, but behave unexpectedly.

This can be avoided if the language requires you to *explicitly* tell the compiler if the symbol should be inherited or not. Which is exactly what the change with "/permissive-" introduces.

Obviously this is more work and kind of annoying, especially since "it just worked" previously. Except it didn't. When the use of templates and specializations became more and more prominent, this kind of error popped up more frequently, which is why the C++ language standard changes the "Look up members in dependend base classes".

### Bottom line
It's more work, as you cannot rely on inheritance in these (template) situations any more. Think of it as `explicit` vs. `implicit` inheritance. But the code is more secure and more obvious this way.
