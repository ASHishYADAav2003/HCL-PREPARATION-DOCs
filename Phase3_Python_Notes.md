# Phase 3: Python (15 lessons)

> HCLTech Cloud Data Engineer prep. Built from the syllabus (Programming Skills → Python & PySpark).
> Read top to bottom once. Each lesson: **concept → code → line-by-line explanation → what to remember**.
> Every code block is followed by an **Explanation** so nothing is left unexplained.

| # | Lesson | Syllabus module |
|---|--------|-----------------|
| 1 | Introduction to Python | Introduction to Python |
| 2 | Python Essentials | Python Essentials |
| 3 | Control Flow | Control Flow |
| 4 | Functions | Functions and Modules |
| 5 | Modules and Scope | Functions and Modules |
| 6 | Lists and Strings | Data Structures |
| 7 | Tuples, Dicts, Sets | Data Structures |
| 8 | File Handling | File Handling |
| 9 | OOP | Object-Oriented Programming |
| 10 | NumPy and Pandas | Python Libraries |
| 11 | Utility Libraries | Python Libraries |
| 12 | Error Handling and Debugging | Error Handling and Debugging |
| 13 | Unit Testing and Logging | Unit Testing / Logging |
| 14 | Advanced Topics | Advanced Topics (functional) |
| 15 | Data Integration, Environments, Copilot | Advanced Topics + GitHub Copilot |

---

## Lesson 1: Introduction to Python

**What is Python?** A high-level, interpreted, dynamically typed language. Easy to read, huge library ecosystem. In data engineering it is used for ETL scripts, automation, API calls, data cleaning (Pandas) and big data (PySpark).

- **High-level:** you write human-like code; you don't manage memory or hardware.
- **Interpreted:** code runs line by line through the Python interpreter (no separate compile step you manage).
- **Dynamically typed:** you don't declare types; Python decides the type at runtime.

**Setup**
- Install Python from python.org, check with `python --version`.
- Editors: **IDLE** (comes with Python, basic), **VS Code** (recommended, install the Python extension).

**Two ways to run code**

| | REPL (interactive) | Script |
|---|---|---|
| How | type `python`, then code line by line | write `file.py`, run `python file.py` |
| Use | quick tests, learning | real programs, pipelines |
| Saved? | No | Yes |

REPL = **R**ead, **E**valuate, **P**rint, **L**oop. You type a line, Python reads it, runs it, prints the result, and waits for the next line.

```python
# hello.py
print("Hello, Data Engineer!")
```

**Explanation**
- `# hello.py`: a **comment**. Anything after `#` is ignored by Python. Here it just labels the file name.
- `print(...)`: a built-in function that shows output on the screen.
- `"Hello, Data Engineer!"`: a string (text) in quotes. This is the value given to `print`.
- Save this as `hello.py` and run `python hello.py`; the screen shows `Hello, Data Engineer!`.

**Syntax and indentation:** Python uses **indentation (4 spaces)** instead of `{ }` to define blocks. Wrong indentation causes `IndentationError`.

```python
if True:
    print("indented = inside the block")
```

**Explanation**
- `if True:` starts a conditional. `True` is always true, so the block always runs. The colon `:` says "a block follows".
- The next line is indented by 4 spaces. That indentation is what tells Python the `print` belongs **inside** the `if`.
- If you removed the spaces, Python would raise `IndentationError: expected an indented block`.

**Remember:** interpreted, dynamically typed, indentation matters, REPL vs script.

---

## Lesson 2: Python Essentials

### Variables and data types
A variable is a name pointing to a value. No type declaration needed.

```python
name = "Ash"        # str
age = 22            # int
price = 99.5        # float
is_ok = True        # bool
nothing = None      # NoneType
```

**Explanation**
- `name = "Ash"`: creates a variable `name` that points to the text `"Ash"`. Text values are type `str` (string).
- `age = 22`: whole number, type `int`.
- `price = 99.5`: number with a decimal point, type `float`.
- `is_ok = True`: a boolean. Only two values exist: `True` and `False` (capital first letter is required).
- `nothing = None`: `None` means "no value / empty". Its type is `NoneType`. It is used when a value is missing or not set yet.
- The text after `#` on each line is a comment showing the type. You can check any type with `type(age)`.

| Type | Example | Mutable? |
|------|---------|----------|
| int, float, bool, str | `5`, `2.5`, `True`, `"hi"` | No |
| list | `[1,2]` | Yes |
| tuple | `(1,2)` | No |
| dict | `{"a":1}` | Yes |
| set | `{1,2}` | Yes |

(Mutable = can be changed after creation. Explained fully in Lesson 7.)

### Input and output
```python
name = input("Enter name: ")      # input() ALWAYS returns a string
print("Hi", name, sep="-", end="!\n")
print(f"Hello {name}")             # f-string (best way to format)
```

**Explanation**
- Line 1: `input("Enter name: ")` shows the message `Enter name: `, waits for the user to type and press Enter, then returns what they typed **as a string**. It is stored in `name`. Even if the user types `25`, you get `"25"` (text), not the number 25.
- Line 2: `print("Hi", name, sep="-", end="!\n")` prints several values.
  - `sep="-"`: the separator placed **between** the items (default is a space). So the output is `Hi-Ash`.
  - `end="!\n"`: what is printed **after** everything (default is a newline `\n`). Here it prints `!` and then a newline. Output: `Hi-Ash!`
- Line 3: `f"Hello {name}"` is an **f-string**. The `f` before the quote lets you put variables inside `{ }`. If `name` is `Ash`, it prints `Hello Ash`. This is the cleanest and most common way to build text.

### Operators
| Type | Operators |
|------|-----------|
| Arithmetic | `+ - * / // % **` (`//` floor division, `**` power) |
| Comparison | `== != > < >= <=` |
| Logical | `and or not` |
| Assignment | `= += -= *= /=` |

Quick examples to understand each:
```python
7 + 2     # 9      addition
7 / 2     # 3.5    normal division (always float)
7 // 2    # 3      floor division (drops the decimal part)
7 % 2     # 1      modulus (remainder)
2 ** 3    # 8      power (2 to the power 3)

5 == 5    # True   equal?
5 != 3    # True   not equal?
(5 > 3) and (2 > 1)   # True  both must be true
not True              # False flips the value

x = 10
x += 5    # same as x = x + 5  -> 15
```

**Explanation**
- `7 / 2` gives `3.5` because `/` always returns a float. `7 // 2` gives `3` because `//` keeps only the whole part.
- `%` gives the remainder; it is used to check even/odd (`n % 2 == 0` means even).
- `and` needs **both** sides true; `or` needs **at least one** true; `not` reverses.
- `x += 5` is a shortcut: take `x`, add 5, store it back in `x`. Same pattern for `-=`, `*=`, `/=`.

### Comments and readability
```python
# single line comment
"""multi-line string, often used as docstring"""
```

**Explanation**
- `#` starts a one-line comment. Python skips it.
- Triple quotes `""" ... """` create a multi-line string. When placed directly under a function or class it is a **docstring** (documentation). Python has no true multi-line comment, so people use this style for long notes.
- Use meaningful names (`total_sales`, not `ts`), follow PEP 8 (snake_case for variables).

