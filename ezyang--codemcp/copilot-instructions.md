## expecttest

> We use expecttest for testing.  If you need to match against a multiline


## expecttest

We use expecttest for testing.  If you need to match against a multiline
string or some output that is likely to change in the future (e.g., the kind
of thing you would have ordinarily done an assertIn with), use an expect test.
If a test is assertExpectedInline and it fails, use 'accept' command to update
the output, do NOT relax the test.  If the output of the test seems
nondeterministic (e.g., you run accept but the test is still failing on the
assertExpectedInline) you should halt and ask for help from the user.

Below is its README.

----

This library implements expect tests (also known as "golden" tests). Expect
tests are a method of writing tests where instead of hard-coding the expected
output of a test, you run the test to get the output, and the test framework
automatically populates the expected output.  If the output of the test changes,
you can rerun the test 'accept' command to accept the output.

Somewhat unusually, this library implements *inline* expect tests: that is to
say, the expected output isn't saved to an external file, it is saved directly
in the Python file (and we modify your Python file when updating the expect
test.)

The general recipe for how to use this is as follows:

  1. Write your test and use `assertExpectedInline()` instead of a normal
     `assertEqual`.  Leave the expected argument blank with an empty string:
     ```py
     self.assertExpectedInline(some_func(), """""")
     ```

  2. Run your test.  It should fail, and you get an error message about
     accepting the output with `EXPECTTEST_ACCEPT=1` (this means to use
     'accept' command)

  3. Rerun the test with 'accept' command.  Now the previously blank string
     literal will contain the expected value of the test.
     ```py
     self.assertExpectedInline(some_func(), """my_value""")
     ```

A minimal working example:

```python
# test.py
import unittest
from expecttest import TestCase

class TestStringMethods(TestCase):
    def test_split(self):
        s = 'hello world'
        self.assertExpectedInline(str(s.split()), """""")

if __name__ == '__main__':
    unittest.main()
```

Run `accept` command, and the content in triple-quoted string will be
automatically updated, so you don't have to fill it out yourself.

---
> Source: [ezyang/codemcp](https://github.com/ezyang/codemcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
