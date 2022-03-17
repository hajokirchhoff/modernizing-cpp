## Modernizing a C++ codebase

You'll find many pages about how great "modern c++" is. The community is exited about all the new stuff. coroutines, modules, new library features and much, much more, and rightfully so. C++20 is exciting. 

But you cannot just open your IDE and start adding C++20 code to your existing projects. Try compiling your code with /std:c++20 for the first time and you will be in for a surprise.

### What is the problem?
Even if your old codebase compiles fine, with 0 warnings, with your current compiler, there are many issues that will prevent you from using C++20.

- Your code may rely on language features that no longer available.
- It may contain errors that went unnoticed so far.
- You've created all your projects with default settings, which allow non-standard language extensions.
- Some variables or classes have names that are now reserved language keywords.

So, even though `C++20` is `C++`, it is not exactly backwards compatible.

These pages show my approach to modernizing a medium sized project with approx 2500 .cpp files.


## Overview

## Preparation




## Ressources
Guides to C++20

- https://changkun.de/modern-cpp/pdf/modern-cpp-tutorial-en-us.pdf
- https://github.com/AnthonyCalandra/modern-cpp-features
- https://www.modernescpp.com/index.php/c-20-an-overview





### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).