### Type casting
Converting one type to another.
```python
int("10")      # 10
float("3.5")   # 3.5
str(100)       # "100"
list("abc")    # ['a','b','c']
```

**Explanation**
- `int("10")`: converts the text `"10"` into the number `10`.
- `float("3.5")`: converts text to a decimal number.
- `str(100)`: converts the number to text `"100"`. Needed when joining numbers with text using `+`.
- `list("abc")`: a string is a sequence of characters, so this makes the list `['a','b','c']`.
- `int("abc")` raises `ValueError` because `"abc"` is not a number.

### `is` vs `==` (frequent interview question)
- `==` compares **values** (are they equal?)
- `is` compares **identity** (same object in memory?)

```python
a = [1, 2]; b = [1, 2]
a == b   # True  (same values)
a is b   # False (different objects)
```

**Explanation**
- Line 1 creates two **separate** lists that happen to contain the same items. The `;` lets you put two statements on one line.
- `a == b` is `True` because the contents are equal.
- `a is b` is `False` because `a` and `b` are two different objects stored at two different memory locations. (`id(a)` and `id(b)` would be different.)
- Use `is` mainly for `None`: `if x is None`.

---

## Lesson 3: Control Flow

### Conditionals
```python
marks = 72
if marks >= 90:
    print("A")
elif marks >= 60:
    print("B")
else:
    print("C")
```

**Explanation**
- `marks = 72` stores the value.
- Python checks conditions **top to bottom** and runs the **first** one that is true, then skips the rest.
- `if marks >= 90`: 72 is not ≥ 90, so skip.
- `elif marks >= 60`: ("else if") 72 ≥ 60 is true, so it prints `B` and stops checking.
- `else`: runs only if none of the above were true. It has no condition.
- Output: `B`.

### Loops
```python
for i in range(3):          # 0,1,2
    print(i)

n = 3
while n > 0:
    print(n)
    n -= 1
```

**Explanation**
- **for loop:** `range(3)` produces 0, 1, 2. The variable `i` takes each value one by one, and the indented `print(i)` runs each time. Output: 0, 1, 2.
- `range(start, stop, step)`: stop is **exclusive** (not included). `range(1, 10, 2)` gives 1,3,5,7,9.
- **while loop:** it repeats **as long as** the condition is true.
  - `n = 3` is the starting value.
  - `while n > 0:` checks the condition each round.
  - `print(n)` prints 3, then 2, then 1.
  - `n -= 1` reduces `n` by 1 each round. Without this line the loop would never end (infinite loop). When `n` becomes 0, `0 > 0` is false and the loop stops.

### Loop control
| Keyword | Meaning |
|---------|---------|
| `break` | exit loop completely |
| `continue` | skip to next iteration |
| `pass` | do nothing (placeholder) |

```python
for i in range(5):
    if i == 2:
        continue      # skip 2
    if i == 4:
        break         # stop the loop at 4
    print(i)          # prints 0, 1, 3
```

**Explanation**
- When `i` is 0 and 1, neither `if` is true, so they are printed.
- When `i == 2`, `continue` jumps straight to the next round, so 2 is **not** printed.
- `i` = 3 is printed.
- When `i == 4`, `break` ends the whole loop. 4 is not printed.
- `pass` is used where Python needs a statement but you want nothing to happen, e.g. `if x: pass`.

### match-case (structural pattern matching, Python 3.10+)
Like switch-case but more powerful.
```python
status = 404
match status:
    case 200:
        print("OK")
    case 404 | 410:
        print("Not found")
    case _:
        print("Other")      # _ is default
```

**Explanation**
- `match status:` takes the value of `status` and compares it against each `case`.
- `case 200:` runs if `status` is 200.
- `case 404 | 410:` the `|` means **or**; it matches 404 or 410. Here status is 404, so it prints `Not found`.
- `case _:` the underscore is a wildcard that matches anything not matched earlier (the default case).
- Only the first matching case runs.

### Nested loops and conditions
A loop inside a loop (e.g. rows and columns). Total runs = outer × inner.
```python
for row in range(2):
    for col in range(3):
        print(row, col)
```

**Explanation**
- The outer loop runs 2 times (row = 0, 1).
- For **each** row, the inner loop runs 3 times (col = 0, 1, 2).
- So `print` runs 2 × 3 = 6 times: (0,0) (0,1) (0,2) (1,0) (1,1) (1,2).

### `enumerate()` and `zip()`
```python
names = ["A", "B", "C"]
for i, n in enumerate(names, start=1):   # index + value
    print(i, n)

ages = [20, 21, 22]
for n, a in zip(names, ages):            # pairs two lists
    print(n, a)
```

**Explanation**
- `names` is a list of three items.
- `enumerate(names, start=1)` gives pairs of (counter, item). `start=1` makes the counter begin at 1 instead of 0. Each loop `i` is the number and `n` is the name. Output: `1 A`, `2 B`, `3 C`. This avoids manually keeping a counter.
- `ages` is a second list.
- `zip(names, ages)` walks both lists **together**, giving `("A",20)`, `("B",21)`, `("C",22)`. The loop unpacks each pair into `n` and `a`. Output: `A 20`, `B 21`, `C 22`.
- `zip` stops at the **shortest** list.

---

## Lesson 4: Functions

A reusable block of code. Write once, call many times.
```python
def add(a, b):
    return a + b

add(2, 3)    # 5
```

**Explanation**
- `def` defines a function. `add` is its name; `a, b` are **parameters** (inputs).
- The indented line is the function body. `return a + b` sends the sum back to whoever called the function.
- `add(2, 3)` **calls** the function with **arguments** 2 and 3. `a` becomes 2, `b` becomes 3, and the result `5` is returned.
- A function with no `return` gives back `None`.

### Argument types
```python
def greet(name, msg="Hello"):     # default argument
    return f"{msg}, {name}"

greet("Ash")                       # positional
greet(msg="Hi", name="Ash")        # keyword arguments
```

**Explanation**
- `msg="Hello"` is a **default argument**: if the caller does not give `msg`, `"Hello"` is used. Default parameters must come **after** non-default ones.
- `greet("Ash")`: one **positional** argument (matched by position to `name`). `msg` uses its default. Result: `"Hello, Ash"`.
- `greet(msg="Hi", name="Ash")`: **keyword arguments** (matched by name), so order does not matter. Result: `"Hi, Ash"`.

### `*args` and `**kwargs`
```python
def total(*args):            # any number of positional args -> tuple
    return sum(args)

def info(**kwargs):          # any number of keyword args -> dict
    return kwargs

total(1, 2, 3)               # 6
info(a=1, b=2)               # {'a':1,'b':2}
```

**Explanation**
- `*args` collects **any number** of positional arguments into a **tuple**. `total(1, 2, 3)` makes `args = (1, 2, 3)`, and `sum(args)` adds them: 6.
- `**kwargs` collects **any number** of keyword arguments into a **dictionary**. `info(a=1, b=2)` makes `kwargs = {'a': 1, 'b': 2}` and returns it.
- The names `args` / `kwargs` are only a convention; the `*` and `**` are what matter.

### Returning multiple values
Python returns a **tuple**, which you can unpack.
```python
def stats(nums):
    return min(nums), max(nums)

lo, hi = stats([3, 9, 1])
```

