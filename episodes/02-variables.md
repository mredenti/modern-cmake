---
title: Variables 
teaching: 10
exercises: 10
---

::::::::::::::::::::::::::::::::::::::: objectives

- Learn about local variables.
- Understand that cached variables persist across runs.
- Know how to glob, and why you might not do it.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- How do variables work?

::::::::::::::::::::::::::::::::::::::::::::::::::



For this exercise, we will just directly run a CMake Script, instead of running `CMakeLists.txt`.
The command to do so is:

```bash
# Assuming you have a file called example.cmake:
$ cmake -P example.cmake
```

This way, we don't have so many little builds sitting around.

## Normal variables

The most basic way of defining a variable is with the `set()` command. A normal variable can be defined
in a `CMakeLists.txt` file as follows:

```cmake
# local.cmake
set(MY_VARIABLE "I am a local variable") # set(varName value... [PARENT_SCOPE])
message(STATUS "${MY_VARIABLE}")
```

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: instructor

The name of the variable, varName, can contain letters, numbers and underscores, with letters being
case-sensitive. The name may also contain the characters ./-+ but these are rarely seen in practice.

In CMake, a variable has a particular scope, much like how variables in other languages have scope
limited to a particular function, file, etc. A variable cannot be read or modified outside its own
scope.

The names of variables are usually all caps, and the value follows. You access a variable by using ${}, such as ${MY_VARIABLE}.1 CMake has the concept of scope; you can access the value of the variable after you set it as long as you are in the same scope. If you leave a function or a file in a sub directory, the variable will no longer be defined. You can set a variable in the scope immediately above your current one with PARENT_SCOPE at the end.


CMake is particularly flexible in that it is also possible to use this form recursively or to
specify the name of another variable to set. In addition, CMake doesn’t require variables to be
defined before using them. Use of an undefined variable simply results in an empty string being
substituted, similar to the way Unix shell scripts behave. By default, no warning is issued for use of
an undefined variable, but the --warn-uninitialized option can be given to the cmake command to
enable such warnings.

Strings are not restricted to being a single line, they can contain embedded newline characters.
They can also contain quotes, which require escaping with backslashes.

If using CMake 3.0 or later, an alternative to quotes is to use the lua-inspired bracket syntax where
the start of the content is marked by [=[ and the end with ]=]. Any number of = characters can
appear between the square brackets, including none at all, but the same number of = characters
must be used at the start and the end.

A variable can be unset either by calling unset() or by calling set() with no value for the named
variable. The following are equivalent, with no error or warning if myVar does not already exist:

```cmake

set(myVar)
unset(myVar)

```

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

Here we see the `set()` command, which sets a variable, and the `message()` command, which prints
out a string. The optional <mode> keyword `STATUS` determines the type of message, which influences the way the message is handled - there are other types (many other types in
CMake 3.15+) [place link to repository].


:::::::::::::::::::::::::::::::::::::::: challenge

## More about normal variables

Try the following:

- Remove the quotes in set. What happens?
- Try setting a cached variable using `-DMY_VARIABLE=something` **before** the `-P` flag. Which variable is shown?

:::::::::::::::::::::::::::::::::::::::: solution

CMake treats all variables as strings. In various contexts (e.g. CMake GUI and TUI), variables may be interpreted as a different type, but ultimately, they are just strings. When setting a variable’s value, CMake doesn’t require those values to be quoted unless the value contains spaces. If multiple values are given, the values will be joined together with a semicolon separating each value - the resultant string is how CMake represents lists.

```cmake
# local.cmake
set(MY_VARIABLE I am a local variable) # set(varName value... [PARENT_SCOPE])
message(STATUS "${MY_VARIABLE}")
```

```bash
$ cmake -P local.cmake
```

```output
-- I;am;a;variable
```

```bash
cmake -D MY_VARIABLE="I am a cached variable" -P local.cmake
```

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## Cache variables

Now, let's look at cache variables: 

```cmake
# cache.cmake
set(MY_CACHE_VAR "I am a cache variable" CACHE STRING "Description")
message(STATUS "${MY_CACHE_VAR}")
```

Unlike normal variables which have a lifetime limited to the processing of the CMakeLists.txt file, cache variables are stored in a special file called `CMakeCache.txt` in the build directory, and they persist between CMake runs. When you rerun the files generation stage, the cache is read in before starting

Once set, cache variables remain set until something explicitly removes them from the cache.

In a build, cached variables are set in the command line or in a graphical tool (such as `ccmake`, `cmake-gui`), and then written to a file called `CMakeCache.txt`. 

Feel free to look back at the example you built in the last lesson and investigate the `CMakeCache.txt` file in your build directory there. Things like the compiler location, as discovered or set on the first
run, are cached.

:::::::::::::::::::::::::::::::::::::::: callout

