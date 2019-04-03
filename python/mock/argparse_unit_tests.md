Argparse Unit Tests Suppress the Help Message
=============================================

References
----------

from:
[Answer](https://stackoverflow.com/a/18652005/1367852)

[Article](https://stackoverflow.com/questions/18651705/argparse-unit-tests-suppress-the-help-message)

Code
----

A simple context manager to trap the `SystemExit` exception:
```
with self.assertRaises(SystemExit) as cm:
    arg_parse_obj.parse_known_args(['-h'])

self.assertEqual(cm.exception.code, 0)
```

More complex example that also nabs `stdout` and `stderr`:
```
from contextlib import contextmanager
from StringIO import StringIO

@contextmanager
def capture_sys_output():
    capture_out, capture_err = StringIO(), StringIO()
    current_out, current_err = sys.stdout, sys.stderr
    try:
        sys.stdout, sys.stderr = capture_out, capture_err
        yield capture_out, capture_err
    finally:
        sys.stdout, sys.stderr = current_out, current_err
```

Used as:
```
with self.assertRaises(SystemExit) as cm:
    with capture_sys_output() as (stdout, stderr):
        arg_parse_obj.parse_known_args(['-h'])

self.assertEqual(cm.exception.code, 0)

self.assertEqual(stderr.getvalue(), '')
self.assertEqual(stdout.getvalue(), 'Some help value printed')
```

Note that you can use these in a class (you're testing, so... wouldn't you be?) like this:
```
class TestFoo(TestCase):
    
    @staticmethod
    def capture_sys_output():
        ... # this bit is the same

    def test_it(self):
        ...
        with ... as cm:
            with TestFoo.capture_sys_output() as (stdout, stderr):
               ... # this bit is the same
        ... # this bit is the same
```