**Explanation**
- `min(nums)` and `max(nums)` are built-in functions giving the smallest and largest items.
- `return min(nums), max(nums)` returns **two values** separated by a comma; Python packs them into a tuple `(1, 9)`.
- `lo, hi = ...` **unpacks** that tuple: `lo = 1`, `hi = 9`.

**Warning:** never use a mutable default like `def f(x=[])`. The list is created once and shared across calls, so values pile up. Use `x=None` and create the list inside the function.

---

## Lesson 5: Modules and Scope

### Modules
A module is just a `.py` file. A package is a folder of modules.
```python
import math                       # built-in
from math import sqrt
import math as m
import my_utils                   # custom: my_utils.py in same folder
```

**Explanation**
- `import math`: loads the whole module. Use it as `math.sqrt(16)`.
- `from math import sqrt`: imports only `sqrt`; use it directly as `sqrt(16)`.
- `import math as m`: gives the module a short alias; use `m.sqrt(16)`.
- `import my_utils`: imports **your own** file `my_utils.py` (must be in the same folder or on the Python path). Use `my_utils.function_name()`.

```python
if __name__ == "__main__":
    main()
```

**Explanation**
- Every file has a built-in variable `__name__`. When you **run** the file directly it equals `"__main__"`; when another file **imports** it, it equals the module name.
- So this `if` runs `main()` only when the file is executed directly, not when imported. It prevents your script's main code from running accidentally during an import.

### Scope (LEGB rule)
Python looks for a name in this order: **L**ocal → **E**nclosing → **G**lobal → **B**uilt-in.

```python
x = 10                 # global
def f():
    x = 5              # local (separate variable)
    print(x)           # 5
def g():
    global x
    x = 99             # modifies the global
```

**Explanation**
- `x = 10` outside any function is a **global** variable.
- In `f()`, `x = 5` creates a **new local** variable also called `x`. It lives only inside `f`. The global `x` is still 10. `print(x)` prints 5.
- In `g()`, `global x` tells Python "I mean the global `x`, not a new local one". So `x = 99` changes the global to 99.
- Without `global`, assigning inside a function always creates a local variable.

### Docstrings
A string right under `def`/`class` that documents it. Read with `help(func)`.
```python
def add(a, b):
    """Return the sum of a and b."""
    return a + b
```

**Explanation**
- The triple-quoted line directly under `def` is the docstring. It describes what the function does.
- `help(add)` or `add.__doc__` shows it. Good docstrings help teammates and tools like IDEs.

---

## Lesson 6: Lists and Strings

### Lists (ordered, mutable)
```python
nums = [10, 20, 30]
nums.append(40)       # add at end
nums.insert(1, 15)    # add at index
nums.remove(20)       # remove by value
nums.pop()            # remove last (or pop(i))
nums.sort()           # in place
nums[0]; nums[-1]     # first, last
nums[1:3]             # slice
```

**Explanation** (start: `[10, 20, 30]`)
- `nums = [10, 20, 30]`: creates a list. Square brackets, items separated by commas. Order is kept and items can be changed.
- `nums.append(40)`: adds 40 at the **end** → `[10, 20, 30, 40]`.
- `nums.insert(1, 15)`: puts 15 at **index 1** and shifts the rest right → `[10, 15, 20, 30, 40]`. Indexes start at 0.
- `nums.remove(20)`: removes the first item whose **value** is 20 → `[10, 15, 30, 40]`.
- `nums.pop()`: removes and **returns** the last item (40) → `[10, 15, 30]`. `pop(0)` removes the item at index 0.
- `nums.sort()`: sorts the list **in place** (changes the original) → `[10, 15, 30]`. Use `sorted(nums)` to get a new list instead.
- `nums[0]` is the first item; `nums[-1]` is the last (negative index counts from the end).
- `nums[1:3]`: slice from index 1 up to (not including) index 3 → items at positions 1 and 2.

### List comprehension (short way to build lists)
```python
squares = [x**2 for x in range(5)]            # [0,1,4,9,16]
evens   = [x for x in range(10) if x % 2 == 0]
```

**Explanation**
- Pattern: `[expression for item in iterable if condition]`.
- Line 1: for each `x` in 0..4, compute `x**2` (x squared) and collect into a new list → `[0, 1, 4, 9, 16]`. Same as a for-loop with `append`, in one line.
- Line 2: loops `x` over 0..9 but keeps only those where `x % 2 == 0` (even) → `[0, 2, 4, 6, 8]`. The `if` at the end is a filter.

### Strings (ordered, **immutable**)
```python
s = "Data Engineer"
s[0]            # 'D'
s[0:4]          # 'Data'  (slicing)
s[::-1]         # reversed
"""multi
line string"""
```

**Explanation**
- `s[0]`: the character at index 0 → `'D'`.
- `s[0:4]`: characters from index 0 up to but **not** including 4 → `'Data'`.
- `s[::-1]`: the slice format is `[start:stop:step]`. Step `-1` walks backwards, so the whole string is reversed → `'reenignE ataD'`.
- Triple quotes create a string that can span **multiple lines**.
- **Immutable** means you cannot do `s[0] = "X"` (error). To change a string you build a new one.

**Formatting**
```python
f"{name} is {age}"            # f-string (preferred)
"{} is {}".format(name, age)
"%s is %d" % (name, age)
```

**Explanation**
- All three produce the same text, e.g. `Ash is 22`.
- f-string: variables written directly inside `{ }`. Easiest and fastest.
- `.format()`: each `{}` is a blank filled in order by the arguments.
- `%` style (old): `%s` is a placeholder for a string, `%d` for an integer; values come after `%` in a tuple.

**Concatenation:** `"a" + "b"` gives `"ab"`. Both sides must be strings (use `str()` to convert numbers).

**Common methods**
| Method | Use |
|--------|-----|
| `upper() lower() title()` | change case |
| `strip()` | remove spaces from ends |
| `split(",")` | string → list |
| `",".join(lst)` | list → string |
| `replace(a,b)` | replace text |
| `find()`, `startswith()`, `endswith()` | search |
| `isdigit()`, `isalpha()` | check content |

```python
"  hi, there ".strip()          # 'hi, there'
"a,b,c".split(",")              # ['a','b','c']
"-".join(["a", "b", "c"])       # 'a-b-c'
"hello".replace("l", "L")       # 'heLLo'
```

**Explanation**
- `strip()` removes the spaces at the start and end only (not in the middle).
- `split(",")` cuts the string at every comma and returns a **list**.
- `"-".join(list)` does the opposite: glues the items together using `-` between them.
- `replace("l", "L")` returns a new string with every `l` changed to `L`. The original is unchanged (strings are immutable).

---

## Lesson 7: Tuples, Dictionaries, Sets

### Tuple: ordered, **immutable**
```python
t = (1, 2, 3)
a, b, c = t        # unpacking
```

**Explanation**
- Round brackets make a tuple. Once created you cannot add, remove or change items.
- `a, b, c = t` is **unpacking**: assigns 1 to `a`, 2 to `b`, 3 to `c`. The number of variables must match the number of items.
- Use for fixed data (coordinates, DB rows). Faster than lists, can be dict keys.

