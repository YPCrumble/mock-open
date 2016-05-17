mock-open
=========

[![GitHub release](https://img.shields.io/github/release/nivbend/mock-open.svg)](https://github.com/nivbend/mock-open/releases/latest)

A better mock for file I/O.

This project was started as part of a [different one](github.com/nivbend/bdd_bot) where I needed
a better mock for the builtin `open`.

Install
-------

```
$ pip install mock-open

```
Quickstart
----------

```
# file_to_test.py

def count_lines_in_file_and_convert_to_array():
    with open("path_to_file.json", "r") as file:
        output = []
        num_lines = sum(1 for line in file)
        file.seek(0)
        for line in file.readlines():
            output.append(line.strip("\n"))
    return num_lines, output
```

```
# test.py

import mock
from mock_open import MockOpen
import unittest

from file_to_test import count_lines_in_file_and_convert_to_array


class MockOpenTest(unittest.TestCase):
    test_input = ["First line of file", "Second line of file"]

    @mock.patch("file_to_test.open", new=MockOpen(read_data="\n".join(test_input)))
    def test_count_lines_in_file_and_convert_to_array(self):
        num_lines, test_output = count_lines_in_file_and_convert_to_array()
        self.assertEqual(num_lines, 2)
        self.assertEqual(test_output, self.test_input)

if __name__=="__main__":
    unittest.main()
```

class `MockOpen`
--------------

The `MockOpen` class should work as a stand-in replacement for `mock.mock_open` with some
added features:
* Multiple file support, including a mapping-like access to file mocks by path:

  ```python
  mock_open = MockOpen()
  mock_open["/path/to/file"].read_data = "Data from a fake file-like object"
  mock_open["/path/to/bad_file"].side_effect = IOError()
  ```

  You can also configure behavior via the regular `mock` library API:

  ```python
  mock_open = MockOpen()
  mock_open.return_value.write.side_effect = IOError()
  ```

* Persistent file contents between calls to `open`:

  ```python
  with patch("builtins.open", MockOpen()):
      with open("/path/to/file", "w") as handle:
          handle.write("Some text")

      with open("/path/to/file", "r") as handle:
          assert "Some text" == handle.read()
  ```

* All the regular file operations: `read`, `readline`, `readlines`, `write`, `writelines`, `seek`, `tell`.
