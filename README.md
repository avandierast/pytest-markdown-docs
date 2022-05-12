# Pytest Markdown Docs

A plugin for [pytest](https://docs.pytest.org) that uses markdown code snippets from markdown files and docstrings as tests.

Detects Python code fences (triple backtick escaped blocks) in markdown files as
well as inline Python docstrings (similar to doctests) and runs them as tests.

Python file example:

````python
# mymodule.py
class Foo:
    def bar(self):
        """Bar the foo

        This is a sample docstring for the bar method

        Usage:
        ```python
        import mymodule
        result = mymodule.Foo().bar()
        assert result == "hello"
        ```
        """
        return "hello"
````

Markdown file examples:

````markdown
# Title

Lorem ipsum yada yada yada

```python
import mymodule
result = mymodule.Foo().bar()
assert result == "hello"
```
````

## Usage

First, make sure to install the plugin, e.g. `pip install pytest-markdown-docs`

To enable markdown python tests, pass the `--markdown-docs` flag to `pytest`:
```shell
pytest --markdown-docs
```

You can also use the `markdown-docs` flag to filter *only* markdown-docs tests:
```shell
pytest --markdown-docs -m markdown-docs
```

### Detection conditions

Fence blocks (` ``` `) starting with the `python`, `python3` or `py` language definitions are detected as tests in:

* Python (.py) files, within docstrings of classes and functions
* `.md`, `.mdx` and `.svx` files

## Skipping tests

To exclude a Python code fence from testing, add a `notest` info string to the
code fence, e.g:

````markdown
```python notest
print("this will not be run")
```
````

## Code block dependencies

Sometimes you might wish to run code blocks that depend on entities to already
be declared in the scope of the code, without explicitly declaring them. There
are currently two ways you can do this with pytest-markdown:

### Injecting global/local variables

If you have some common imports or other common variables that you want to make
use of in snippets, you can add them by creating a `pytest_markdown_docs_globals`
hook in your `conftest.py`:

```python
def pytest_markdown_docs_globals():
    import math
    return {"math": math, "myvar": "hello"}
```

With this conftest, you would be able to run the following markdown snippet as a
test, without causing an error:

````markdown
```python
print(myvar, math.pi)
```
````

### Fixtures

You can use both `autouse=True` pytest fixtures in a conftest.py or named fixtures with
your markdown tests. To specify named fixtures, add `fixture:<name>` markers to the code
fence info string, e.g.,

````markdown
```python fixture:capsys
print("hello")
captured = capsys.readouterr()
assert captured.out == "hello\n"
```
````
As you can see above, the fixture value will be injected as a global. For `autouse=True` fixtures, the value is only injected as a global if it's explicitly added using a `fixture:<name>` marker.


### Depending on previous snippets

If you have multiple snippets following each other and want to keep the side
effects from the previous snippets, you can do so by adding the `continuation`
info string to your code fence:

````markdown
```python
a = "hello"
```

```python continuation
assert a + " world" == "hello world"
```
````

## Testing of this plugin
You can test this module itself (sadly not using markdown tests at the moment) using pytest:

```shell
> poetry run pytest
```

Or for fun, you can use this plugin to include testing of the validity of snippets in this README.md file:
```shell
> poetry run pytest --markdown-docs
```


## Known issues
* Only tested with pytest 6.2.5. There seem to be some minor issue with pytest >7 due to changes of some internal functions in pytest, but that should be relatively easy to fix. Contributions are welcome :)
* Code for docstring-inlined test discovery can probably be done better (similar to how doctest does it). Currently, seems to sometimes traverse into Python's standard library which isn't great...