### Dictionary: key → value
```python
d = {"name": "Ash", "age": 22}
d["city"] = "Noida"            # add/update
d.get("zip", "N/A")            # safe access
d.keys(); d.values(); d.items()
for k, v in d.items():
    print(k, v)
del d["age"]
```

**Explanation**
- Line 1: curly brackets with `key: value` pairs. Here keys are `"name"`, `"age"`.
- `d["city"] = "Noida"`: if the key is new, it **adds** the pair; if it exists, it **updates** the value.
- `d.get("zip", "N/A")`: looks up key `"zip"`. If missing, returns the default `"N/A"` instead of raising `KeyError` (which `d["zip"]` would).
- `d.keys()`, `d.values()`, `d.items()`: give all keys, all values, and all (key, value) pairs.
- The `for k, v in d.items():` loop goes through each pair, unpacking into `k` (key) and `v` (value), and prints them.
- `del d["age"]` deletes that key and its value.
- Keys must be unique and immutable.

### Set: unordered, **unique** values
```python
s = {1, 2, 2, 3}      # {1,2,3}
a | b    # union
a & b    # intersection
a - b    # difference
```

**Explanation**
- A set silently removes duplicates, so `{1, 2, 2, 3}` becomes `{1, 2, 3}`. It has no order and no indexing.
- `a | b` (union): everything in either set.
- `a & b` (intersection): only items in **both**.
- `a - b` (difference): items in `a` but **not** in `b`.
- Great for removing duplicates and fast membership checks (`x in s`).
- Note: `{}` alone creates an empty **dict**; use `set()` for an empty set.

### Mutable vs immutable
| Immutable (cannot change in place) | Mutable |
|---|---|
| int, float, str, tuple, bool | list, dict, set |

### Shallow copy vs deep copy
```python
import copy
a = [[1, 2], [3, 4]]
b = a                   # same object (not a copy)
c = a.copy()            # shallow: new outer list, SAME inner lists
d = copy.deepcopy(a)    # deep: everything copied
a[0][0] = 99
# c[0][0] -> 99 (affected),  d[0][0] -> 1 (safe)
```

**Explanation**
- `import copy` loads the copy module.
- `a` is a list that contains two inner lists (nested).
- `b = a`: **not a copy**. `b` is just another name for the same list. Any change through `a` shows in `b`.
- `c = a.copy()`: a **shallow copy**. A new outer list is made, but the inner lists are still the **same objects** shared with `a`.
- `d = copy.deepcopy(a)`: a **deep copy**. The outer and all inner lists are fully new and independent.
- `a[0][0] = 99` changes the first number of the first inner list. Because `c` shares that inner list, `c[0][0]` is now 99 too. `d` is independent so `d[0][0]` is still 1.

### Dict and set comprehensions
```python
{x: x**2 for x in range(3)}      # {0:0, 1:1, 2:4}
{x % 3 for x in range(10)}       # {0,1,2}
```

**Explanation**
- Dict comprehension: `{key: value for item in iterable}`. For x in 0,1,2 it creates `x: x squared` → `{0: 0, 1: 1, 2: 4}`.
- Set comprehension: curly braces with a single expression. It computes `x % 3` (remainder) for 0..9, giving 0,1,2,0,1,2,..., and since sets keep only unique values the result is `{0, 1, 2}`.

---

## Lesson 8: File Handling

### Read and write
```python
with open("data.txt", "w") as f:     # modes: r, w, a, r+, rb, wb
    f.write("hello\n")

with open("data.txt") as f:
    content = f.read()
```

**Explanation**
- `open("data.txt", "w")` opens (creates) the file for **writing**. The second argument is the **mode**.
- `with ... as f:` is a **context manager**. It gives you the file object `f` and **automatically closes** the file when the block ends, even if an error occurs. Always use it.
- `f.write("hello\n")` writes the text. `\n` is a newline (write does not add one automatically).
- The second `with open("data.txt")` opens for **reading** (default mode `r`).
- `f.read()` reads the entire file into one string stored in `content`.
- **Modes:** `r` read, `w` write (**overwrites** existing content!), `a` append (adds to the end), `r+` read and write, `b` = binary (`rb`, `wb`).

### Large files: read line by line (memory safe)
```python
with open("big.txt") as f:
    for line in f:          # reads one line at a time
        process(line)
```

**Explanation**
- The file object is **iterable**. `for line in f` pulls **one line at a time** from disk, so memory stays small even for a multi-GB file.
- `process(line)` is a placeholder for whatever you do with each line (clean it, parse it, count it).
- Never `f.read()` a huge file because it loads everything into RAM.

### CSV
```python
import csv
with open("a.csv") as f:
    for row in csv.DictReader(f):
        print(row["name"])
```

**Explanation**
- `import csv` loads Python's built-in CSV module.
- `csv.DictReader(f)` reads the CSV and treats the **first line as headers**. Each row becomes a dictionary like `{"name": "Ash", "age": "22"}`.
- `row["name"]` gets the value of the `name` column for that row. All values are strings.

### JSON
```python
import json
data = json.load(open("a.json"))        # file -> Python
json.dump(data, open("b.json", "w"), indent=2)
s = json.dumps(data)                    # Python -> string
d = json.loads(s)                       # string -> Python
```

**Explanation**
- `json.load(file)`: reads a JSON **file** and converts it to Python objects (dict/list).
- `json.dump(data, file, indent=2)`: writes Python data **into a file** as JSON; `indent=2` makes it nicely formatted with 2-space indentation.
- `json.dumps(data)`: converts Python data to a JSON **string** (the **s** stands for string).
- `json.loads(s)`: converts a JSON **string** back to Python.
- Memory trick: no `s` = file, with `s` = string.
- Mapping: JSON object→dict, array→list, string→str, number→int/float, true/false→True/False, null→None.

### Exceptions in file operations
```python
try:
    with open("missing.txt") as f: ...
except FileNotFoundError:
    print("File not found")
```

**Explanation**
- The code in `try` is attempted. If the file does not exist Python raises `FileNotFoundError`.
- `except FileNotFoundError:` catches that **specific** error and runs the message instead of crashing the program.
- The `...` (ellipsis) is just a placeholder meaning "code goes here".

### Paths: `os` and `pathlib`
```python
import os
os.path.join("data", "a.csv"); os.path.exists("a.csv"); os.listdir(".")

from pathlib import Path
p = Path("data") / "a.csv"      # modern, cross-platform
p.exists(); p.suffix; p.read_text()
```