We have to include the variable type here, which we didn't have to do before (but we could have) -
it helps graphical CMake tools show the correct options. The main difference is the `CACHE` keyword
and the description. If you were to run `cmake -L` or `cmake -LH`, you would see all the cached
variables and descriptions.

:::::::::::::::::::::::::::::::::::::::::::::::::: 

:::::::::::::::::::::::::::::::::::::::: challenge

## More about normal variables

Try the following:

- Remove the quotes in set. What happens?
- Try setting a cached variable using `-DMY_VARIABLE=something` **before** the `-P` flag. Which variable is shown?

:::::::::::::::::::::::::::::::::::::::: discussion

The normal set command *only* sets the cached variable if it is not already set - this allows you to
override cached variables with `-D`. Try:

```bash
cmake -DMY_CACHE_VAR="command line" -P cache.cmake
```

You can use `FORCE` to set a cached variable even if it already set; this should not be very common.
Since cached variables are global, sometimes they get used as a makeshift global variable - the
keyword `INTERNAL` is identical to `STRING FORCE`, and hides the variable from listings/GUIs.

:::::::::::::::::::::::::::::::::::::::::::::::::: 

:::::::::::::::::::::::::::::::::::::::::::::::::: 

Since bool cached variables are so common for builds, there is a shortcut syntax for making one
using [`option`][]:

```cmake
option(MY_OPTION "On or off" OFF)
```

## Environment variables

Although rarely useful, CMake also allows the value of environment variables to be retrieved and set using a modified form of the CMake variable notation. The following example shows how to retrieve and set an environment variable:

```cmake

set(ENV{PATH} "$ENV{PATH}:/opt/myDir")

```

You can check to see if an environment variable is defined with `if(DEFINED ENV{name})` (notice the missing `$`).

::::::::::::::::::::::::::::::::::::::::: callout

## Note:

Setting an environment variable like this only affects the currently running CMake
instance. As soon as the CMake run is finished, the change to the environment variable is lost. In
particular, the change to the environment variable will not be visible at build time.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::  callout

## Handy tip:
Use [`include(CMakePrintHelpers)`][`CMakePrintHelpers`] to add the useful commands
`cmake_print_properties` and `cmake_print_variables` to save yourself some typing when debugging variables and properties.

::::::::::::::::::::::::::::::::::::::::::::::::::

## Target properties and variables

You have seen targets; they have properties attached that control their behavior. 
Properties are a form of variable that is attached to a target; you can use [`get_property`][] and
[`set_property`][], or [`get_target_properties`][] and [`set_target_properties`][] (stylistic preference) to
access and set these. You can [see a list of all
properties](https://cmake.org/cmake/help/latest/manual/cmake-properties.7.html) by CMake version;
there is no way to get this programmatically.
Many of these properties, such as [`CXX_EXTENSIONS`][], have a matching variable that starts with `CMAKE_`, such
as [`CMAKE_CXX_EXTENSIONS`][], that will be used to initialize them. So you can using set property
on each target by setting a variable before making the targets.

## Globbing

There are several commands that help with [`string`][]s, [`file`][]s, [`lists`][], and the like.
Let's take a quick look at one of the most interesting: glob.

```cmake
file(GLOB OUTPUT_VAR *.cxx)
```

This will make a list of all files that match the pattern and put it into `OUTPUT_VAR`. You can also
use `GLOB_RECURSE`, which will recurse subdirectories. There are several useful options, which you
can look at [in the
documentation](https://cmake.org/cmake/help/latest/command/file.html?highlight=glob#filesystem), but
one is particularly important: `CONFIGURE_DEPENDS` (CMake 3.12+).

When you rerun the *build* step (not the configure step), then unless you set`CONFIGURE_DEPENDS`,
your build tool will not check to see if you have added any new files that now pass the glob. This
is the reason poorly written CMake projects often have issues when you are trying to add files; some
people are in the habit of rerunning `cmake` before every build because of this. You shouldn't ever
have to manually reconfigure; the build tool will rerun CMake as needed with this one exception. If
you add `CONFIGURE_DEPENDS`, then *most* build tools will actually start checking glob too. The
classic rule of CMake was "never glob"; the new rule is "never glob, but if you have to, add
`CONFIGURE_DEPENDS`".

:::::::::::::::::::::::::::::::::::::::::  callout

## More reading

* Based on [Modern CMake basics/variables][]
* Also see [CMake's docs](https://cmake.org/cmake/help/latest/index.html)

::::::::::::::::::::::::::::::::::::::::::::::::::


:::::::::::::::::::::::::::::::::::::::: keypoints

- Local variables work in this directory or below.
- Cached variables are stored between runs.
- You can access environment variables, properties, and more.
- You can glob to collect files from disk, but it might not always be a good idea.
  
::::::::::::::::::::::::::::::::::::::::::::::::::

[Modern CMake basics/variables]: https://cliutils.gitlab.io/modern-cmake/chapters/basics/variables.html
