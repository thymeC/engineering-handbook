# Python decorators


## Reference
- https://realpython.com/primer-on-python-decorators/
- https://pythonbasics.org/decorators/

## What is python decorator

Add functionality to an existing function with decorators. This is called metaprogramming.
A function can **take a function as argument** (the function to be decorated) and **return the same function** with or without extension.

Python中的装饰器是一种设计模式，用于在不修改原有函数代码的情况下，增加函数额外的功能。
装饰器本质上是一个函数，它**接受一个函数作为参数并返回一个新的函数**。使用装饰器可以把扩展功能和业务逻辑分开，使得代码更加模块化和可重用。

装饰器通过**闭包**（closure）来实现，即内部函数可以访问外部函数的变量。在装饰器中，内部函数（inner function）可以访问外部函数的参数，包括被装饰的函数本身。这样做有几个好处：

1. **封装性**：装饰器将扩展功能封装在内部，使得被装饰的函数保持清晰和专注于其主要功能。
2. **可重用性**：装饰器可以被应用到多个函数上，而不需要修改这些函数的代码。
3. **灵活性**：装饰器可以在运行时动态地添加功能，而不需要修改函数的定义。
4. **非侵入性**：装饰器不会改变原有函数的代码，因此不会引入潜在的错误。


## write decorator

1. simple decorator

```python
def decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

def say_whee():
    print("Whee!")

say_whee = decorator(say_whee)
```

use decorator with syntactic sugar
```python
def decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

@decorator
def say_whee():
    print("Whee!")
```

2. decorator function with arguments

```python
def do_twice(func):
    def wrapper_do_twice(*args, **kwargs):
        func(*args, **kwargs)
        func(*args, **kwargs)
    return wrapper_do_twice
```

3. return value from decorator
```python
def do_twice(func):
    def wrapper_do_twice(*args, **kwargs):
        func(*args, **kwargs)
        return func(*args, **kwargs)
    return wrapper_do_twice
```

4. find yourself
```
>>> say_whee
<function do_twice.<locals>.wrapper_do_twice at 0x7f43700e52f0>

>>> say_whee.__name__
'wrapper_do_twice'
```
After being decorated, the function name is changed to the name of the decorator function.

How to fix:
```python
import functools

def do_twice(func):
    @functools.wraps(func)
    def wrapper_do_twice(*args, **kwargs):
        func(*args, **kwargs)
        return func(*args, **kwargs)
    return wrapper_do_twice
```

5. decorator with arguments
```
import functools

def repeat(num_times):
    def decorator_repeat(func):
        @functools.wraps(func)
        def wrapper_repeat(*args, **kwargs):
            for _ in range(num_times):
                value = func(*args, **kwargs)
            return value
        return wrapper_repeat
    return decorator_repeat
```

6. decorator support different format
```
@repeat
@repeat(num_times)
```

```
import functools

def repeat(_func=None, *, num_times=2):
    def decorator_repeat(func):
        @functools.wraps(func)
        def wrapper_repeat(*args, **kwargs):
            for _ in range(num_times):
                value = func(*args, **kwargs)
            return value
        return wrapper_repeat

    if _func is None:
        return decorator_repeat
    else:
        return decorator_repeat(_func)
```