**Explanation**
- `os.path.join("data", "a.csv")` builds the path `data/a.csv` using the correct separator for your OS (`/` or `\`).
- `os.path.exists("a.csv")` returns `True` if the file/folder exists.
- `os.listdir(".")` lists the names of everything in the current folder (`.` = current).
- `Path("data") / "a.csv"`: `pathlib` lets you join paths with the `/` operator. Cleaner and works on all operating systems.
- `p.exists()` checks existence, `p.suffix` gives the extension (`.csv`), `p.read_text()` reads the file as text.

---

## Lesson 9: Object-Oriented Programming (OOP)

**Idea:** bundle data (attributes) and behavior (methods) into a **class**. A class is a blueprint; an **object** is a real thing built from it.

```python
class Employee:
    company = "HCLTech"                 # class attribute

    def __init__(self, name, salary):   # constructor
        self.name = name                # instance attributes
        self.salary = salary

    def raise_salary(self, pct):        # method
        self.salary += self.salary * pct / 100

e = Employee("Ash", 50000)
e.raise_salary(10)
```

**Explanation**
- `class Employee:` defines a new class (blueprint). Class names use CapitalizedWords.
- `company = "HCLTech"`: a **class attribute**, shared by all employees.
- `def __init__(self, name, salary):` is the **constructor**. It runs **automatically** when you create an object. Double underscores on both sides mark it as a special method.
- `self` means "this particular object". Python passes it automatically; you never supply it when calling.
- `self.name = name` and `self.salary = salary` store the given values on the object. These are **instance attributes**, different for each object.
- `def raise_salary(self, pct):` is a **method** (a function inside a class). It increases salary by a percentage: `self.salary * pct / 100` is the raise amount, `+=` adds it to the current salary.
- `e = Employee("Ash", 50000)` **creates an object** (instance). Python calls `__init__` with `name="Ash"`, `salary=50000`.
- `e.raise_salary(10)` calls the method: raise = 50000 × 10 / 100 = 5000, so `e.salary` becomes 55000.

### Inheritance: child reuses parent
```python
class Manager(Employee):
    def __init__(self, name, salary, team):
        super().__init__(name, salary)   # call parent constructor
        self.team = team
```

**Explanation**
- `class Manager(Employee):` makes `Manager` a **child** of `Employee`. It automatically gets `company`, `raise_salary()` and everything else from the parent.
- The child's own `__init__` takes the parent's values plus a new one, `team`.
- `super().__init__(name, salary)` calls the **parent's** constructor so `name` and `salary` are set up properly (no need to rewrite that code).
- `self.team = team` adds an attribute that only managers have.

### Polymorphism: same method name, different behavior
```python
class Dog:
    def speak(self): return "Woof"
class Cat:
    def speak(self): return "Meow"

for a in [Dog(), Cat()]:
    print(a.speak())          # each behaves differently
```

**Explanation**
- Two unrelated classes both define a method named `speak`, but each returns something different.
- `[Dog(), Cat()]` creates one object of each class inside a list.
- The loop calls `a.speak()` on each object. Python runs the version that belongs to that object's class: `Woof` for the dog, `Meow` for the cat.
- This is polymorphism ("many forms"): same call, different behavior. Method overriding (a child redefining a parent's method) is the other common form.

**The four pillars:** Encapsulation (bundle and protect data), Abstraction (hide complexity), Inheritance (reuse), Polymorphism (many forms).

---

## Lesson 10: NumPy and Pandas

### NumPy: fast numeric arrays
```python
import numpy as np
a = np.array([1, 2, 3])
a * 2                        # [2,4,6]  (vectorized, no loop)
np.arange(5); np.zeros((2,3)); a.mean(); a.sum()
m = np.array([[1,2],[3,4]])  # 2D array
m.shape                      # (2,2)
```

**Explanation**
- `import numpy as np`: loads NumPy with the standard short name `np`.
- `np.array([1, 2, 3])` converts a list into a NumPy **array** (a fixed-type, fast structure).
- `a * 2` multiplies **every element** by 2 → `[2, 4, 6]`. This is **vectorization**: no loop needed. With a normal list, `[1,2,3] * 2` would repeat the list instead.
- `np.arange(5)` → `[0,1,2,3,4]` (like `range`, but returns an array).
- `np.zeros((2,3))` → a 2-row, 3-column array filled with zeros.
- `a.mean()` → average (2.0); `a.sum()` → total (6).
- `m = np.array([[1,2],[3,4]])` makes a **2D array** (matrix) from a list of lists.
- `m.shape` → `(2, 2)`, meaning 2 rows and 2 columns.
- Much faster than Python lists because of vectorization.

### Pandas: tables of data
- **Series** = one column (1D)
- **DataFrame** = table (2D, rows and columns)

```python
import pandas as pd
df = pd.read_csv("sales.csv")          # also read_json
df.head(); df.info(); df.describe(); df.shape
```

**Explanation**
- `import pandas as pd`: standard short name `pd`.
- `pd.read_csv("sales.csv")` loads the CSV file into a **DataFrame** `df` (`pd.read_json` for JSON).
- `df.head()` shows the first 5 rows (a quick peek).
- `df.info()` shows column names, data types and non-null counts.
- `df.describe()` shows statistics (count, mean, min, max, etc.) for numeric columns.
- `df.shape` gives `(rows, columns)`.

**Select and filter**
```python
df["city"]                        # one column
df[["city", "amount"]]            # multiple
df[df["amount"] > 1000]           # filter rows
df.loc[0, "city"]                 # label-based
df.iloc[0, 1]                     # position-based
```

**Explanation**
- `df["city"]` returns one column (a Series).
- `df[["city", "amount"]]`: the **double brackets** (a list inside brackets) return several columns as a DataFrame.
- `df[df["amount"] > 1000]`: the inner part `df["amount"] > 1000` gives True/False for each row; the outer `df[...]` keeps only rows where it is True.
- `df.loc[0, "city"]`: select by **label**: row label 0, column named "city".
- `df.iloc[0, 1]`: select by **integer position**: row 0, column number 1 (second column).

**GroupBy (split → aggregate → combine)**
```python
df.groupby("city")["amount"].sum()
df.groupby("city").agg({"amount": ["sum", "mean"]})
```

**Explanation**
- `groupby("city")` splits the rows into groups, one per city.
- `["amount"].sum()` takes the `amount` column and adds it up within each group → total amount per city.
- The second line uses `.agg()` with a dictionary: for the column `amount`, compute **both** sum and mean per city.
- This is the same idea as SQL `GROUP BY`.

**Joins**
```python
pd.merge(orders, customers, on="cust_id", how="left")   # inner, left, right, outer
```

**Explanation**
- `pd.merge` joins two DataFrames, like SQL JOIN.
- `on="cust_id"`: the common column used to match rows.
- `how="left"`: keep **all** rows from `orders` (the left table), adding matching customer info; unmatched get `NaN`. Other options: `inner` (matches only), `right`, `outer` (all rows from both).

**Missing values**
```python
df.isnull().sum()               # count nulls per column
df.dropna()                     # remove rows with nulls
df.fillna(0)                    # replace nulls
df["age"].fillna(df["age"].mean())
```

**Explanation**
- `df.isnull()` gives True where a value is missing; `.sum()` counts the Trues per column.
- `df.dropna()` removes every row that has any missing value.
- `df.fillna(0)` replaces all missing values with 0.
- The last line fills missing ages with the **average age**: `df["age"].mean()` calculates the mean (ignoring nulls) and `fillna` uses it.
- Note: these return **new** DataFrames; assign the result back (`df = df.dropna()`) to keep it.

**Write**
```python
df.to_csv("out.csv", index=False)
df.to_json("out.json", orient="records")
```

**Explanation**
- `to_csv` saves the DataFrame to a CSV file. `index=False` stops Pandas from writing the row numbers as an extra column.
- `to_json(... orient="records")` saves as a list of row objects: `[{"col": value, ...}, ...]`.

---

## Lesson 11: Utility Libraries

### Matplotlib: basic plotting
```python
import matplotlib.pyplot as plt
plt.plot([1,2,3], [10,20,15])        # line
plt.bar(["A","B"], [5,8])            # bar
plt.title("Sales"); plt.xlabel("Month"); plt.ylabel("Amount")
plt.show()
```

**Explanation**
- `import matplotlib.pyplot as plt`: the plotting interface, usually named `plt`.
- `plt.plot(x, y)`: draws a **line chart**. Here x = [1,2,3], y = [10,20,15].
- `plt.bar(labels, heights)`: draws a **bar chart** with bars "A" (height 5) and "B" (height 8).
- `plt.title(...)`, `plt.xlabel(...)`, `plt.ylabel(...)`: set the chart title and axis labels.
- `plt.show()` displays the chart window. Nothing appears until you call it.

### datetime
```python
from datetime import datetime, timedelta
now = datetime.now()
now.strftime("%Y-%m-%d %H:%M")                 # date -> string
datetime.strptime("2026-01-15", "%Y-%m-%d")    # string -> date
now + timedelta(days=7)
```

**Explanation**
- `datetime.now()` returns the current date and time.
- `strftime("%Y-%m-%d %H:%M")` converts a date to a **formatted string**. Codes: `%Y` 4-digit year, `%m` month, `%d` day, `%H` hour (24h), `%M` minutes. Example output `2026-11-19 14:30`.
- `strptime("2026-01-15", "%Y-%m-%d")` **parses** text into a datetime object; the second argument tells Python the format of the text.
- `timedelta(days=7)` represents a duration of 7 days; adding it to `now` gives the date one week later.
- Memory trick: strf**time** = to **str**ing, strp**time** = **p**arse from string.

### requests: call APIs
```python
import requests
r = requests.get("https://api.example.com/users", params={"page": 1}, timeout=10)
r.status_code        # 200 = OK
r.json()             # response body as dict/list
r.raise_for_status() # raises error on 4xx/5xx
requests.post(url, json={"name": "Ash"})
```

**Explanation**
- `requests.get(url, ...)` sends an HTTP **GET** request (asks for data) and returns a response object `r`.
- `params={"page": 1}` adds a query string: the URL becomes `...users?page=1`.
- `timeout=10` gives up after 10 seconds so your program doesn't hang forever.
- `r.status_code`: the HTTP status. 200 = success, 404 = not found, 500 = server error.
- `r.json()` converts the JSON response body into a Python dict/list.
- `r.raise_for_status()` raises an exception if the status is 4xx or 5xx, so failures are not ignored silently.
- `requests.post(url, json={...})` sends an HTTP **POST** (sends data to the server). `json=` converts the dict to JSON automatically.

### os module
`os.getcwd()` (current folder), `os.listdir()` (list a folder), `os.makedirs()` (create folders), `os.environ.get("KEY")` (read an environment variable; the right place for secrets like API keys).

---

## Lesson 12: Error Handling and Debugging

### Types of errors
| Type | When | Example |
|------|------|---------|
| Syntax error | code is written wrong, won't run | missing `:` |
| Runtime exception | error while running | `ZeroDivisionError`, `KeyError`, `TypeError`, `ValueError`, `FileNotFoundError`, `IndexError` |
| Logical error | runs but gives wrong answer | wrong formula |

### try / except / else / finally
```python
try:
    x = int(input())
    result = 10 / x
except ValueError:
    print("Not a number")
except ZeroDivisionError:
    print("Cannot divide by zero")
else:
    print("Success", result)       # runs only if NO error
finally:
    print("Always runs")           # cleanup (close files, connections)
```

**Explanation**
- `try:` the risky code goes here. Python runs it and watches for errors.
- `x = int(input())`: reads text and converts to a number. If the user types `abc`, this raises `ValueError`.
- `result = 10 / x`: if the user types `0`, this raises `ZeroDivisionError`.
- `except ValueError:` runs **only** if a `ValueError` happened → prints "Not a number".
- `except ZeroDivisionError:` runs only for division by zero.
- `else:` runs **only if no exception occurred** in `try`. Good for code that depends on the try succeeding.
- `finally:` runs **always**, error or not. Used for cleanup like closing files or database connections.
- Only the matching `except` runs; the program continues normally afterward instead of crashing.

### Custom exceptions
```python
class InvalidDataError(Exception):
    pass

if age < 0:
    raise InvalidDataError("Age cannot be negative")
```

**Explanation**
- `class InvalidDataError(Exception):` creates your own error type by inheriting from the built-in `Exception`. A clear name makes failures easy to understand.
- `pass` means the class needs no extra code; it just inherits everything.
- `raise InvalidDataError("...")` **throws** the error deliberately with a message when data is invalid (here when `age < 0`). Somewhere else you can `except InvalidDataError:` to handle it.

### When to catch vs not catch
- **Catch** when you can handle it: retry, use default, log and continue, show friendly message.
- **Don't catch** when you cannot do anything useful. Let it fail loudly.
- Never use bare `except:` or silently `pass`. It hides bugs. Catch specific exceptions.

### Debugging techniques
- `print()` statements (quick but messy)
- **Logging** (better, see next lesson)
- **Debugger:** VS Code breakpoints (click left of the line number, run in debug mode, then step over/into); `pdb` (`breakpoint()` or `import pdb; pdb.set_trace()` pauses the program so you can inspect variables)
- Read the **traceback from the bottom up**: the last line is the error type and message, the lines above show the chain of calls and line numbers that led to it.

---

## Lesson 13: Unit Testing and Logging

### Unit testing with `unittest`
Test small pieces of code automatically.

```python
import unittest
from calc import add

class TestCalc(unittest.TestCase):          # inherit TestCase
    def setUp(self):                         # runs before each test
        self.a, self.b = 2, 3

    def test_add(self):                      # method name MUST start with test
        self.assertEqual(add(self.a, self.b), 5)

    def test_error(self):
        with self.assertRaises(ZeroDivisionError):
            1 / 0

if __name__ == "__main__":
    unittest.main()
```

**Explanation**
- `import unittest`: Python's built-in testing framework.
- `from calc import add`: imports the function we want to test from a file `calc.py`.
- `class TestCalc(unittest.TestCase):` a group of tests. Inheriting from `TestCase` gives you all the assert methods.
- `def setUp(self):` special method that runs **before every test**. Use it to prepare data. Here it sets `self.a = 2` and `self.b = 3`. (`tearDown` runs after each test for cleanup.)
- `def test_add(self):` a test. The name **must start with `test`** or it will not run.
- `self.assertEqual(add(self.a, self.b), 5)` checks that `add(2, 3)` equals 5. If not, the test fails and reports it.
- `def test_error(self):` a test that checks an error is raised.
- `with self.assertRaises(ZeroDivisionError):` passes only if the code inside raises `ZeroDivisionError`. `1 / 0` does, so this test passes.
- `if __name__ == "__main__": unittest.main()` runs all tests when you execute the file directly.
- Run from terminal: `python -m unittest` (or `python -m unittest test_calc.py`).

**Common asserts:** `assertEqual`, `assertNotEqual`, `assertTrue`, `assertFalse`, `assertIn`, `assertIsNone`, `assertRaises`.

### Logging
```python
import logging
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
    filename="app.log"
)
logging.info("Pipeline started")
logging.error("Load failed")
```

**Explanation**
- `import logging`: built-in logging module.
- `logging.basicConfig(...)` sets up logging once at the start of the program.
  - `level=logging.INFO`: show messages of level INFO and above (hides DEBUG).
  - `format="%(asctime)s - %(levelname)s - %(message)s"`: how each line looks: timestamp, level name, then your message. Example: `2026-11-19 10:00:01 - INFO - Pipeline started`.
  - `filename="app.log"`: write logs to this file instead of the screen.
- `logging.info("Pipeline started")`: records a normal progress message.
- `logging.error("Load failed")`: records an error message.

**Levels (low → high):** `DEBUG` < `INFO` < `WARNING` < `ERROR` < `CRITICAL`. Setting a level shows that level and above.

**Logging vs print**
| print | logging |
|-------|---------|
| always prints | can filter by level |
| console only | file, console, remote |
| no timestamp | timestamp, level, module |
| must delete later | switch off by changing level |

**Common pitfalls:** calling `basicConfig` more than once (only the first call works), logging secrets or passwords, using `print` in production, wrong level (everything as ERROR), using f-strings inside log calls (prefer `logging.info("x=%s", x)`), and not using `logging.exception()` inside `except` to capture the traceback.

---

## Lesson 14: Advanced Topics

### Lambda (anonymous one-line function)
```python
square = lambda x: x ** 2
sorted(data, key=lambda r: r["age"])
```

**Explanation**
- `lambda x: x ** 2` is a small function with no name: input `x`, output `x` squared. Equivalent to `def square(x): return x ** 2`. Assigning it to `square` lets you call `square(4)` → 16.
- `sorted(data, key=lambda r: r["age"])`: `sorted` needs to know **what to sort by**. The lambda takes each record `r` and returns its `"age"`, so the list of dictionaries is sorted by age. Lambdas are mostly used like this, passed into other functions.

### `map`, `filter`, `reduce`
```python
nums = [1, 2, 3, 4]
list(map(lambda x: x * 2, nums))          # [2,4,6,8]    transform each
list(filter(lambda x: x % 2 == 0, nums))  # [2,4]        keep matching
from functools import reduce
reduce(lambda a, b: a + b, nums)          # 10           combine into one
```

**Explanation**
- `map(function, items)` applies the function to **every** item. The lambda doubles each number. `map` returns a lazy object, so `list(...)` turns it into a list → `[2, 4, 6, 8]`.
- `filter(function, items)` keeps only items for which the function returns True. The lambda tests "is even" → `[2, 4]`.
- `reduce` (from `functools`) combines all items into **one value**. The lambda takes two values `a` (running result) and `b` (next item) and adds them: ((1+2)+3)+4 = 10.
- These same ideas (map / filter / reduce) are the foundation of **Spark RDD** in Phase 4.

### Iterators and generators
- **Iterator:** object you can loop over one item at a time (`iter()`, `next()`).
- **Generator:** function using `yield`. Produces values lazily, one at a time, so it uses very little memory.

```python
def count_up(n):
    i = 1
    while i <= n:
        yield i          # pauses here, resumes on next()
        i += 1

