# cool-python-stuff-i-forget-sometimes
A collection of neat python tips/tricks that I forget about.

## Table of Contents
[Generator Expressions](#generator-expressions)
[Named Tuples](#named-tuples)
[Sorting lists/Bisect](#sorting-listsbisect)
[Deques](#deques)
[Lambdas](#lambdas)
[Callables](#callables)
[Itemgetter](#itemgetter)
[Functools.partial](#functoolspartial)

## Generator Expressions

Generator expressions are a nice way of avoiding list comprehensions. These are especially useful when you only want to iterate over the list once and do not need to save it in memory.

```python
# This is a list comprehension
squares_list = [x**2 for x in range(1000)]

# This achieves the same with a generator
squares_generator = (x**2 for x in range(1000))
```

## Named Tuples

The `namedtuple` function from the `collections` library is a subclass of tuple that also adds field names and a class name. This makes using Tuples as records much easier. They are also very memory efficient. They take up the same space as a regular tuple since the field information is stored in the class.

```python
from collections import namedtuple

# Define a named tuple for a weather data point
WeatherData = namedtuple('WeatherData', ['temperature', 'humidity', 'pressure'])

# Example usage
weather_data_point_1 = WeatherData(25.5, 60, 1015)
```

## Sorting lists/Bisect
Sorting lists can be slow if the list is long, so once you have a sorted list you'd really rather not do it again. So what if you have one new element that needs to be added to the list. One option would be to append the element and redo the sort but this quickly becomes ineffecient if we have to repeat this many times. Bisect offers a handy way of inserting a single element to an already sorted list.

```python
import bisect

# Sorted list of data
sorted_data = [10, 20, 30, 40, 50]

# Adding a new element
new_value = 35
index = bisect.bisect_left(sorted_data, value)
sorted_data.insert(index, value)

```

## Deques
The append and pop methods mean that lists can be used as stacks, however this can be slow since removing from the left of the list requires the entire list to be shifted. `deque` (double-ended queue) offers a convenient solution to this. It is very efficient for removing elements from the end of the deque or from shifting the elements. Note however that it is slow if we need to work with the data in the middle of the deque.


In the following example, we can use deque for conveniently computing a moving sum of some data.
```python
from collections import deque

def moving_sum(data, k):
    result = []
    window = deque(maxlen=k)  # Use deque to maintain the sliding window

    for i, value in enumerate(data):
        window.append(value)  # append a new value to the deque (old values will be removed to maintain max length)
        if len(window)==k:
            result.append(sum(window))

    return result

# Example usage
data_stream = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
window_size = 3

result = moving_average(data_stream, window_size)
```

## Lambdas

I subscribe to following Frank Lundh's approach to lambdas:
1. Write a lambda function.
2. Write a comment explaining what the heck that lambda does.
3. Study the comment for a while, and think of a name that captures the essence of the comment.
4. Convert the lambda to a def statement, using that name.
5. Remove the comment.

## Callables

Any object can be made to behave like a function. All that is required is to implement the __call__ instance method.

For example, we might make a random_data class:

```python
class RandomDataGenerator:
    def __init__(self, data):
        self.data = data

    def __call__(self):
        return random.choice(self.data)

# Example usage
data_generator = RandomDataGenerator([1, 2, 3, 4, 5])
random_data_point = data_generator()
```

## ItemGetter

The standard package `operator` includes a handy function called `itemgetter` which can be used to pick out items from lists/tuples.

As an example, consider sorting the list of tuples by the country code at index 1.
```python
metro_data = [('Tokyo', 'JP', 36.933, (35.689722, 139.691667)),
              ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),
              ('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),
              ('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),
              ('Sao Paulo', 'BR', 19.649, (-23.547778, -46.635833)),
             ]

from operator import itemgetter
sorted_cities = sorted(metro_data, key=itemgetter(1))
```

Similarly `attrgetter` can be used with namedtuples to get named items from the object.

## Functools.partial

Functools has a useful function called `partial` which can freeze some of the inputs of a function.

As an example:

```python
from functools import partial

# Original function
def exponential_growth(base, exponent, time):
    return base * (1 + exponent) ** time


# Create a partial function with fixed base and exponent
fixed_base_exp_growth = partial(exponential_growth, base=2, exponent=0.1)

# Use the partial function in a data processing pipeline
time_values = [0, 1, 2, 3, 4, 5]
result = [fixed_base_exp_growth(t) for t in time_values]
```

This is useful when we expect to use a function in a very particular way in our code.

## Decorators

Decorators are a super useful syntax for enhancing functions. There's two things to remember:
1. They have the power to replace the decorated function with a new one.
2. They are executed immediately when a module is loaded.

As an example, the following code snippets are completely equivalent:
```python
@decorate
def target():
    print('running target()')
```

```python
def target():
    print('running target()')

target = decorate(target)
```

### Decorators as registers
One neat use case is to use a decorator to register a function:
```python

promos = []

def promotion(promo_func):
    promos.append(promo_func)
    return promo_func

@promotion
def bulk_item(order):
    """10% discount for each LineItem with 20 or more units"""
    discount = 0
    for item in order.cart:
        if item.quantity >= 20:
            discount += item.total() * .1
    return discount
```

### functools.singledispatch

`singledispatch` allows you to write a single function with multiple implmentations. For example, let's say we want to have a function which will square it's input. Typically if we want to handle different data types (list, int etc.) then we need to have a long list of if-else statements. `singledispatch` allows us to write custom functions based on the type.

```python
from functools import singledispatch

@singledispatch
def square_elements(data):
    raise NotImplementedError("Unsupported data type")

@square_elements.register(list)
def _(data):
    return [item ** 2 for item in data]

@square_elements.register(dict)
def _(data):
    return {key: value ** 2 for key, value in data.items()}

@square_elements.register(int)
@square_elements.register(float)
def _(data):
    return data ** 2
```

### Parameterizing decorators

We can add parameters to decorators by wrapping them in an additional layer.

```python
registry = set()
def register(active=True):
    def decorate(func):
        print('running register(active=%s)->decorate(%s)'
        % (active, func))
        if active:
            registry.add(func)
        else:
            registry.discard(func)
        return func
    return decorate

@register(active=False)
def f1():
    print('running f1()')
```

### Decorators as classes

We can also write decorators as classes rather than function wrappers by creating `__init__` and `__call__` methods.

```python
class function_wrapper(object):
    def __init__(self, wrapped):
        self.wrapped = wrapped
    def __call__(self, *args, **kwargs):
        return self.wrapped(*args, **kwargs)

@function_wrapper
def function():
    pass
```

## Making my Classes immutable

Sometimes when I make a class object with properties, I don't want to be able to accidentally update any of the values.

I can do this by using double underscores to make attributes private and instead access via class methods.

```python
class MyClass:
    def __init__(self, x):
        self.__x = x
    @property
    def x(self):
        return self.__x
```

So now I can't make changes directly to the x attribute since `x` is actually a function which just returns the value of the private property.