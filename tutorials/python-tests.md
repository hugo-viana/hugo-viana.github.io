# Tests

## Basics

Before you start testing your python program you must previously answer:

**What do I want to test?**

This seams like a trick question but in fact it's the most important step of testing.

Generally speaking you'll want to test everything (that makes sense).

Let's see a dummy program and then think about want we should be testing or not:

```{python}
def get_data(raw_data_1: str, raw_data_2: str) -> list:
  """Get raw data from the desired sources.

  Just compiles a list of all data arguments.
  """
  raw_data = [raw_data_1, raw_data_2]
  return raw_data

def transform_data(raw_data: list) -> dict:
  """Transform raw data into final data.

  Counts the number of items in the raw_data list and
  creates a dict with 2 keys:
  - datas_counter (with the len of the list).
  - datas (with another dict inside having each raw_data item
    as a seperate key/value).
  """
  prepared_data = {
    "datas_counter": len(raw_data),
    "datas": {},
  }

  i = 1
  for d in raw_data:
    prepared_data["datas"]["data_"+str(i)] = d
    i += 1

  return prepared_data

def output_data(prepared_data):
  """Output the final data.

  Outputs in a string-form the prepared_data dict.
  """
  counter_str = f"Found datas: {prepared_data['datas_counter']}"
  datas_str = ""
  for key in prepared_data["datas"].keys():
    datas_str += f"Data '{key}': {prepared_data['datas'][key]}\n"
  return f"{datas_str}{counter_str}"

def main():
  """Run all."""
  data_1 = "John"
  data_2 = "Doe"
  raw_data = get_data(data_1,data_2)
  prepared_data = transform_data(raw_data)
  output = output_data(prepared_data)
  return output

print(main())
```

Sorry for the real dummy example but I think it has all the major pieces to start understanding what you should and should not test.

Basically we have 3 separate functions that have a **specific input**, **do something(s)**, and **return an output**. And then we have a fourth function that doesn't have any inputs, **defines 2 variables** and **calls all the previous 3 functions** secuencially, and **prints the result** of the last function.

So... any ideas?

Without knowing anything about python tests, i think you can agree that we must ensure everything works, and probably testing the last function does exactly that, right?

Let's see how we can test the last function (the one that does it all):

1. We know it has 2 variables harcoded:
  ```{python}
  data_1 = "John"
  data_2 = "Doe"
  ```
  since they are harcoded we do not need to test them. a whole different story would be it they were argument passes along as main() arguments. But that's not the case.

2. It call the first function **get_data()** passing along the 2 previous variables as arguments, and saving the result into a new variable **raw_data**:
  ```{python}
  raw_data = get_data(data_1,data_2)
  ```

3. With the result from the previous step it calls the second function **transform_data()** and again, stores the result into a new variable **prepared_data**:
  ```{python}
  prepared_data = transform_data(raw_data)
  ```

4. Then, it calls the third function **output_data()** passing the prepared_data variable and again, stores the result into a new variable **output**:
  ```{python}
  output = output_data(prepared_data)
  ```

5. Finally, it returns the output variable.

Outside of these functions, we can also see that there is a print statement that prints to the screen the result of the main() function.

So, now that we understand what all this functions do, let's see how we can test them.

### Integration tests

In this case, since there are no argumens passed to the function, we know exactly the result of this function:

```{console}
Data 'data_1': John
Data 'data_2': Doe
Found datas: 2
```

It's a string that has 3 lines, so let's build a variable that has reproduces this output:

```{python}
expected_string = "Data 'data_1': John\nData 'data_2': Doe\nFound datas: 2"
```

The **\n** is an escape character to line break strings.

At this point we can test if the **main()** result equals the **expected_string**:

```{python}
resulted_string = main()
assert resulted_string == expected_string
```

The assert returns True so no output is displayed to the screen. Let's add a print so that we can see that it actually is True:

```{python}
# First test:
resulted_string = main()
assert resulted_string == expected_string
print("First test:", resulted_string == expected_string)
```

output:

```{console}
First test: True
```

What we've just done is considered an **integration test**, because it ensures all the peaces work as expected (as a hole).

If we wanted to test all the peaces seperatelly (and yes... we do want that), we would have to add some **unit-tests**.

### Unit-Tests

Before entering the unit-test, let's see why we need to unit-test all this parts seperatelly.

We have ensured that our program will run perfectly and that it returns the exact string we were looking for but...

- what if it didn't? how would you find where the error was?
- what if you want to use one of the first 3 functions to build another feature? would you be fine with having your 2 only tests (integration-tests) failing and not knowing where the real problem lies?

With uni-tests you can test separate pieces of your code and ensure every one of them works separately. Then, with an integration test you can make sure they all work together.

Let's see how we can unit-test this functions seperately, and how many unit-tests we need:

1. Ensure **get_data()** works as expected
2. Ensure **transform_data()** works as expected
3. Ensure **output_data()** works as expected
4. Ensure **main()** works as expected


TODO 3 unit-tests


At this moment i think we can all agree to this first unit-tests, but what about the fourth one **main() unit-test**... isn't it the same as our integration test?

When we're unit-testing, we must always have in mind what are we really testing.

The integration test we just saw, ensures the correct output but it doesn't ensures that all that we're actually doing is everything we want in the main() function to do.