for x in count_up(3): print(x)    # 1 2 3

gen = (x**2 for x in range(10**9))   # generator expression: no memory blowup
```

**Explanation**
- `def count_up(n):` looks like a function, but because it contains `yield` it is a **generator function**.
- `i = 1` starts the counter; `while i <= n:` loops until `i` passes `n`.
- `yield i` **hands out** the value `i` and **pauses** the function, remembering where it was. It does not end like `return`. When the next value is requested, it resumes right after the `yield`.
- `i += 1` moves to the next number after resuming.
- `for x in count_up(3): print(x)` asks for values one by one: 1, then 2, then 3, and stops when the function finishes.
- `gen = (x**2 for x in range(10**9))`: **round brackets** make a generator expression. It describes a billion squares but does **not** create them; each is computed only when requested. A list of a billion numbers would crash your memory.
- Why important for DE: processing huge files and streams without loading everything.

### Decorators (basic)
A function that **wraps** another function to add behavior.
```python
import time
def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time()-start:.2f}s")
        return result
    return wrapper

@timer
def load_data():
    ...
```

**Explanation**
- `import time` gives access to `time.time()` (current time in seconds).
- `def timer(func):` the **decorator**. It receives a function `func` as its input.
- Inside, `def wrapper(*args, **kwargs):` defines a new function that accepts any arguments, so it works for any function.
- `start = time.time()` notes the start time.
- `result = func(*args, **kwargs)` calls the **original** function with the same arguments and keeps its result.
- `print(f"{func.__name__} took {time.time()-start:.2f}s")` prints the function name and elapsed time. `:.2f` formats the number to 2 decimals.
- `return result` gives back whatever the original function returned.
- `return wrapper` the decorator returns the new wrapped function.
- `@timer` above `def load_data():` is shorthand for `load_data = timer(load_data)`. From now on, calling `load_data()` runs the timed wrapper version.
- `...` is a placeholder for the real function body.
- Typical uses: timing, logging, retry, authentication.

### Regular expressions (`re`)
```python
import re
re.search(r"\d+", "Order 123")          # first match
re.findall(r"\w+@\w+\.com", text)       # all matches
re.sub(r"\s+", " ", text)               # replace
re.match(r"^\d{10}$", phone)            # match at start
```

**Explanation**
- `import re` loads the regular expression module. The `r` before the quote makes a **raw string** so backslashes are kept as-is.
- `re.search(r"\d+", "Order 123")` scans the text and returns the **first** match. `\d+` means "one or more digits" → matches `123`.
- `re.findall(r"\w+@\w+\.com", text)` returns **all** matches as a list. Pattern: word characters, `@`, word characters, a literal dot (`\.`), `com` → finds simple email addresses.
- `re.sub(r"\s+", " ", text)` **replaces** every run of whitespace (`\s+`) with a single space. Useful for cleaning messy text.
- `re.match(r"^\d{10}$", phone)` checks the **whole** string from start (`^`) to end (`$`) is exactly 10 digits (`\d{10}`), e.g. validating a phone number. `match` only checks from the start of the string.

| Pattern | Meaning |
|---|---|
| `\d` `\w` `\s` | digit, word char, space |
| `.` | any char |
| `* + ?` | 0+, 1+, 0 or 1 |
| `{n}` | exactly n |
| `^ $` | start, end |
| `[abc]` | any of a, b, c |
| `( )` | capture group |

---

## Lesson 15: Data Integration, Environments and GitHub Copilot

### Python JSON (marked "Important" in syllabus)
Nested JSON is everywhere in APIs. Know how to walk it.
```python
data = {"user": {"name": "Ash", "skills": ["SQL", "Python"]}}
data["user"]["skills"][0]                      # 'SQL'
data.get("user", {}).get("age", "unknown")     # safe nested access

