---
title: "Python Notes"
date: 2017-05-16T22:32:05+09:00
lastmod: 2017-05-16T22:32:05+09:00
draft: false
keywords: []
description: ""
tags: ['python']
categories: ['Learning']

isCJKLanguage: false

comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
showMeta: true
showActions: true
---

<!-- toc -->

# 1. Numeric Values

```
>>> 17 // 3    # floor division
5
>>> 17 % 3     # remainder
2
>>> 2 ** 7  # 2 to the power of 7
128
```

# 2. Strings

* Unlike other languages, special characters such as `\n` have the same meaning with both single (`'...'`) and double (`"..."`) quotes. The only difference between the two is that within single quotes you don’t need to escape `"` (but you have to escape `\'`) and vice versa.

* If you don’t want characters prefaced by `\` to be interpreted as special characters, you can use raw strings by adding an `r` before the first quote:

```
>>> print('C:\some\name')  # here \n means newline!
C:\some
ame
>>> print(r'C:\some\name')  # note the r before the quote
C:\some\name
```

* Python strings cannot be changed — they are immutable

# 3. Lists

* lists are a mutable type, i.e. it is possible to change their content

* To iterate over the indices of a sequence, you can combine `range()` and `len()` as follows:
```
>>> a = ['Mary', 'had', 'a', 'little', 'lamb']
>>> for i in range(len(a)):
...     print(i, a[i])
...
0 Mary
1 had
2 a
3 little
4 lamb
```
Or use the `enumerate(list, start=0)` function:
```
>>> seasons = ['Spring', 'Summer', 'Fall', 'Winter']
>>> list(enumerate(seasons))
[(0, 'Spring'), (1, 'Summer'), (2, 'Fall'), (3, 'Winter')]
```

## 3.1 List Comprehensions

Examples:
```
squares = [x**2 for x in range(10)]
```
```
>>> [(x, y) for x in [1,2,3] for y in [3,1,4] if x != y]
[(1, 3), (1, 4), (2, 3), (2, 1), (2, 4), (3, 1), (3, 4)]
```

# 4. Functions

* Global variables may be referenced, but cannot be assigned a value within a function (because they will be treated as local variables) unless named in a global statement.

* Arguments are passed using _call by value_ (where the value is always an _object reference_, not the value of the object).

* Functions without a `return` statement returns `None`

## 4.1 Functions with Variable Number of Arguments

There are three forms, which can be combined.

### 4.1.1. Default Values

For example:

```
def ask_ok(prompt, retries=4, reminder='Please try again!'):
```

Default values are evaluated at the point of function definition in the defining scope, so that

```
i = 5

def f(arg=i):
    print(arg)

i = 6
f() # will print 5
```

The default value is evaluated only once. This makes a difference when the default is a mutable object such as a list. For example

```
def f(a, L=[]):
    L.append(a)
    return L

print(f(1))  # [1]
print(f(2))  # [1, 2]
print(f(3))  # [1, 2, 3]
```

### 4.1.2. Keyword Arguments

Can rearrange the order of arguments by specifying them in the form `func(kwarg3=val3, kwarg1=val1, kwarg2=val2)`.

Also, we have the form `def func(a, **kwargs)`
to hold all passed keyword arguments in a dict.

### 4.1.3. Arbitrary Argument Lists

`def func(a, *args)`

## 4.2 Unpacking Argument Lists

```
>>> def func(a, b='1', c='2')
...    print(a, b, c)

d = {"a": 0, "b": 1, "c": 2}
func(**d)
```

## 4.3 Lambda Expressions
Small anonymous functions can be created with the lambda keyword.

# 5. Tuples

* Tuples are immutable

# 6. Modules

* A module can contain executable statements as well as function definitions.
    * functions can be called by `module_name.func_name()`
    * statements are executed only the first time the module is imported.
* `from module_name import *` imports all names except those beginning with an underscore (_).
* The built-in function `dir(module_name)` is used to find out which names a module defines. It returns a sorted list of strings

# 7. Classes

* Inheritance of multiple base classes is allowed.

## 7.1 Scopes

* Directly used variable
    * read: search-up from local until global
    * write: create variable in local scope
* `nonlocal`: search-up _except_ global
* `global`: search in global scope _only_


# 8. Others

## 8.1 Generator

A function which returns a generator iterator. It looks like a normal function except that it contains yield expressions for producing a series of values usable in a for-loop or that can be retrieved one at a time with the next() function.

```
def reverse(data):
    for index in range(len(data)-1, -1, -1):
        yield data[index]

for char in reverse('golf'):
    print(char)

```

