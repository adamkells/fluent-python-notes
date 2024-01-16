# cool-python-stuff-i-forget-sometimes
A collection of neat python tips/tricks that I forget about.


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