import pandas as pd
df = pd.json_normalize(list_of_nested_dicts)   # flatten nested JSON to a table
```

**Explanation**
- `data` is a dictionary containing another dictionary (`"user"`), which contains a list (`"skills"`). This is typical API data.
- `data["user"]["skills"][0]`: go step by step: `data["user"]` → the inner dict; `["skills"]` → the list; `[0]` → the first item, `'SQL'`.
- `data.get("user", {}).get("age", "unknown")`: a **safe** way to read nested keys. `.get("user", {})` returns the inner dict (or an empty dict if missing). `.get("age", "unknown")` returns the age, or `"unknown"` if there is no such key. This avoids `KeyError` when the data is incomplete.
- `pd.json_normalize(...)` converts a list of nested dictionaries into a flat table; nested keys become columns like `user.name`.

### API calls (data ingestion pattern)
```python
import requests, json
r = requests.get(url, headers={"Authorization": "Bearer <token>"}, timeout=10)
r.raise_for_status()
records = r.json()
df = pd.DataFrame(records)
df.to_csv("raw/users.csv", index=False)       # land the data
```

**Explanation**
- `requests.get(url, headers=..., timeout=10)`: calls the API. `headers` carries extra info; `Authorization: Bearer <token>` is the common way to prove who you are (replace `<token>` with your real token, stored safely in an environment variable).
- `r.raise_for_status()`: stops with an error if the call failed (4xx/5xx).
- `records = r.json()`: converts the response into Python data (usually a list of dicts).
- `pd.DataFrame(records)`: turns that list of dicts into a table.
- `df.to_csv("raw/users.csv", index=False)`: **lands** (saves) the data in a raw file. This request → parse → store flow is the basic **ingestion** step of a pipeline.
- Real pipelines add: pagination, retries, error handling, logging.

### Web scraping (basic)
```python
from bs4 import BeautifulSoup
html = requests.get(url).text
soup = BeautifulSoup(html, "html.parser")
titles = [h.text for h in soup.find_all("h2")]
```

**Explanation**
- `BeautifulSoup` (from the `bs4` package, install with `pip install beautifulsoup4`) parses HTML.
- `requests.get(url).text` downloads the page's HTML as text.
- `BeautifulSoup(html, "html.parser")` builds a searchable structure from the HTML using Python's built-in parser.
- `soup.find_all("h2")` finds every `<h2>` heading tag; the list comprehension extracts the text of each → `titles` is a list of heading strings.
- Prefer an API when one exists. Respect `robots.txt` and terms of use.

### SQLite (basic): built-in file database
```python
import sqlite3
conn = sqlite3.connect("test.db")
cur = conn.cursor()
cur.execute("CREATE TABLE IF NOT EXISTS emp(id INTEGER, name TEXT)")
cur.execute("INSERT INTO emp VALUES (?, ?)", (1, "Ash"))   # ? placeholders prevent SQL injection
conn.commit()
rows = cur.execute("SELECT * FROM emp").fetchall()