Before, we've identified 5 sentences in the main function, and since we want to unit-test the main function, the result should not be our primary focus. Instead we must ensure all steps are made correctly.

Let's look at the second step:

```{python}
raw_data = get_data(data_1,data_2)
```

We need to ensure get_data() is called only once, and that we'll pass 2 arguments (data_1, data_2). But we don't need to ensure that the result given by get_data() is correct. For that we already have a separate unit-test of get_data().

So... how do we acomplish this?

For this kind of tests, we'll use the Mock library, that enables us to impersonate a python object and have control over it during the test execution. Let's see this particular example:

```{python}
from unittest import mock

with mock.patch("test.get_data", autospec=True) as mocked_get_data:
  expected_string = main()

mocked_get_data.assert_called_once_with("John","Doe")
print("Unit-test (main):", mocked_get_data.assert_called_once_with("John","Doe"))
```

output:

```{console}
Unit-test (main): None
```

Again, we included a print statement just to be sure the mock assert returns None in case of passing the test.

Another way to acomplish this is by puting our assert inside a function an use the patch decorator:

```{python}
from unittest import mock

@mock.patch("test.get_data", autospec=True)
def unittest_main(mocked_get_data):
  expected_string = main()
  mocked_get_data.assert_called_once_with("John","Doe")
  print("Unit-test (main) 2:", mocked_get_data.assert_called_once_with("John","Doe"))

unittest_main()
```

output:

```{console}
Unit-test (main) 2: None
```

This second aproach will help use a lot since for this specific function (main) we'll have to mock several called functions. That said, lets mock the next call (transform_data), but before we do it, we must understand that this second call will depend on the resulting data from the first call. An since we mocked it, the result for get_data() is now None.

Since we need the transform_data() argument to be a list, let's first tell mock to return a list with at least one item when calling the mocked get_data():

```{python}
from unittest import mock

@mock.patch("test.get_data", autospec=True, return_value=["a"])
def unittest_main(mocked_get_data):
  main()
  mocked_get_data.assert_called_once_with("John","Doe")
  print("Unit-test (main) 2:", mocked_get_data.assert_called_once_with("John","Doe"))

unittest_main()
```

This won't change the test, but it will help us to make the next mocker:

```{python}
from unittest import mock

@mock.patch("test.transform_data", autospec=True)
@mock.patch("test.get_data", autospec=True, return_value = ["a"])
def unittest_main(mocked_get_data, mocked_transform_data):

  main()

  mocked_get_data.assert_called_once_with("John","Doe")
  print("Unit-test (main) 3a:", mocked_get_data.assert_called_once_with("John","Doe"))

  mocked_transform_data.assert_called_once_with(["a"])
  print("Unit-test (main) 3b:", mocked_transform_data.assert_called_once_with(["a"]))

unittest_main()
```

output:

```{console}
Unit-test (main) 3a: None
Unit-test (main) 3b: None
```

The same logic applies to the third call (output_data). We'll asign a dummy dict as the return value and mock the output_data function:

```{python}
from unittest import mock

@mock.patch("test.output_data", autospec=True)
@mock.patch("test.transform_data", autospec=True, return_value = {"x": "y"})
@mock.patch("test.get_data", autospec=True, return_value = ["a"])
def unittest_main(mocked_get_data, mocked_transform_data, mocked_output_data):

  main()

  mocked_get_data.assert_called_once_with("John","Doe")
  print("Unit-test (main) 4a:", mocked_get_data.assert_called_once_with("John","Doe"))

  mocked_transform_data.assert_called_once_with(["a"])
  print("Unit-test (main) 4b:", mocked_transform_data.assert_called_once_with(["a"]))

  mocked_output_data.assert_called_once_with({"x": "y"})
  print("Unit-test (main) 4c:", mocked_output_data.assert_called_once_with({"x": "y"}))

unittest_main()
```

output:

```{console}
Unit-test (main) 4a: None
Unit-test (main) 4b: None
Unit-test (main) 4c: None
```

The only thing missing here is to ensure the main function returns exactly the same as the output_data, so let's give our mocked output_data some string return value and ensure main() returns that exact string:

```{python}
from unittest import mock

@mock.patch("test.output_data", autospec=True, return_value = "dummy string")
@mock.patch("test.transform_data", autospec=True, return_value = {"x": "y"})
@mock.patch("test.get_data", autospec=True, return_value = ["a"])
def unittest_main(mocked_get_data, mocked_transform_data, mocked_output_data):

  result_string = main()

  mocked_get_data.assert_called_once_with("John","Doe")
  print("Unit-test (main) 5a:", mocked_get_data.assert_called_once_with("John","Doe"))

  mocked_transform_data.assert_called_once_with(["a"])
  print("Unit-test (main) 5b:", mocked_transform_data.assert_called_once_with(["a"]))

  mocked_output_data.assert_called_once_with({"x": "y"})
  print("Unit-test (main) 5c:", mocked_output_data.assert_called_once_with({"x": "y"}))

  assert result_string == "dummy string"
  print("Unit-test (main) 5d:", result_string == "dummy string")

unittest_main()
```

output:

```{console}
Unit-test (main) 5a: None
Unit-test (main) 5b: None
Unit-test (main) 5c: None
Unit-test (main) 5d: True
```