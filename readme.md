> On a UNIX system, everything is a file; if something is not a file, it
> is a process.

# What could go wrong?

Quite a lot, actually.

For a full list of the built-in exceptions, see
<https://docs.python.org/3/library/exceptions.html>.

Common exceptions you may want to make use of include:

- `TypeError` - a value is of the wrong type
- `ValueError` - a value is outside expected boundaries
- `IndexError` - the given index is out of range for the sequence
- `KeyError` - the given key is not in the mapping
- `EOFError` - The end of the file or stream was encountered without
  obtaining data
- `ZeroDivisionError`
- `FileNotFoundError`
- `PermissionError`
- `FileExistsError`
- `IsADirectoryError`
- `IsNotADirectoryError`

# Raising Exceptions

Often we need to be able to signal from one function to another than
something has gone wrong. For this we use the `raise` keyword with the
appropriate exception type.

``` python
raise Exception
```

In this case, `Exception` is the constructor for an `Exception` object.
You can pass arguments to this constructor to give more information
about why the exception has been raised.

``` python
raise ValueError("min() argument is empty")
```

# Handling Exceptions

The full syntax for handling exceptions is as follows:

``` python
try:
    # some code that may raise an exception
except Exception:
    # some code that handles the given exception
except ZeroDivisionError:
    # you can have multiple `except` blocks for different exceptions
except (IndexError, KeyError):
    # you can specify a tuple of errors for an `except` block
except ValueError as ve:
    # you can assign a name to the exception for use in the `except` block
else:
    # some code that will run if there are no exceptions
finally:
    # some code that will run regardless
    # this block is typically used to clean up after whatever was tried
```

You do not need all of these blocks in any given statement. At minimum,
you only need a `try` and either an `except` or `finally` block.

# Best Practices

- Try to use the most relevant or specific exceptions both when raising
  them and when handling them.
- Many exceptions exist which indicate a bug in your code. Rather than
  handling these exceptions, you should try to find the bug and fix it.
- Exceptions are best used for reacting to things external to the piece
  of code you are working on. Things like validating input or errors or
  events arising from systems such as files and network resources.

# Context Managers

There is a much better way to handle cleaning up resources like files
and network connections than `finally`: the **context manager** used
with the `with` compound statement.

``` python
with open('somefile.txt', 'r') as txtfile:
    # some code dealing with the file
```

When you use `with open... as...:` to open a file, a context manager
handles the set up and clean up of the file for you.

To catch exceptions while working with files, you will have to wrap the
context manager statement in a try/except statement. This can happen
immediately or at a higher \"level\" in your code.

``` python
try:
    with open('somefile.txt', 'r') as txtfile:
        # some code dealing with the file
except (FileNotFoundError, PermissionError):
    # some code to be run when the file is not accessible
```

# Basic File I/O

## Text Mode

You can read and write strings directly to/from a `File` object created
with the `open` function.

``` python
with open(filename, 'r') as textfile: # Open file for reading
    entire_contents: str = textfile.read()
    # or... some_content: str = textfile.read(4)

with open(filename, 'r') as textfile:
    single_line: str = textfile.readline()

with open(filename, 'r') as textfile:
    lines: list[str] = textfile.readlines()

with open(filename, 'r') as textfile:
    for line in textfile:
        print(line, end='')

with open(filename, 'w') as textfile: # Open file for writing
    textfile.write(single_line)

with open(filename, 'x') as textfile: # Creates a file and opens it for writing
    textfile.writelines(lines)

with open(filename, 'w') as textfile:
    for line in lines:
        print(line, file=f)
```

Where `filename` is a variable with a string describing a file path, or
some other \"path-like\" object.

## Pickle

The `pickle` module in the Python Standard Library allows you to store
nearly any Python object as a byte-string, i.e. a string of binary. The
binary format used is specific to Python and likely cannot be read
directly using other programming languages.

``` python
with open(filename, 'wb') as datafile:
    pickle.dump(data, datafile)

with open(filename, 'rb') as datafile:
    data = pickle.load(datafile)

binary = pickle.dumps(data)

data = pickle.loads(binary)

```

Note that you need to open files in *binary* mode to use them with
`pickle`.

The `pickle` module has a couple of exceptions of it\'s own,
`pickle.PicklingError` and `pickle.UnpicklingError` that will be raised
if there is a problem pickling or unpickling an object, such as data
corruption. Other exceptions may also be raised during (un)pickling,
including `AttributeError`, `EOFError`, `ImportError`, and `IndexError`.

With the great power of `pickle` to handle most Python objects comes the
great responsibility to only unpickle from trusted sources. Since
objects contain both data and functions, loading an object from an
untrusted source may run malicious code.

## JSON

JSON is a good alternative to `pickle` when you don\'t need the power to
preserve arbitrary Python objects or when you need to store information
that needs to be generally accessible to other programs and human
beings. JSON can store most of the simple Python data types that we\'ve
seen and does so in a format that is virtually identical to Python
source code. Python\'s `json` library has a very similar interface to
`pickle`, but works in text mode, since we\'re storing a text
representation of the data. The `json` module will automatically convert
data to and from its string representation for you.

## CSV

Plain-text files consisting of values separated by some delimiter is an
exceeding common format for datasets. While we call these \"csv\" files
for *Comma Separated Values*, the delimiter may be any number of other
characters, including whitespace characters such as tabs. Python\'s
`csv` module can automatically handle most of the plethera of different
variants of csv file and provides a few different ways to access that
data, but will not automatically convert data from their string
representation unless asked using `QUOTE_NONNUMERIC` to convert unquoted
numeric data to floats.

To use the `csv` module, we need to open files in text mode with
`newline=''`, i.e. \"universal newline mode\".

The basic usage of `csv` is as follows:

``` python
with open(filename, 'r', newline='') as csvfile:
    data = list(csv.reader(csvfile))

with open(filename, 'w', newline='') as csvfile:
    writer = csv.writer(csvfile)
    writer.writerows(data)
```

There\'s also a dictionary version of the `reader` and `writer` that may
make processing the spreadsheet easier:

``` python
with open(filename, 'r', newline='') as csvfile:
    data: list[dict[str, str | float] = list(csv.DictReader(csvfile))

with open(filename, 'w', newline='') as csvfile:
    fieldnames: list[str] = [...]
    writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
    writer.writeheader()
    writer.writerows(data)      
```

The `csv` module has a lot of options to deal with various \"dialects\"
of csv file. If you are having trouble with a specific sheet, it is
worth checking out the documentation
<https://docs.python.org/3/library/csv.html>.