df = pd.read_sql("SELECT * FROM emp", conn)   # straight into Pandas
conn.close()
```

**Explanation**
- `sqlite3` is built into Python; no installation needed.
- `sqlite3.connect("test.db")` opens (or creates) a database stored in the file `test.db`. `conn` is the connection.
- `cur = conn.cursor()` creates a **cursor**, the object used to run SQL commands.
- `cur.execute("CREATE TABLE IF NOT EXISTS emp(...)")` creates table `emp` with columns `id` (integer) and `name` (text), only if it does not already exist.
- `cur.execute("INSERT INTO emp VALUES (?, ?)", (1, "Ash"))` inserts a row. The `?` are **placeholders**; the values are passed separately in a tuple. This prevents **SQL injection** (never build SQL by pasting user text into the string).
- `conn.commit()` saves the changes permanently. Without it, inserts are lost.
- `cur.execute("SELECT * FROM emp").fetchall()` runs a query and returns **all rows** as a list of tuples, e.g. `[(1, 'Ash')]`.
- `pd.read_sql(query, conn)` runs the query and loads the result directly into a Pandas DataFrame.
- `conn.close()` closes the connection when done.

### Virtual environments and pip
A virtual environment gives each project its own isolated packages, so projects with different library versions don't clash.
```bash
python -m venv venv
source venv/bin/activate         # Windows: venv\Scripts\activate
pip install pandas requests
pip freeze > requirements.txt    # save dependencies
pip install -r requirements.txt  # recreate on another machine
deactivate
```

**Explanation**
- `python -m venv venv`: creates a new virtual environment in a folder named `venv`.
- `source venv/bin/activate` (Mac/Linux) or `venv\Scripts\activate` (Windows): **activates** it. Your terminal now uses this environment's own Python and packages.
- `pip install pandas requests`: installs packages **inside** the active environment only.
- `pip freeze > requirements.txt`: lists every installed package with its version and saves it into the file `requirements.txt` (`>` redirects output to a file).
- `pip install -r requirements.txt`: installs everything listed in that file, so a teammate or a server can recreate the same environment.
- `deactivate`: leaves the virtual environment.

### GitHub Copilot (in the syllabus, short and practical)
AI pair programmer inside your IDE (VS Code, JetBrains) that suggests code as you type.

| Area | What to know |
|---|---|
| Basics | Suggests lines/functions from comments and context; accept with Tab; supports many languages |
| Copilot Chat | Ask questions, explain code, generate tests, in the IDE |
| Advanced uses | Generate commit messages, docstrings/documentation, refactoring, optimisation, debugging help, Copilot for CLI |
| Strengths | Boilerplate, repetitive code, tests, docs, unfamiliar syntax |
| Limitations | Can be wrong or insecure, may use outdated APIs, limited knowledge of your whole project |
| Best practices | **Always review** suggestions, write clear comments/prompts, give context, test the output, don't paste secrets |
| Legacy code | Great for explaining and documenting old code before refactoring |

### Phase 3 revision checklist
- [ ] `is` vs `==`, mutable vs immutable, shallow vs deep copy
- [ ] `*args`, `**kwargs`, default arguments, LEGB scope
- [ ] List/dict/set comprehensions
- [ ] `with open`, read big files line by line, JSON load/dump
- [ ] OOP: class, `__init__`, inheritance, polymorphism
- [ ] Pandas: read, filter, groupby, merge, fillna
- [ ] try/except/else/finally, custom exceptions
- [ ] unittest structure, logging levels
- [ ] lambda/map/filter/reduce, generators, decorators
- [ ] requests API call → DataFrame → file

---

# Quick Interview Questions (Python)

1. `is` vs `==`? → identity vs value equality.
2. List vs tuple? → mutable vs immutable.
3. Shallow vs deep copy? → deep copies nested objects too.
4. What are `*args` / `**kwargs`? → variable positional (tuple) / keyword (dict) arguments.
5. What is a generator and why use it? → lazy `yield`, low memory for large data.
6. What is a decorator? → function that wraps another to add behavior.
7. How do you read a huge file? → iterate line by line / chunks (`pd.read_csv(chunksize=...)`).
8. What does `with` do? → context manager, auto-closes resources.
9. `try/except/else/finally` order? → else only if no error; finally always.
10. How do you handle missing values in Pandas? → `isnull`, `dropna`, `fillna`.

---

*Next: Phase 4 (PySpark). The map/filter/reduce and lambda ideas from Lesson 14 carry straight into Spark RDDs.*
