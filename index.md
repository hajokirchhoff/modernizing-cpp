## Modernizing a C++ codebase

You'll find many pages about how great "modern c++" is. The community is exited about all the new stuff. coroutines, modules, new library features and much, much more, and rightfully so. C++20 is exciting. 

But you cannot just open your IDE and start adding C++20 code to your existing projects, because when you first try to compile your code with /std:c++20, you will be in for a surprise.

### What is the problem?
Even if your old codebase compiles fine, with 0 warnings, with your current compiler, there are many issues that will prevent you from using C++20.

- Your code may rely on language features that no longer available.
- It may contain errors that went unnoticed so far.
- You've created all your projects with default settings, which allow non-standard language extensions.
- Some variables or classes have names that are now reserved language keywords.

So, even though `C++20` is `C++`, it is not exactly backwards compatible.

These pages show my approach to modernizing a medium sized project with approx 2500 .cpp files.














You can use the [editor on GitHub](https://github.com/hajokirchhoff/modernizing-cpp/edit/gh-pages/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

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

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/hajokirchhoff/modernizing-cpp/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
