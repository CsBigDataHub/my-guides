# From Python to Emacs Lisp: A Comprehensive Guide

## Table of Contents

1. [Mental Model Shift](#mental-model-shift)
2. [The Environment](#the-environment)
3. [Basic Syntax](#basic-syntax)
4. [Data Types](#data-types)
5. [Variables and Binding](#variables-and-binding)
6. [Functions](#functions)
7. [Control Flow](#control-flow)
8. [Lists and Sequences](#lists-and-sequences)
9. [Hash Tables and Association Lists](#hash-tables-and-association-lists)
10. [Strings](#strings)
11. [Macros](#macros)
12. [Error Handling](#error-handling)
13. [Object System](#object-system)
14. [Buffers and Text](#buffers-and-text)
15. [Processes and Async](#processes-and-async)
16. [Modules and Libraries](#modules-and-libraries)
17. [Testing](#testing)
18. [Common Patterns](#common-patterns)
19. [Debugging](#debugging)
20. [Style and Conventions](#style-and-conventions)

---

## 1. Mental Model Shift

This is the most important section. Before writing a single line, you need to understand how Emacs Lisp differs fundamentally from Python.

### Python thinks in objects. Emacs Lisp thinks in buffers and symbols.

In Python, your program manipulates objects in memory. In Emacs Lisp, your program manipulates **the editor itself**. Everything is live. When you evaluate code, it immediately affects the running Emacs instance. There is no separate compile-run cycle.

### The REPL is the program

In Python:

```
write code → run interpreter → see result → modify file → repeat
```

In Emacs Lisp:

```
everything is already running → evaluate expressions anywhere → changes take effect immediately
```

### Evaluation is everywhere

In Python you run a file or a REPL. In Emacs Lisp you can evaluate:

- Any expression in any buffer with `C-x C-e` (eval last sexp)
- An entire buffer with `M-x eval-buffer`
- A region with `M-x eval-region`
- A single expression with `M-: ` (eval-expression)
- Definitions with `C-M-x` (eval-defun)

### Dynamic scope (mostly)

Python uses lexical scope everywhere. Emacs Lisp historically used **dynamic scope** (variables are looked up in the call stack, not the definition environment). Modern Emacs Lisp uses lexical scope when you add this to the top of your file:

```elisp
;;; myfile.el --- description  -*- lexical-binding: t; -*-
```

**Always add this.** Without it you will encounter baffling bugs. All code in this guide assumes lexical binding is enabled.

### Lisp-2

Python is a Lisp-1: functions and variables share one namespace. Emacs Lisp is a **Lisp-2**: functions and variables have **separate namespaces**. A variable named `foo` and a function named `foo` can coexist without conflict.

```elisp
(defun foo (x) (* x 2))
(setq foo 42)

foo        ; → 42   (variable)
(foo 10)   ; → 20   (function)
```

This is why you will see `(funcall fn arg)` and `(apply fn args)` instead of just `(fn arg)` when calling a function stored in a variable.

---

## 2. The Environment

### Setting Up

You need Emacs 29.1 or later. The built-in tools are sufficient to start:

- `*scratch*` buffer: a live Lisp playground, starts in `lisp-interaction-mode`
- `M-x ielm`: a proper Emacs Lisp REPL
- `M-x describe-function` (`C-h f`): documentation for any function
- `M-x describe-variable` (`C-h v`): documentation for any variable
- `M-x describe-key` (`C-h k`): what a key binding does
- `M-x apropos` (`C-h a`): search for functions/variables by pattern

### The _scratch_ Buffer

Think of it as Python's interactive interpreter but better. Type an expression and press `C-j` to evaluate it and insert the result, or `C-x C-e` to evaluate and show the result in the minibuffer.

```elisp
(+ 1 2 3)    ; press C-x C-e → shows 6 in minibuffer
             ; press C-j     → inserts 6 into buffer
```

### Package Management

Python has pip. Emacs has `package.el` and the more modern `use-package`:

```elisp
;; In your init.el

;; Add MELPA repository
(require 'package)
(add-to-list 'package-archives
             '("melpa" . "https://melpa.org/packages/") t)
(package-initialize)

;; Install and configure a package
(use-package magit
  :ensure t
  :bind ("C-c g" . magit-status))
```

Python equivalent:

```python
# requirements.txt
# magit==1.0.0

# in your code
import magit
```

### File Structure

A typical Emacs Lisp file (`.el`) has this structure:

```elisp
;;; my-package.el --- Short description  -*- lexical-binding: t; -*-

;; Copyright (C) 2024  Your Name
;; Author: Your Name <you@example.com>
;; Version: 1.0.0
;; Package-Requires: ((emacs "29.1"))
;; Keywords: tools

;;; Commentary:
;; Longer description of what this package does.

;;; Code:

(require 'some-dependency)

;; ... your code here ...

(provide 'my-package)
;;; my-package.el ends here
```

Python equivalent:

```python
"""
Short description.

Longer description of what this module does.
"""

import some_dependency

# ... your code here ...
```

---

## 3. Basic Syntax

### Everything is an S-expression

The fundamental syntactic unit is the **symbolic expression** (sexp). It is either an atom (a single value) or a list enclosed in parentheses.

```elisp
42           ; atom: integer
3.14         ; atom: float
"hello"      ; atom: string
t            ; atom: true
nil          ; atom: false AND empty list
:keyword     ; atom: keyword symbol
'symbol      ; atom: quoted symbol
foo          ; atom: symbol (variable reference)

(+ 1 2)      ; list: function call
(if t 1 2)   ; list: special form
```

### Function calls

In Python: `function(arg1, arg2)`
In Emacs Lisp: `(function arg1 arg2)`

The function or operator **always** goes first, inside the parentheses. No exceptions.

```python
# Python
print("hello")
len([1, 2, 3])
1 + 2 + 3
x = 5
if x > 3:
    print("big")
```

```elisp
;; Emacs Lisp
(message "hello")
(length '(1 2 3))
(+ 1 2 3)
(setq x 5)
(if (> x 3)
    (message "big"))
```

### No operator precedence

Because every expression has explicit parentheses, there is no precedence to remember:

```python
# Python - need to know precedence
result = 2 + 3 * 4    # = 14, not 20
```

```elisp
;; Emacs Lisp - explicit grouping
(+ 2 (* 3 4))   ; = 14, obviously
(* (+ 2 3) 4)   ; = 20, obviously
```

### Comments

```elisp
;; Double semicolon for top-level comments (most common)
; Single semicolon for inline or marginal comments
;;; Triple semicolon for section headers / file headers

(defun foo (x)
  ;; This comment explains the function body
  (* x 2))   ; inline comment

;;; Section: Utility Functions
;;; These are used throughout the package.
```

Python equivalent:

```python
# regular comment
x = 1  # inline comment
```

### Quoting

This has no direct Python equivalent. When Emacs sees a symbol, it tries to evaluate it (look up its value). Quoting prevents evaluation:

```elisp
foo          ; evaluates foo → its variable value
'foo         ; the symbol foo itself, not its value

(+ 1 2)      ; evaluates the expression → 3
'(+ 1 2)     ; the list (+ 1 2) as data, not evaluated
```

Think of `'thing` as `"thing"` in Python — you're saying "treat this as data, not code":

```python
# Python - strings are data, not evaluated
x = "foo"        # the string "foo"
x = foo          # the value of variable foo
```

```elisp
;; Emacs Lisp
(setq x 'foo)    ; the symbol foo
(setq x foo)     ; the value of variable foo
```

---

## 4. Data Types

### Comparison Table

| Python              | Emacs Lisp            | Notes                            |
| ------------------- | --------------------- | -------------------------------- |
| `int`               | `integer`             | arbitrary precision via bignum   |
| `float`             | `float`               | IEEE 754 double                  |
| `str`               | `string`              | mutable in-place, unlike Python  |
| `bool` (True/False) | `t` / `nil`           | nil is also empty list           |
| `None`              | `nil`                 | same as false                    |
| `list`              | `list`                | linked list, not array           |
| `tuple`             | `vector`              | fixed-size, random access        |
| `dict`              | `hash-table` or alist | two different things             |
| `set`               | no built-in           | use hash-table                   |
| class instance      | record / EIEIO object |                                  |
| function            | function / closure    |                                  |
| `bytes`             | `string`              | strings can hold arbitrary bytes |

### Numbers

```elisp
42          ; integer
-7          ; negative integer
3.14        ; float
1.0e10      ; scientific notation
#b1010      ; binary → 10
#o17        ; octal  → 15
#xff        ; hex    → 255

(+ 1 2)     ; → 3
(- 10 3)    ; → 7
(* 4 5)     ; → 20
(/ 10 3)    ; → 3  (integer division when both args are integers)
(/ 10 3.0)  ; → 3.3333...
(% 10 3)    ; → 1  (modulo)
(expt 2 8)  ; → 256
(sqrt 16.0) ; → 4.0
(abs -5)    ; → 5
(max 1 2 3) ; → 3
(min 1 2 3) ; → 1
```

```python
# Python equivalents
10 // 3    # integer division
10 % 3     # modulo
2 ** 8     # exponentiation
```

### Booleans and nil

```elisp
t           ; true
nil         ; false (also: empty list, also: null)

;; IMPORTANT: anything that is not nil is true
;; 0, "", '(), 0.0 are ALL true in Emacs Lisp!
(if 0   "yes" "no")   ; → "yes"  (unlike Python!)
(if ""  "yes" "no")   ; → "yes"  (unlike Python!)
(if nil "yes" "no")   ; → "no"

;; Only nil and () are false
(if '() "yes" "no")   ; → "no"  ('() is the same as nil)
```

```python
# Python - many things are falsy
bool(0)    # False
bool("")   # False
bool([])   # False
bool(None) # False
```

### Symbols

Symbols are a type that Python doesn't have. They are interned names — there is exactly one object for each name:

```elisp
'hello        ; the symbol hello
'my-function  ; symbols can contain hyphens (preferred over underscores)
'some/thing   ; can contain slashes
:keyword      ; keyword symbol — evaluates to itself

;; Symbols are used as:
;; - variable names
;; - function names
;; - keys in association lists
;; - enumeration values
;; - mode names

(symbolp 'foo)          ; → t
(symbol-name 'foo)      ; → "foo"  (convert to string)
(intern "foo")          ; → foo    (convert string to symbol)
```

### Strings

```elisp
"hello"                    ; basic string
"line1\nline2"             ; escape sequences work
"say \"hello\""            ; escaped quotes

;; String operations
(length "hello")            ; → 5
(concat "hello" " " "world") ; → "hello world"
(substring "hello" 1 3)     ; → "el"  (like Python's "hello"[1:3])
(upcase "hello")            ; → "HELLO"
(downcase "HELLO")          ; → "hello"
(string= "foo" "foo")       ; → t  (equality)
(string< "abc" "abd")       ; → t  (less than)
(string-match "lo" "hello") ; → 3  (like Python's re.search, returns position)
(replace-regexp-in-string "o" "0" "hello") ; → "hell0"

;; Format strings (like Python's str.format or f-strings)
(format "Hello, %s! You are %d years old." "Alice" 30)
; → "Hello, Alice! You are 30 years old."

;; Python equivalent:
;; f"Hello, {name}! You are {age} years old."
;; "Hello, {}! You are {} years old.".format(name, age)
```

Format specifiers:

```
%s  - any object (calls prin1-to-string or just inserts string)
%S  - any object, with quotes (prin1)
%d  - integer
%f  - float
%e  - scientific notation
%g  - shorter of %f and %e
%c  - character
```

---

## 5. Variables and Binding

### Global Variables

```python
# Python
x = 42
MY_CONSTANT = "hello"
```

```elisp
;; Emacs Lisp

;; setq: set a variable (creates it if needed, but bad practice globally)
(setq x 42)

;; defvar: declare a global variable with documentation
(defvar my-variable 42
  "Documentation string for my-variable.")

;; defconst: declare a constant (convention only, not enforced)
(defconst my-constant "hello"
  "Documentation for my-constant.")

;; defcustom: user-customizable variable (appears in M-x customize)
(defcustom my-setting t
  "Whether to enable my feature."
  :type 'boolean
  :group 'my-package)
```

### Local Variables

```python
# Python - local variables in functions are automatic
def foo():
    x = 10  # local to foo
    return x
```

```elisp
;; Emacs Lisp - local variables require explicit let or let*

;; let: all bindings happen simultaneously
(let ((x 10)
      (y 20))
  (+ x y))   ; → 30

;; let*: bindings happen sequentially (can reference earlier bindings)
(let* ((x 10)
       (y (* x 2)))   ; can use x here
  (+ x y))   ; → 30

;; Python equivalent of let*:
;; x = 10
;; y = x * 2
;; result = x + y
```

Common mistake from Python:

```elisp
;; WRONG: trying to reference x in the same let binding
(let ((x 10)
      (y (* x 2)))   ; ERROR: x is not bound yet in let
  y)

;; RIGHT: use let*
(let* ((x 10)
       (y (* x 2)))
  y)   ; → 20
```

### Buffer-Local Variables

This has no Python equivalent. Variables can be **local to a specific buffer**:

```elisp
;; Declare a variable that can be buffer-local
(defvar-local my-mode-state nil
  "State variable local to each buffer.")

;; Or make any variable buffer-local in the current buffer
(make-local-variable 'some-variable)
(setq some-variable "local value")

;; permanent-local: survives major mode changes
(put 'my-variable 'permanent-local t)
```

### Dynamic vs Lexical Binding Deep Dive

```elisp
;;; -*- lexical-binding: t; -*-

;; With lexical binding (modern, recommended):
(let ((x 10))
  (defun get-x () x))   ; closes over x = 10

(get-x)   ; → 10, always, because x was captured at definition time

;; Without lexical binding (dynamic, old behavior):
;; get-x would look up x in the CALL STACK at call time
;; If x happens to be bound somewhere in the call chain, it uses that value
;; This leads to action-at-a-distance bugs
```

---

## 6. Functions

### Defining Functions

```python
def greet(name, greeting="Hello"):
    """Greet someone."""
    return f"{greeting}, {name}!"
```

```elisp
(defun greet (name &optional greeting)
  "Greet someone.
NAME is the person's name.
GREETING defaults to \"Hello\"."
  (format "%s, %s!"
          (or greeting "Hello")
          name))

(greet "Alice")           ; → "Hello, Alice!"
(greet "Bob" "Hi")        ; → "Hi, Bob!"
```

### Parameter Types

```elisp
;; Required parameters
(defun foo (a b c) ...)

;; Optional parameters
(defun foo (a &optional b c) ...)

;; Rest parameters (like *args in Python)
(defun foo (a &rest args) ...)
;; args will be a list of remaining arguments

;; Combined
(defun foo (a b &optional c &rest rest) ...)

;; Calling with apply (like Python's func(*args))
(apply #'+ '(1 2 3 4))    ; → 10
(apply #'+ 1 2 '(3 4))    ; → 10  (some args before the list)
```

### Anonymous Functions (Lambdas)

```python
# Python
square = lambda x: x * x
add = lambda x, y: x + y
```

```elisp
;; Emacs Lisp
(lambda (x) (* x x))
(lambda (x y) (+ x y))

;; Assign to a variable
(setq square (lambda (x) (* x x)))

;; Call it (function namespace requires funcall)
(funcall square 5)   ; → 25

;; More idiomatic: use function shorthand #'
(setq square #'(lambda (x) (* x x)))
;; #' is like ' but for functions — ensures proper lexical closure capture
```

### Higher-Order Functions

```python
# Python
numbers = [1, 2, 3, 4, 5]
squares = list(map(lambda x: x**2, numbers))
evens = list(filter(lambda x: x % 2 == 0, numbers))
total = sum(numbers)
```

```elisp
;; Emacs Lisp
(setq numbers '(1 2 3 4 5))

(mapcar (lambda (x) (* x x)) numbers)
; → (1 4 9 16 25)

(seq-filter (lambda (x) (= (% x 2) 0)) numbers)
; → (2 4)

(apply #'+ numbers)
; → 15

;; Also available:
(seq-map    #'function list)     ; like mapcar
(seq-reduce #'function list 0)  ; like functools.reduce
(seq-every-p #'evenp '(2 4 6))  ; like all()
(seq-some   #'oddp  '(1 2 3))   ; like any()
```

### Closures

```python
def make_adder(n):
    def adder(x):
        return x + n  # closes over n
    return adder

add5 = make_adder(5)
add5(3)  # → 8
```

```elisp
;;; -*- lexical-binding: t; -*-

(defun make-adder (n)
  (lambda (x) (+ x n)))   ; closes over n

(setq add5 (make-adder 5))
(funcall add5 3)   ; → 8
```

Without `lexical-binding: t`, the closure would not capture `n` and this would fail.

### The `#'` Function Reader Syntax

```elisp
;; These three are equivalent ways to refer to a named function:
(mapcar 'upcase '("hello" "world"))     ; quoted symbol (works but not ideal)
(mapcar #'upcase '("hello" "world"))    ; function shorthand (preferred)
(mapcar (function upcase) ...)          ; explicit function special form

;; Use #' when:
;; - passing named functions as arguments
;; - storing functions in variables
;; - anywhere you want the compiler to know it's a function reference

;; Use lambda when defining anonymous functions:
(mapcar (lambda (x) (* x 2)) '(1 2 3))
```

---

## 7. Control Flow

### If / Cond

```python
# Python
if x > 0:
    print("positive")
elif x < 0:
    print("negative")
else:
    print("zero")
```

```elisp
;; if: only two branches
(if (> x 0)
    (message "positive")
  (message "not positive"))   ; else branch

;; IMPORTANT: if takes exactly three forms: condition, then, else
;; The else form is optional but there can only be one of each

;; cond: multiple branches (like if/elif/else)
(cond
 ((> x 0) (message "positive"))
 ((< x 0) (message "negative"))
 (t        (message "zero")))   ; t is the else/default

;; when: if without else (returns nil if condition is false)
(when (> x 0)
  (message "positive")
  (do-something-else))   ; can have multiple forms in body

;; unless: inverted when
(unless (= x 0)
  (message "not zero"))
```

### Progn (Sequencing)

In Python, a function body can have multiple statements. In Emacs Lisp, most forms take a single expression. `progn` sequences multiple expressions:

```python
def foo():
    print("first")
    print("second")
    return 42
```

```elisp
(defun foo ()
  ;; defun body is implicitly a progn
  (message "first")
  (message "second")
  42)

;; But if branch only takes one form, so use progn:
(if condition
    (progn
      (do-thing-1)
      (do-thing-2))
  (do-else))

;; when and unless implicitly include progn:
(when condition
  (do-thing-1)    ; multiple forms allowed
  (do-thing-2))
```

### Loops

```python
# Python
for i in range(5):
    print(i)

for item in my_list:
    print(item)

i = 0
while i < 10:
    print(i)
    i += 1
```

```elisp
;; dotimes: like range(n)
(dotimes (i 5)
  (message "%d" i))   ; prints 0 1 2 3 4

;; dolist: like for item in list
(dolist (item '(a b c))
  (message "%s" item))

;; while: explicit while loop
(let ((i 0))
  (while (< i 10)
    (message "%d" i)
    (setq i (1+ i))))

;; cl-loop: powerful loop macro (like Python's itertools + for in one)
(require 'cl-lib)
(cl-loop for i from 0 to 9
         do (message "%d" i))

(cl-loop for x in '(1 2 3 4 5)
         when (> x 2)
         collect (* x x))
; → (9 16 25)

;; Named return value from dolist
(let ((result nil))
  (dolist (x '(1 2 3 4 5) result)   ; result is the return value
    (push (* x x) result)))
; → (25 16 9 4 1)  (reversed because of push)
```

### Pattern Matching with pcase

This is like Python's `match` statement (Python 3.10+):

```python
# Python 3.10+
match command:
    case "quit":
        quit_game()
    case "go" if direction in valid_directions:
        go(direction)
    case ["go", direction]:
        go(direction)
    case _:
        print("unknown command")
```

```elisp
;; Emacs Lisp pcase
(pcase command
  ("quit"
   (quit-game))
  ((pred stringp)
   (message "got a string: %s" command))
  (`(go ,direction)           ; pattern matching on lists
   (go direction))
  ((and (pred numberp) x)     ; bind and test
   (message "got number %d" x))
  (_
   (message "unknown command")))

;; pcase-let: destructuring bind
(pcase-let ((`(,a ,b ,c) '(1 2 3)))
  (+ a b c))   ; → 6

;; seq-let: simpler destructuring
(seq-let (a b c) '(1 2 3)
  (+ a b c))   ; → 6
```

---

## 8. Lists and Sequences

Lists are the fundamental data structure in Lisp. Unlike Python lists (arrays), Lisp lists are **linked lists**.

### Basic List Operations

```python
# Python
lst = [1, 2, 3, 4, 5]
lst[0]          # first element
lst[1:]         # all but first
lst.append(6)   # add to end (mutates)
[0] + lst       # add to front (new list)
len(lst)        # length
1 in lst        # membership
```

```elisp
;; Emacs Lisp
(setq lst '(1 2 3 4 5))

(car lst)             ; → 1       first element (like lst[0])
(cdr lst)             ; → (2 3 4 5) rest of list (like lst[1:])
(cadr lst)            ; → 2       second element (car of cdr)
(caddr lst)           ; → 3       third element
(nth 2 lst)           ; → 3       nth element (0-indexed)

(cons 0 lst)          ; → (0 1 2 3 4 5)  prepend (new list)
(append lst '(6))     ; → (1 2 3 4 5 6)  append (new list)

(length lst)          ; → 5
(member 3 lst)        ; → (3 4 5)  truthy if 3 is in lst (returns tail)
(memq 3 lst)          ; like member but uses eq for comparison

;; car/cdr shorthand: up to 4 levels
;; c[ad]+r where a=car, d=cdr
(cadr '(1 2 3))       ; → 2    (car (cdr '(1 2 3)))
(cddr '(1 2 3))       ; → (3)  (cdr (cdr '(1 2 3)))
(caddr '(1 2 3))      ; → 3    (car (cdr (cdr '(1 2 3))))
```

### Building Lists

```python
# Python
result = []
for i in range(5):
    result.append(i * 2)
```

```elisp
;; Common pattern: push then nreverse
(let ((result nil))
  (dotimes (i 5)
    (push (* i 2) result))   ; push adds to front
  (nreverse result))         ; reverse to restore order
; → (0 2 4 6 8)

;; Or use mapcar for transformations
(mapcar (lambda (i) (* i 2)) '(0 1 2 3 4))
; → (0 2 4 6 8)

;; list: create a list from arguments
(list 1 2 3)     ; → (1 2 3)
(list 'a 'b 'c)  ; → (a b c)
```

### Destructive Operations

```elisp
;; nreverse: reverse in place (destructive)
(setq x '(1 2 3))
(nreverse x)      ; → (3 2 1)
;; WARNING: x is now undefined/garbage after this

;; setcar / setcdr: mutate list cells
(setq x '(1 2 3))
(setcar x 99)     ; → 99, x is now (99 2 3)
(setcdr x '(a b)) ; → (a b), x is now (99 a b)
```

### seq Library (Modern, Recommended)

The `seq` library provides Python-like sequence operations that work on lists, vectors, and strings:

```elisp
(require 'seq)

(seq-length '(1 2 3))          ; → 3
(seq-elt '(a b c) 1)           ; → b
(seq-first '(1 2 3))           ; → 1
(seq-rest '(1 2 3))            ; → (2 3)
(seq-take '(1 2 3 4 5) 3)      ; → (1 2 3)  like lst[:3]
(seq-drop '(1 2 3 4 5) 2)      ; → (3 4 5)  like lst[2:]
(seq-subseq '(1 2 3 4 5) 1 3)  ; → (2 3)    like lst[1:3]
(seq-reverse '(1 2 3))         ; → (3 2 1)
(seq-sort #'< '(3 1 2))        ; → (1 2 3)
(seq-contains-p '(1 2 3) 2)    ; → t
(seq-position '(a b c) 'b)     ; → 1
(seq-count #'oddp '(1 2 3 4 5)); → 3  count matching elements
(seq-uniq '(1 2 1 3 2))        ; → (1 2 3)  remove duplicates
(seq-concatenate 'list '(1 2) '(3 4)) ; → (1 2 3 4)
```

### Vectors

Vectors are like Python tuples or lists but with O(1) random access:

```elisp
[1 2 3 4 5]         ; literal vector

(vector 1 2 3)      ; create vector
(make-vector 5 0)   ; vector of 5 zeros: [0 0 0 0 0]

(aref [a b c] 1)    ; → b  (array reference)
(aset v 1 'x)       ; mutate element (like v[1] = 'x')
(length [a b c])    ; → 3
(vconcat [1 2] [3 4]) ; → [1 2 3 4]  concatenate vectors
```

---

## 9. Hash Tables and Association Lists

### Association Lists (alists)

Alists are lists of `(key . value)` pairs. They are simple but O(n) for lookup. Used extensively in Emacs Lisp.

```python
# Python dict (sort of)
d = {"name": "Alice", "age": 30}
d["name"]   # "Alice"
```

```elisp
;; Association list
(setq person '((name . "Alice")
               (age  . 30)
               (city . "NYC")))

;; Lookup
(assoc 'name person)        ; → (name . "Alice")  returns the pair
(cdr (assoc 'name person))  ; → "Alice"           just the value
(alist-get 'name person)    ; → "Alice"           convenience function

;; assoc uses equal for comparison
;; assq uses eq (faster, for symbols and integers)
(assq 'age person)          ; → (age . 30)

;; Add/update (alists are typically prepended, shadowing old keys)
(push '(email . "alice@example.com") person)
;; Now person has email at front, and looking up email finds it first

;; alist-get with default
(alist-get 'missing person nil)    ; → nil (default)
(alist-get 'missing person 'none)  ; → none

;; Remove a key
(setq person (assoc-delete-all 'city person))

;; setf with alist-get (gv.el setter)
(setf (alist-get 'age person) 31)  ; update age to 31
```

### Hash Tables

For O(1) lookup. Like Python dicts:

```python
# Python
d = {}
d["key"] = "value"
d.get("key", "default")
"key" in d
del d["key"]
for k, v in d.items():
    print(k, v)
```

```elisp
;; Create
(setq h (make-hash-table :test 'equal))  ; equal compares by value
;; other :test options: eq (by identity), eql (numbers and symbols)

;; Set
(puthash "key" "value" h)
(puthash 'symbol-key 42 h)

;; Get
(gethash "key" h)           ; → "value"
(gethash "missing" h)       ; → nil
(gethash "missing" h 'none) ; → none (default value)

;; Membership
(gethash "key" h)   ; truthy if key exists

;; Delete
(remhash "key" h)

;; Iterate
(maphash (lambda (key value)
           (message "%s → %s" key value))
         h)

;; Size
(hash-table-count h)

;; :test options explained:
;; 'eq    - same object (use for symbols, integers)
;; 'eql   - same value for numbers, same object otherwise
;; 'equal - same value (deep equality, use for strings)
```

---

## 10. Strings

### String Operations Reference

```python
s = "Hello, World!"
len(s)                  # 13
s.upper()               # "HELLO, WORLD!"
s.lower()               # "hello, world!"
s[7:12]                 # "World"
s.split(", ")           # ["Hello", "World!"]
", ".join(["a", "b"])   # "a, b"
s.strip()               # remove whitespace
s.startswith("Hello")   # True
s.endswith("!")         # True
s.replace("o", "0")     # "Hell0, W0rld!"
"World" in s            # True
```

```elisp
(setq s "Hello, World!")

(length s)                          ; → 13
(upcase s)                          ; → "HELLO, WORLD!"
(downcase s)                        ; → "hello, world!"
(substring s 7 12)                  ; → "World"
(split-string s ", ")               ; → ("Hello" "World!")
(mapconcat #'identity '("a" "b") ", ") ; → "a, b"
(string-trim s)                     ; → remove whitespace
(string-prefix-p "Hello" s)         ; → t
(string-suffix-p "!" s)             ; → t
(replace-regexp-in-string "o" "0" s); → "Hell0, W0rld!"
(string-match-p "World" s)          ; → truthy (position)
(string-search "World" s)           ; → 7 (position, Emacs 28+)
```

### Regular Expressions

Emacs uses its own regex syntax (not PCRE):

```
.       any char
*       zero or more (greedy)
+       one or more (greedy)
?       zero or one
^       start of string/line
$       end of string/line
[abc]   character class
[^abc]  negated class
\(  \)  grouping (note: backslash required!)
\|      alternation
\w      word character
\b      word boundary
```

```elisp
;; Test if string matches regex
(string-match "hel+o" "hello")        ; → 0 (match start position)
(string-match "hel+o" "goodbye")      ; → nil (no match)
(string-match-p "hel+o" "hello")      ; → 0 (doesn't modify match data)

;; Extract groups
(string-match "\\(\\w+\\), \\(\\w+\\)" "Smith, John")
(match-string 1 "Smith, John")  ; → "Smith"
(match-string 2 "Smith, John")  ; → "John"

;; Replace
(replace-regexp-in-string "\\b\\w" #'upcase "hello world")
; → "Hello World"  (capitalize first letter of each word)

;; Split on regex
(split-string "a  b  c" "\\s-+")   ; → ("a" "b" "c")
```

### String Formatting

```elisp
;; format is like Python's % formatting or str.format()
(format "Name: %s, Age: %d" "Alice" 30)
; → "Name: Alice, Age: 30"

;; message also formats but prints to echo area
(message "Processing %d of %d items..." current total)

;; prin1-to-string: convert any Lisp object to string
(prin1-to-string '(1 2 3))    ; → "(1 2 3)"
(prin1-to-string "hello")     ; → "\"hello\""  (with quotes)

;; princ-to-string: like prin1 but without quotes for strings
(with-output-to-string
  (princ "hello"))             ; → "hello"

;; number-to-string / string-to-number
(number-to-string 42)         ; → "42"
(string-to-number "42")       ; → 42
(string-to-number "3.14")     ; → 3.14
(string-to-number "0xff" 16)  ; → 255
```

---

## 11. Macros

Macros are the most powerful feature Emacs Lisp inherits from Lisp. They are transformations on code at **read time**, before evaluation. Python has no direct equivalent (decorators are somewhat similar but much weaker).

### Why Macros?

In Python, you cannot define new syntax or new control structures. In Emacs Lisp you can.

```python
# Python - you cannot define something like this yourself:
with open("file") as f:
    data = f.read()
# 'with' is baked into the language
```

```elisp
;; Emacs Lisp - you CAN define new control structures:
(defmacro with-resource (binding &rest body)
  "Execute BODY with resource BINDING, cleaning up after."
  `(let ((resource ,(cadr binding)))
     (unwind-protect
         (progn ,@body)
       (cleanup-resource resource))))

;; Now you can use it like a built-in:
(with-resource (r (acquire-resource))
  (use-resource r)
  (more-work r))
```

### The Backquote Template

Backquote (`` ` ``) is to macros what f-strings are to Python strings — it creates a template with interpolation:

```python
# Python f-string
name = "Alice"
f"Hello, {name}!"   # → "Hello, Alice!"
```

```elisp
;; Emacs Lisp backquote
(setq name "Alice")
`(hello ,name)      ; → (hello "Alice")   , splices in value
`(1 2 ,@'(3 4) 5)  ; → (1 2 3 4 5)       ,@ splices in a list
```

### Simple Macro Examples

```elisp
;; swap: like Python's a, b = b, a
(defmacro swap (a b)
  "Swap the values of variables A and B."
  `(let ((tmp ,a))
     (setq ,a ,b)
     (setq ,b tmp)))

(let ((x 1) (y 2))
  (swap x y)
  (list x y))   ; → (2 1)

;; while-let: loop while condition is truthy, binding its value
(defmacro while-let (binding &rest body)
  (declare (indent 1))
  `(let ((,(car binding) ,(cadr binding)))
     (while ,(car binding)
       ,@body
       (setq ,(car binding) ,(cadr binding)))))
```

### Checking Macro Expansion

```elisp
;; Use macroexpand to see what a macro generates
(macroexpand '(when (> x 0)
                (message "positive")
                (do-thing)))
; → (if (> x 0) (progn (message "positive") (do-thing)))

;; macroexpand-all expands nested macros too
(macroexpand-all '(while-let (line (read-line))
                   (process line)))
```

### Macro Hygiene

A common macro pitfall is **variable capture** — the macro accidentally uses a variable name from the calling context:

```elisp
;; BAD: uses 'tmp' which might clash with caller's 'tmp'
(defmacro bad-swap (a b)
  `(let ((tmp ,a))
     (setq ,a ,b)
     (setq ,b tmp)))

;; GOOD: use gensym to generate a unique name
(defmacro good-swap (a b)
  (let ((tmp (gensym "tmp")))
    `(let ((,tmp ,a))
       (setq ,a ,b)
       (setq ,b ,tmp))))
```

---

## 12. Error Handling

### Basic Error Handling

```python
try:
    result = risky_operation()
except ValueError as e:
    print(f"Value error: {e}")
except (TypeError, RuntimeError) as e:
    print(f"Other error: {e}")
else:
    print("Success!")
finally:
    cleanup()
```

```elisp
;; condition-case is the try/except equivalent
(condition-case err
    (risky-operation)           ; protected form
  (error                        ; catch any error
   (message "Error: %s" (error-message-string err)))
  (file-error                   ; catch specific error type
   (message "File error: %s" (cadr err))))

;; With multiple handlers:
(condition-case err
    (progn
      (do-thing-1)
      (do-thing-2))
  (wrong-type-argument
   (message "Wrong type: %s" err))
  (error
   (message "General error: %s" (error-message-string err))))

;; unwind-protect is the try/finally equivalent
(unwind-protect
    (risky-operation)           ; protected form
  (cleanup))                    ; always runs (like finally)

;; Combined:
(condition-case err
    (unwind-protect
        (risky-operation)
      (cleanup))                ; always runs
  (error
   (handle-error err)))         ; runs on error
```

### Signaling Errors

```python
raise ValueError("something went wrong")
raise RuntimeError("bad state")
```

```elisp
(error "Something went wrong: %s" details)  ; like raise Exception(...)
(user-error "Invalid input: %s" input)       ; user-facing error (no debug)

;; signal: like raise with a specific error type
(signal 'wrong-type-argument (list 'stringp value))
(signal 'file-error (list "Cannot open file" filename))

;; Define custom error type:
(define-error 'my-error "My custom error" 'error)
(signal 'my-error '("specific message"))
```

### Common Error Types

```
error              base error type
wrong-type-argument  passed wrong type
wrong-number-of-arguments  wrong arg count
void-variable      unbound variable
void-function      undefined function
args-out-of-range  index out of bounds
file-error         file operation failed
search-failed      regexp search failed
user-error         user-facing (no backtrace)
```

---

## 13. Object System

### EIEIO (Enhanced Implementation of Emacs Interpreted Objects)

EIEIO is Emacs's CLOS-like object system. Think of it as Python's classes but with multiple inheritance done via method specialization.

```python
class Animal:
    def __init__(self, name, sound):
        self.name = name
        self.sound = sound

    def speak(self):
        return f"{self.name} says {self.sound}"

class Dog(Animal):
    def __init__(self, name):
        super().__init__(name, "woof")

    def fetch(self):
        return f"{self.name} fetches!"
```

```elisp
(require 'eieio)

(defclass animal ()
  ((name  :initarg :name  :accessor animal-name)
   (sound :initarg :sound :accessor animal-sound))
  "An animal.")

(cl-defmethod speak ((a animal))
  (format "%s says %s" (animal-name a) (animal-sound a)))

(defclass dog (animal)
  ()
  "A dog.")

(cl-defmethod initialize-instance :after ((d dog) &rest _)
  (setf (animal-sound d) "woof"))

(cl-defmethod fetch ((d dog))
  (format "%s fetches!" (animal-name d)))

;; Usage
(setq rex (make-instance 'dog :name "Rex"))
(speak rex)   ; → "Rex says woof"
(fetch rex)   ; → "Rex fetches!"
```

### Records (Simpler Alternative)

For simple data structures without methods, use `cl-defstruct`:

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: float
    y: float

p = Point(1.0, 2.0)
p.x   # 1.0
```

```elisp
(cl-defstruct point
  x y)

(setq p (make-point :x 1.0 :y 2.0))
(point-x p)              ; → 1.0
(setf (point-y p) 3.0)   ; → 3.0 (mutation)
(point-p p)              ; → t   (type predicate)

;; Can add default values and type checks:
(cl-defstruct (point (:constructor make-point (x y)))
  (x 0.0 :type float)
  (y 0.0 :type float))
```

---

## 14. Buffers and Text

This is where Emacs Lisp truly diverges from Python. **Buffers are first-class objects** and most interesting operations work on them.

### Buffer Basics

```elisp
;; Create a buffer
(get-buffer-create "*my-buffer*")    ; get or create
(generate-new-buffer "*my-buffer*")  ; always creates new, unique name

;; Switch to a buffer
(with-current-buffer "*my-buffer*"
  ;; code here runs as if *my-buffer* is current
  (insert "hello\n"))

;; Get current buffer
(current-buffer)

;; Check if buffer exists
(get-buffer "*my-buffer*")   ; → buffer or nil

;; Kill (close) a buffer
(kill-buffer "*my-buffer*")

;; List all buffers
(buffer-list)
```

### Writing to Buffers

```elisp
(with-current-buffer (get-buffer-create "*output*")
  (erase-buffer)                        ; clear buffer
  (insert "Line 1\n")                   ; insert at point
  (insert (format "Value: %d\n" 42))    ; with formatting
  (goto-char (point-min))               ; move to beginning
  (display-buffer (current-buffer)))    ; show it
```

### Reading from Buffers

```elisp
(with-current-buffer some-buffer
  ;; Get entire buffer contents
  (buffer-string)

  ;; Get region
  (buffer-substring (point-min) (point-max))

  ;; Get current line
  (thing-at-point 'line)

  ;; Get word at point
  (thing-at-point 'word)

  ;; Search forward
  (re-search-forward "pattern" nil t)   ; t means don't error if not found
  (match-string 0)                      ; → matched text

  ;; Move around
  (goto-char (point-min))               ; beginning
  (goto-char (point-max))               ; end
  (forward-line 1)                      ; next line
  (beginning-of-line)
  (end-of-line)
  (forward-word 1))
```

### Point and Mark

The **point** is the cursor position (a number from 1 to `point-max`). The **mark** is a saved position used to define a region.

```elisp
(point)           ; current cursor position
(point-min)       ; minimum valid position (usually 1)
(point-max)       ; maximum valid position (= buffer size + 1)

(save-excursion   ; save and restore point
  (goto-char (point-min))
  (do-something))
;; point is back where it was after save-excursion
```

### The Minibuffer (User Input)

```python
# Python
name = input("Enter your name: ")
choice = input(f"Enter choice ({'/'.join(options)}): ")
```

```elisp
;; Read a string
(read-string "Enter your name: ")

;; Read with completion
(completing-read "Choose: " '("option1" "option2" "option3"))

;; Read a file name
(read-file-name "Open file: ")

;; Read a number
(read-number "Enter a number: " 42)  ; 42 is default

;; Yes or no question
(yes-or-no-p "Are you sure? ")   ; requires "yes" or "no"
(y-or-n-p "Proceed? ")           ; accepts "y" or "n"
```

---

## 15. Processes and Async

### Synchronous Process Execution

```python
import subprocess
result = subprocess.run(["ls", "-la"], capture_output=True, text=True)
print(result.stdout)
```

```elisp
;; call-process: synchronous, like subprocess.run
(call-process "ls" nil t nil "-la")   ; output goes to current buffer

;; with output to string
(with-temp-buffer
  (call-process "ls" nil t nil "-la")
  (buffer-string))

;; process-lines: get output as list of lines
(process-lines "git" "log" "--oneline" "-5")
; → ("abc1234 first commit" "def5678 second commit" ...)

;; call-process-region: pipe buffer region to process
(call-process-region (point-min) (point-max)
                     "python3" nil t nil "-c" "import sys; print(sys.stdin.read().upper())")
```

### Asynchronous Process

```python
import subprocess
proc = subprocess.Popen(["long-running-command"],
                        stdout=subprocess.PIPE)
# ... do other things ...
output, _ = proc.communicate()
```

```elisp
;; make-process: async, like subprocess.Popen
(make-process
 :name    "my-process"
 :buffer  (get-buffer-create "*my-process-output*")
 :command '("long-running-command" "arg1" "arg2")
 :sentinel
 (lambda (proc status)
   (when (string= status "finished\n")
     (message "Process completed!")
     (with-current-buffer (process-buffer proc)
       (do-something-with-output))))
 :filter
 (lambda (proc output)
   ;; Called whenever process produces output
   (with-current-buffer (process-buffer proc)
     (goto-char (point-max))
     (insert output))))

;; start-process: simpler async process
(start-process "my-proc" "*output*" "command" "arg1" "arg2")
```

### Timers

```python
import threading
timer = threading.Timer(5.0, my_function)
timer.start()
```

```elisp
;; run after delay
(run-with-timer 5.0 nil #'my-function)

;; repeat every N seconds
(run-with-timer 0 30 #'my-refresh-function)

;; cancel a timer
(setq my-timer (run-with-timer 5.0 nil #'foo))
(cancel-timer my-timer)

;; run when idle
(run-with-idle-timer 2.0 nil #'my-function)  ; after 2s of idleness
```

---

## 16. Modules and Libraries

### Python Import vs Emacs Require

```python
# Python
import os
import os.path
from pathlib import Path
from functools import partial
```

```elisp
;; Emacs Lisp
(require 'cl-lib)         ; common lisp extensions (always useful)
(require 'seq)            ; sequence functions
(require 'subr-x)         ; extra string/list utilities
(require 'map)            ; map/alist/hash-table abstraction

;; Optional require (don't error if not available)
(require 'some-package nil t)    ; t = noerror

;; With autoloads (lazy loading — like Python's lazy import)
(autoload 'some-function "some-file" "Docstring." t)
;; some-function won't load until actually called
```

### Providing a Library

```python
# Python - nothing needed, module system is automatic
```

```elisp
;; At end of your .el file:
(provide 'my-library)

;; Other files can then:
(require 'my-library)
```

### cl-lib: Common Lisp Compatibility

`cl-lib` is essential. It provides many functions Python programmers expect:

```elisp
(require 'cl-lib)

;; cl-loop: powerful iteration
(cl-loop for i from 1 to 5 collect (* i i))
; → (1 4 9 16 25)

;; cl-remove-if / cl-remove-if-not (like filter)
(cl-remove-if #'oddp '(1 2 3 4 5))     ; → (2 4)
(cl-remove-if-not #'oddp '(1 2 3 4 5)) ; → (1 3 5)

;; cl-find-if (like next(filter(...)))
(cl-find-if #'evenp '(1 3 5 6 7))      ; → 6

;; cl-position (like list.index())
(cl-position 'c '(a b c d))            ; → 2

;; cl-count-if
(cl-count-if #'oddp '(1 2 3 4 5))      ; → 3

;; cl-sort (non-destructive sort)
(cl-sort (copy-sequence '(3 1 4 1 5)) #'<)  ; → (1 1 3 4 5)

;; cl-reduce (like functools.reduce)
(cl-reduce #'+ '(1 2 3 4 5))           ; → 15
(cl-reduce #'max '(3 1 4 1 5))         ; → 5

;; cl-case (like case/switch/match)
(cl-case day
  (monday  (message "Start of week"))
  (friday  (message "End of week"))
  (t       (message "Midweek")))

;; cl-destructuring-bind (like tuple unpacking)
(cl-destructuring-bind (a b &rest rest) '(1 2 3 4 5)
  (list a b rest))   ; → (1 2 (3 4 5))
```

---

## 17. Testing

### ERT: Emacs Regression Testing

ERT is the built-in test framework. It's similar to Python's `unittest`:

```python
import unittest

class TestMyFunctions(unittest.TestCase):
    def test_addition(self):
        self.assertEqual(add(2, 3), 5)

    def test_greeting(self):
        self.assertEqual(greet("Alice"), "Hello, Alice!")

    def test_error(self):
        with self.assertRaises(ValueError):
            parse_number("not a number")

if __name__ == "__main__":
    unittest.main()
```

```elisp
(require 'ert)

(ert-deftest test-addition ()
  "Test that addition works."
  (should (= (+ 2 3) 5))
  (should-not (= (+ 2 3) 6)))

(ert-deftest test-greeting ()
  "Test greeting function."
  (should (string= (greet "Alice") "Hello, Alice!")))

(ert-deftest test-error ()
  "Test that errors are signaled correctly."
  (should-error (parse-number "not a number") :type 'error))

;; Run tests:
;; M-x ert RET t RET    (run all tests)
;; M-x ert-run-tests-interactively

;; In batch mode (for CI):
;; emacs --batch -l my-package.el -l my-tests.el -f ert-run-tests-batch-and-exit
```

### Assertions Reference

```elisp
(should FORM)                    ; assert FORM is truthy
(should-not FORM)                ; assert FORM is nil
(should-error FORM)              ; assert FORM signals an error
(should-error FORM :type 'error-type)  ; assert specific error type

;; For equality testing, should uses equal by default
(should (equal (my-func 1) '(1 2 3)))
(should (string= (my-string-func) "expected"))
(should (= (my-number-func) 42))
```

---

## 18. Common Patterns

### Advice (Monkey Patching)

Python allows monkey patching but it's ad hoc. Emacs Lisp has a formal advice system:

```python
# Python monkey patching
original_func = some_module.some_function
def patched_func(*args, **kwargs):
    print("before")
    result = original_func(*args, **kwargs)
    print("after")
    return result
some_module.some_function = patched_func
```

```elisp
;; around advice: most like Python monkey patching
(defun my-around-advice (orig-fn &rest args)
  "Advice that wraps orig-fn."
  (message "before")
  (let ((result (apply orig-fn args)))
    (message "after")
    result))

(advice-add 'some-function :around #'my-around-advice)

;; Remove advice
(advice-remove 'some-function #'my-around-advice)

;; before advice: runs before the function
(advice-add 'some-function :before
            (lambda (&rest _) (message "about to call")))

;; after advice: runs after, receives result
(advice-add 'some-function :after
            (lambda (&rest _) (message "just called")))

;; filter-args: modify arguments
(advice-add 'some-function :filter-args
            (lambda (args) (cons (1+ (car args)) (cdr args))))

;; filter-return: modify return value
(advice-add 'some-function :filter-return
            (lambda (result) (* result 2)))
```

### Hooks

Hooks are lists of functions called at specific times. Like Python's event system or signals:

```python
# Python equivalent concept:
on_open_handlers = []
def register_open_handler(fn):
    on_open_handlers.append(fn)

def open_file():
    # ... open the file ...
    for handler in on_open_handlers:
        handler()
```

```elisp
;; Add to a hook
(add-hook 'after-init-hook #'my-startup-function)
(add-hook 'before-save-hook #'delete-trailing-whitespace)
(add-hook 'python-mode-hook
          (lambda ()
            (setq indent-tabs-mode nil)
            (setq python-indent-offset 4)))

;; Remove from a hook
(remove-hook 'after-init-hook #'my-startup-function)

;; Run a hook manually
(run-hooks 'my-custom-hook)

;; Buffer-local hook (only applies to current buffer)
(add-hook 'some-hook #'my-function nil t)   ; final t = local

;; Hooks are just variables containing lists of functions
(setq my-hook nil)   ; create a hook variable
(defvar my-hook nil "Runs when something happens.")
(run-hooks 'my-hook) ; runs all functions in my-hook
```

### Property Lists (plists)

```elisp
;; A plist is a flat list of key-value pairs
(setq my-plist '(:name "Alice" :age 30 :city "NYC"))

(plist-get my-plist :name)          ; → "Alice"
(plist-put my-plist :age 31)        ; returns new plist
(setq my-plist (plist-put my-plist :email "alice@example.com"))

(plist-member my-plist :city)       ; → (:city "NYC")  or nil

;; Symbol property lists (stored on the symbol itself)
(put 'my-symbol 'my-property "value")
(get 'my-symbol 'my-property)       ; → "value"
```

### Generalized Variables (setf)

`setf` allows setting "places" — not just variables but computed locations:

```python
# Python
lst[0] = 99
d["key"] = "new-value"
obj.attr = "new"
```

```elisp
;; setf works on many "places"
(setf (car my-list) 99)                    ; set first element
(setf (nth 2 my-list) 99)                  ; set nth element
(setf (aref my-vector 0) 99)              ; set vector element
(setf (gethash key my-hash) "new-value")  ; set hash table entry
(setf (alist-get key my-alist) "value")   ; set alist entry
(setf (point) 100)                         ; move point

;; You can define your own setf targets with gv-define-setter
```

---

## 19. Debugging

### Interactive Debugging

```elisp
;; Toggle debug on error (like Python's pdb.set_trace() but automatic)
(setq debug-on-error t)

;; Debug a specific function
(debug-on-entry 'my-function)
(cancel-debug-on-entry 'my-function)

;; Insert a breakpoint in code
(debug)   ; drops into debugger at this point
```

### The Debugger

When an error occurs with `debug-on-error` enabled, you enter the **Edebug** debugger:

```
Debugger commands:
  c  - continue
  d  - step (one expression at a time)
  b  - set breakpoint
  u  - remove breakpoint
  e  - evaluate expression in current context
  q  - quit debugger
  r  - return a value
  l  - list local variables
```

### Edebug: Stepping Debugger

```elisp
;; Instrument a function for stepping:
;; Place point inside defun and press: C-u C-M-x
;; Or: M-x edebug-defun

(defun my-function (x y)
  (let ((result (+ x y)))    ; pause here
    (* result 2)))            ; and here

;; Edebug key bindings while stepping:
;;   SPC  - step
;;   n    - next (step over)
;;   i    - step in
;;   o    - step out
;;   c    - continue
;;   G    - run to end
;;   e    - evaluate
;;   q    - quit
```

### Printing Debug Output

```python
print(f"Debug: x = {x}")
import pprint
pprint.pprint(complex_object)
```

```elisp
;; message: print to echo area and *Messages* buffer
(message "Debug: x = %S" x)

;; prin1: print to *standard-output*
(prin1 complex-object)

;; pp: pretty-print
(require 'pp)
(pp complex-object)

;; In a buffer:
(with-current-buffer (get-buffer-create "*debug*")
  (goto-char (point-max))
  (insert (format "%S\n" complex-object)))
```

---

## 20. Style and Conventions

### Naming Conventions

```elisp
;; Functions and variables use hyphen-separated-words (kebab-case)
;; NOT snake_case, NOT camelCase
(defun my-great-function (x) ...)
(defvar my-global-variable nil)

;; Package prefix to avoid name collisions
(defun kubed-list-pods () ...)     ; kubed- prefix
(defvar kubed--private-var nil)    ; kubed-- for package-private

;; Predicates end in -p
(defun my-thing-p (x) ...)        ; returns t or nil
(defun stringp (x) ...)           ; built-in example

;; Destructive functions end in !  (convention, not enforced)
(defun my-sort! (list) ...)

;; Constants by convention use + delimiters
(defconst +my-constant+ 42)        ; optional, not universal
```

### Docstrings

```elisp
(defun my-function (arg1 arg2 &optional arg3)
  "Short summary on first line.  End with period.

Longer description on subsequent paragraphs.  ARG1 should be a string.
ARG2 is a number.  Optional ARG3 defaults to nil.

Returns a list of results.

See also `other-function' and `another-function'."
  ...)

;; Conventions:
;; - First line: one complete sentence ending with period
;; - Arguments mentioned in UPPER CASE
;; - References to other functions in `backtick-quotes'
;; - Blank line between paragraphs
```

### When to Use What

```
Use defvar for:     global state that changes
Use defconst for:   values that shouldn't change
Use defcustom for:  user settings

Use let for:        temporary local bindings
Use let* for:       sequential local bindings

Use mapcar for:     transform every element
Use seq-filter for: keep elements matching predicate
Use dolist for:     side effects on each element
Use cl-loop for:    complex iteration

Use alist for:      small lookup tables (< ~20 items)
Use hash-table for: large lookup tables

Use condition-case for:  catching errors
Use unwind-protect for:  cleanup/finally
```

### The Lisp Style Aesthetic

```elisp
;; GOOD: closing parens at end of last line (NOT on their own line)
(defun foo (x)
  (if (> x 0)
      (progn
        (message "positive")
        (* x 2))
    0))

;; BAD: allman-style (don't do this)
(defun foo (x)
  (if (> x 0)
      (progn
        (message "positive")
        (* x 2)
      )
    0
  )
)

;; GOOD: indent consistently with 2 spaces
;; GOOD: align continuation arguments
(some-function arg1
               arg2
               arg3)

;; GOOD: blank line between top-level definitions
(defvar x 1)

(defun foo () ...)

(defun bar () ...)
```

---

## Quick Reference Card

### Python → Emacs Lisp Cheat Sheet

| Python                     | Emacs Lisp                           |
| -------------------------- | ------------------------------------ |
| `x = 5`                    | `(setq x 5)`                         |
| `x = y = 0`                | `(setq x 0 y 0)`                     |
| `def f(x): return x*2`     | `(defun f (x) (* x 2))`              |
| `lambda x: x*2`            | `(lambda (x) (* x 2))`               |
| `f(x)`                     | `(f x)`                              |
| `fn_var(x)`                | `(funcall fn-var x)`                 |
| `f(*args)`                 | `(apply #'f args)`                   |
| `if a: b else: c`          | `(if a b c)`                         |
| `a and b`                  | `(and a b)`                          |
| `a or b`                   | `(or a b)`                           |
| `not a`                    | `(not a)`                            |
| `a == b`                   | `(equal a b)`                        |
| `a is b`                   | `(eq a b)`                           |
| `print(x)`                 | `(message "%S" x)`                   |
| `[1,2,3]`                  | `'(1 2 3)`                           |
| `lst[0]`                   | `(car lst)`                          |
| `lst[1:]`                  | `(cdr lst)`                          |
| `lst[n]`                   | `(nth n lst)`                        |
| `[x]+lst`                  | `(cons x lst)`                       |
| `lst+[x]`                  | `(append lst (list x))`              |
| `len(lst)`                 | `(length lst)`                       |
| `x in lst`                 | `(member x lst)`                     |
| `[f(x) for x in lst]`      | `(mapcar f lst)`                     |
| `[x for x in lst if p(x)]` | `(seq-filter p lst)`                 |
| `{}`                       | `(make-hash-table :test 'equal)`     |
| `d[k]`                     | `(gethash k d)`                      |
| `d[k] = v`                 | `(puthash k v d)`                    |
| `try/except`               | `(condition-case ... )`              |
| `try/finally`              | `(unwind-protect ...)`               |
| `raise Exception(msg)`     | `(error msg)`                        |
| `import mod`               | `(require 'mod)`                     |
| `from mod import f`        | just `(require 'mod)` then `(f ...)` |
| `# comment`                | `; comment`                          |
| `"""docstring"""`          | first string in `defun` body         |
| `True`                     | `t`                                  |
| `False` / `None`           | `nil`                                |

---

## Where to Go From Here

1. **Read existing packages** — open any `.el` file and read it. The Emacs source itself is written in Emacs Lisp.

2. **The Emacs Lisp Reference Manual** — `C-h i m Elisp RET` — this is the authoritative reference, kept up to date with your exact version of Emacs.

3. **Write your own `init.el`** — configuring Emacs in Lisp is the best practice environment.

4. **Start small** — write a function that does something you do manually. Bind it to a key. Use it daily.

5. **Use `M-x describe-function` constantly** — every function has documentation. Read it.

6. **Study the `kubed.el` and `kubed-ext.el` files you already have** — they demonstrate nearly every concept covered here: macros (`kubed-define-resource`), advice, hooks, async processes, buffer manipulation, hash tables, and more.
