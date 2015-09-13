# The Emacs Lisp Style Guide

> Role models are important. <br/>
> -- Officer Alex J. Murphy / RoboCop

This Emacs Lisp style guide recommends best practices so that real-world Emacs Lisp
programmers can write code that can be maintained by other real-world Emacs Lisp
programmers. A style guide that reflects real-world usage gets used, and a
style guide that holds to an ideal that has been rejected by the people it is
supposed to help risks not getting used at all &mdash; no matter how good it is.

The guide is separated into several sections of related rules. I've
tried to add the rationale behind the rules (if it's omitted, I've
assumed that it's pretty obvious).

I didn't come up with all the rules out of nowhere; they are mostly
based on my extensive career as a professional software engineer,
feedback and suggestions from members of the Emacs Lisp community, and
various highly regarded Emacs Lisp programming resources, such as
["GNU Emacs Lisp Reference Manual"](https://www.gnu.org/software/emacs/manual/elisp.html).

The guide is still a work in progress; some sections are missing,
others are incomplete, some rules are lacking examples, some rules
don't have examples that illustrate them clearly enough. In due time
these issues will be addressed &mdash; just keep them in mind for now.

Please note, that the Emacs developers maintain a list of
[coding conventions and tips](http://www.gnu.org/software/emacs/manual/html_node/elisp/Tips.html#Tips)
too.

You can generate a PDF or an HTML copy of this guide using
[Transmuter](https://github.com/TechnoGate/transmuter).

## Table of Contents

* [Source Code Layout & Organization](#source-code-layout--organization)
* [Syntax](#syntax)
* [Naming](#naming)
* [Strings](#strings)
* [Macros](#macros)
    * [Macro Declarations](#macro-declarations)
* [Anonymous (lambda) functions](#anonymous-lambda-functions)
* [Loading and Autoloading](#loading-and-autoloading)
* [Comments](#comments)
    * [Comment Annotations](#comment-annotations)
    * [Docstrings](#docstrings)
* [Existential](#existential)

## Source Code Layout & Organization

> Nearly everybody is convinced that every style but their own is
> ugly and unreadable. Leave out the "but their own" and they're
> probably right... <br/>
> -- Jerry Coffin (on indentation)

The indentation guidelines specified here can be taken for granted if
you're writing in Emacs. It will always do the right thing when you
hit `<tab>`.

* Use **spaces** for indentation. No hard tabs.

* For regular functions, vertically align function arguments.

    ```el
    ;; good
    (format "%s %d"
            something
            something-else)

    ;; bad
    (format "%s %d"
      something
      something-else)
    ```

* If the first argument is on a new line, align it with the function's name.

    ```el
    ;; good
    (format 
     "%s %d"
     something
     something-else)

    ;; bad
    (format 
      "%s %d"
      something
      something-else)
    ```

* Some forms are special, they take 1 or more _"special"_ arguments
  followed by a _"body"_ (an arbitrary number of arguments where only
  the final return value matters), e.g. `if`, `let`,
  `with-current-buffer`, etc. The special arguments should either be
  on the same line as the form's name or be indented by 4 spaces. The
  body arguments should be indented by 2 spaces.

    ```el
    ;; good
    (when something
      (something-else))

    ;; bad - four spaces on the body
    (when something
        (something-else))
        
    ;; bad - aligned like a regular function
    (when 
     something
     (something-else))
    ```

* Note the "if" clause of an `if` form is a special argument, indent it
  by 4 spaces.

    ```el
    ;; good
    (if something
        if-clause
      else-clause)

    ;; bad
    (if something
      if-clause
      else-clause)
    ```

* Vertically align `let` bindings.

    ```el
    ;; good
    (let ((thing1 "some stuff")
          (thing2 "other stuff")
      ...)

    ;; bad
    (let ((thing1 "some stuff")
      (thing2 "other stuff"))
      ...)
    ```

* Use Unix-style line endings. (*BSD/Solaris/Linux/OSX users are
  covered by default, Windows users have to be extra careful.)
    * If you're using Git you might want to add the following
    configuration setting to protect your project from Windows line
    endings creeping in:

    ```
    bash$ git config --global core.autocrlf true
    ```

* If any text precedes an opening bracket(`(`, `{` and
`[`) or follows a closing bracket(`)`, `}` and `]`), separate that
text from that bracket with a space. Conversely, leave no space after
an opening bracket and before following text, or after preceding text
and before a closing bracket.

    ```el
    ;; good
    (foo (bar baz) quux)

    ;; bad
    (foo(bar baz)quux)
    (foo ( bar baz ) quux)
    ```

* Place all trailing parentheses on a single line instead of distinct lines.

    ```el
    ;; good; single line
    (when something
      (something-else))

    ;; bad; distinct lines
    (when something
      (something-else)
    )
    ```

* Use empty lines between top-level forms.

    ```el
    ;; good
    (defvar x ...)

    (defun foo ...)

    ;; bad
    (defvar x ...)
    (defun foo ...)
    ```

    An exception to the rule is the grouping of related `def`s together.

    ```el
    ;; good
    (defconst min-rows 10)
    (defconst max-rows 20)
    (defconst min-cols 15)
    (defconst max-cols 30)
    ```

* Do not place blank lines in the middle of a function or
macro definition.  An exception can be made to indicate grouping of
pairwise constructs as found in e.g. `let` and `cond`.

* Where feasible, avoid making lines longer than 80 characters.

* Avoid trailing whitespace.

* Avoid parameter lists with more than three or four positional parameters.

* Always enable lexical scoping. This must be done on the first line
  as a file local variable.

    ```el
    ;;; -*- lexical-binding: t; -*-
    ```

## Syntax

* Don't wrap the else clause of an `if` in a `progn` (it's wrapped in `progn` implicitly).

    ```el
    ;; good
    (if something
        if-clause
      (something)
      (something-else))

    ;; bad
    (if something
        if-clause
      (progn
        (something)
        (something-else)))
    ```

* Use `when` instead of `(if ... (progn ...)`.

    ```el
    ;; good
    (when pred
      (foo)
      (bar))

    ;; bad
    (if pred
      (progn
        (foo)
        (bar)))
    ```

* Use `unless` instead of `(when (not ...) ...)`.

    ```el
    ;; good
    (unless pred
      (foo)
      (bar))

    ;; bad
    (when (not pred)
      (foo)
      (bar))
    ```

* When doing comparisons, keep in mind that the functions `<`,
  `>`, etc. accept a variable number of arguments as of Emacs 24.4.

    ```el
    ;; Preferred
    (< 5 x 10)

    ;; Old
    (and (> x 5) (< x 10))
    ```

* Use `t` as the catch-all test expression in `cond`.

    ```el
    ;; good
    (cond
      ((< n 0) "negative")
      ((> n 0) "positive")
      (t "zero"))

    ;; bad
    (cond
      ((< n 0) "negative")
      ((> n 0) "positive")
      (:else "zero"))
    ```

* Use `(1+ x)` & `(1- x)` instead of `(+ x 1)` and `(- x 1)`.

## Naming

> The only real difficulties in programming are cache invalidation and
> naming things. <br/>
> -- Phil Karlton

* Use `lisp-case` for function and variable names.

    ```el
    ;; good
    (defvar some-var ...)
    (defun some-fun ...)

    ;; bad
    (defvar someVar ...)
    (defun somefun ...)
    (defvar some_fun ...)
    ```

* Prefix top-level names with the name of the library they belong to in order to avoid
name clashes.

    ```el
    ;; good
    (defun projectile-project-root ...)

    ;; bad
    (defun project-root ...)
    ```

* Prefix unused local (lexically scoped) variables with `_`.

    ```el
    ;; good
    (lambda (x _y) x)

    ;; bad
    (lambda (x y) x)
    ```

* Use `--` to denote private top-level definitions (e.g. `projectile--private-fun`).

* The names of predicate methods (methods that return a boolean value)
  should end in a `p` if it's a single-word name and a `-p` if it's a
  multi-word name (e.g., `evenp` and `buffer-live-p`).

    ```el
    ;; good
    (defun palindromep ...)
    (defun only-one-p ...)

    ;; bad
    (defun palindrome? ...) ; Scheme style
    (defun is-palindrome ...) ; Java style
    ```

## Macros

* Don't write a macro if a function will do.

* Create an example of a macro usage first and the macro afterwards.

* Break complicated macros into smaller functions whenever possible.

* A macro should usually just provide syntactic sugar and the core of
  the macro should be a plain function. Doing so will improve
  composability.

* Prefer syntax-quoted forms over building lists manually.

## Functions

* Use `lambda`s for local bindings and function calls, **not** for
  hooks or global variables. Define named functions for the latter,
  they aid readability and customizability.

    ```el
    ;;; Good
    (mapcar (lambda (x) (or (car x) "")) some-list)
    (let ((predicate (lambda (x) (and (numberp x) (evenp x)))))
      (funcall predicate 1000))

    ;;; Bad - Define real functions for these.
    (defcustom my-predicate (lambda (x) (and (numberp x) (evenp x)))
      ...)
    (define-key my-keymap (kbd "C-f")
      (lambda () (interactive) (forward-char 1)))
    (add-hook 'my-hook (lambda () (save-some-buffers)))
    ```

* **Never** hard quote a lambda, it impedes byte-compilation.

    ```el
    ;;; Good
    (lambda (x) (car x))

    ;;; Ok, but redundant.
    #'(lambda (x) (car x))

    ;;; Bad
    '(lambda (x) (car x))
    ```

* Don't wrap functions in anonymous functions when you don't need to.

    ```el
    ;; good
    (cl-remove-if-not #'evenp numbers)

    ;; bad
    (cl-remove-if-not (lambda (x) (evenp x)) numbers)
    ```

* Use a sharp quote (`#'`) when quoting function names. It's a good
  hint for the byte-compiler, which will warn you if the function is
  undefined. Some macros can also behave differently otherwise (like
  `cl-labels`).

    ```el
    ;; good
    (cl-remove-if-not #'evenp numbers)
    (global-set-key (kbd "C-l C-l") #'redraw-display)
    (cl-labels ((butterfly () (message "42")))
      (funcall #'butterfly))
    
    ;; bad
    (cl-remove-if-not 'evenp numbers)
    (global-set-key (kbd "C-l C-l") 'redraw-display)
    (cl-labels ((butterfly () (message "42")))
      (funcall 'butterfly))
    ```

### Macro Declarations

* Always declare the [debug-specification](http://www.gnu.org/software/emacs/manual/html_node/elisp/Specification-List.html#Specification-List),
  this tells edebug which arguments are meant for evaluation. If all
  arguments are evaluated, a simple `(declare (debug t))` is enough.

* Declare the [indent specification](https://www.gnu.org/software/emacs/manual/html_node/elisp/Indenting-Macros.html#Indenting-Macros)
  if the macro arguments should not be aligned like a function (think
  of `defun` or `with-current-buffer`).

    ```el
    (defmacro define-widget (name &rest forms)
      "Description"
      (declare (debug (sexp body))
               (indent defun))
      ...)
    ```

### Loading and Autoloading

* Always end each library file with a `provide` statement and an
  appropriate comment (the `provide` statement will allow dependent
  libraries to use `require`).

    ```el
    (provide 'foo)

    ;;; foo.el ends here
    ```

* Always load library dependencies with `require`, rather than `load`
  or `load-library` (the former is idempotent, while the others can
  result in multiple evaluations).

* Include `autoload` cookies for mode definitions and commonly-used
  user-facing functions and commands (i.e. setup functions and
  commands that could be bound to a key). Conversely, **do not**
  provide autoload cookies for global variables or internal functions.

    ```el
    ;;; good
    ;;;###autoload
    (define-derived-mode foo-mode ...)

    ;;;###autoload
    (define-minor-mode foo-minor-mode ...)

    ;;;###autoload
    (defun foo-setup () ...)

    ;;; bad
    ;;;###autoload
    (defun foo--internal () ...)

    ;;;###autoload
    (defvar foo-option)
    ```

* **Do not** provide `autoload` cookies for non-definition top-level
  forms (autoloading a library should never alter the behavior of a
  user's configuration). The single exception: `auto-mode-alist` can
  be altered for new major modes.

    ```el
    ;;; good
    ;;;###autoload
    (add-to-list 'auto-mode-alist '("\\.foo\\'" . foo-mode))

    ;;; bad
    ;;;###autoload
    (foo-setup)
    ```

## Comments

> Good code is its own best documentation. As you're about to add a
> comment, ask yourself, "How can I improve the code so that this
> comment isn't needed?" Improve the code and then document it to make
> it even clearer. <br/>
> -- Steve McConnell

* Endeavor to make your code as self-documenting as possible.

* Write heading comments with at least three semicolons.

* Write top-level comments with three semicolons if it represents a
  heading, otherwise use two semicolons.

* Write comments on a particular fragment of code before that fragment
  and aligned with it, using two semicolons.

* Write margin comments with one semicolon.

* Always have at least one space between the semicolon
and the text that follows it.

    ```el
    ;;; Frob Grovel
    ;; This is where Frob grovels and where Grovel frobs.

    ;; This section of code has some important implications:
    ;;   1. Foo.
    ;;   2. Bar.
    ;;   3. Baz.

    (defun fnord (zarquon)
      ;; If zob, then veeblefitz.
      (quux zot
            mumble             ; Zibblefrotz.
            frotz))
    ```

* Comments longer than a word begin with a capital letter and use
  punctuation. Separate sentences with two spaces.

* Avoid superfluous comments.

    ```el
    ;; bad
    (1+ counter) ; increments counter by one
    ```

* Keep existing comments up-to-date. An outdated comment is worse than no comment
at all.

> Good code is like a good joke - it needs no explanation. <br/>
> -- Russ Olsen

* Avoid writing comments to explain bad code. Refactor the code to
  make it self-explanatory. ("Do, or do not. There is no try." --Yoda)

### Comment Annotations

* Annotations should usually be written on the line immediately above
  the relevant code.

* The annotation keyword is followed by a colon and a space, then a note
  describing the problem.

* If multiple lines are required to describe the problem, subsequent
  lines should be indented as much as the first one.

* Tag the annotation with your initials and a date so its relevance can
  be easily verified.

    ```el
    (defun some-fun ()
      ;; FIXME: This has crashed occasionally since v1.2.3. It may
      ;;        be related to the BarBazUtil upgrade. (xz 13-1-31)
      (baz))
    ```

* In cases where the problem is so obvious that any documentation would
  be redundant, annotations may be left at the end of the offending line
  with no note. This usage should be the exception and not the rule.

    ```el
    (defun bar ()
      (sleep 100)) ; OPTIMIZE
    ```

* Use `TODO` to note missing features or functionality that should be
  added at a later date.

* Use `FIXME` to note broken code that needs to be fixed.

* Use `OPTIMIZE` to note slow or inefficient code that may cause
  performance problems.

* Use `HACK` to note "code smells" where questionable coding practices
  were used and should be refactored away.

* Use `REVIEW` to note anything that should be looked at to confirm it
  is working as intended. For example: `REVIEW: Are we sure this is how the
  client does X currently?`

* Use other custom annotation keywords if it feels appropriate, but be
  sure to document them in your project's `README` or similar.

### Docstrings

Emacs is famous for the breadth, depth, and ubiquity of its
documentation. By taking the time to write docstrings in your package,
you are helping to continue that tradition!

* Begin with a terse, complete sentence. Use imperative language. For
  example, prefer "Verify" over "Verifies", and "Check" over "Checks".

* When a function takes arguments, mention what the arguments do,
  whether they are required, and so on. Describe the arguments in
  UPCASE, and order them as they are used.

* Always capitalize "Emacs".

* Do not indent subsequent lines of a documentation string. This looks nice in the source code, but looks bizarre when users view the documentation.

  ```el
  ;; good
  (defun goto-line (line &optional buffer)
    "Go to LINE, counting from line 1 at beginning of buffer.
  If called interactively, a numeric prefix argument specifies
  LINE; without a numeric prefix argument, read LINE from the
  minibuffer..."
  ...)

  ;; bad
  (defun goto-line (line &optional buffer)
    "Go to LINE, counting from line 1 at beginning of buffer.
     If called interactively, a numeric prefix argument specifies
     LINE; without a numeric prefix argument, read LINE from the
     minibuffer..."
    ...)

  ;; also bad
  (defun goto-line (line &optional buffer)
    "Go to LINE, counting from line 1 at beginning of buffer.
     If called interactively, a numeric prefix argument specifies
   LINE; without a numeric prefix argument, read LINE from the
   minibuffer..."
    ...)
  ```

* Emacs' built-in utility, Checkdoc, can automatically check
  docstrings for adherence to coding conventions with the built-in
  Checkdoc utility. Many in the Emacs community use Checkdoc by way of
  [Flycheck](http://flycheck.readthedocs.org/en/latest/).

## Existential

* Be consistent. In an ideal world, be consistent with these guidelines.
* Use common sense.

# Contributing

Nothing written in this guide is set in stone. It's my desire to work
together with everyone interested in Emacs Lisp coding style, so that we could
ultimately create a resource that will be beneficial to the entire Emacs
community.

Feel free to open tickets or send pull requests with improvements. Thanks in
advance for your help!

You can also support the style guide with financial
contributions via [gittip](https://www.gittip.com/bbatsov).

[![Support via Gittip](https://rawgithub.com/twolfson/gittip-badge/0.2.0/dist/gittip.png)](https://www.gittip.com/bbatsov)

# License

![Creative Commons License](http://i.creativecommons.org/l/by/3.0/88x31.png)
This work is licensed under a
[Creative Commons Attribution 3.0 Unported License](http://creativecommons.org/licenses/by/3.0/deed.en_US)

# Spread the Word

A community-driven style guide is of little use to a community that
doesn't know about its existence. Tweet about the guide, share it with
your friends and colleagues. Every comment, suggestion or opinion we
get makes the guide just a little bit better. And we want to have the
best possible guide, don't we?

Cheers,<br/>
[Bozhidar](https://twitter.com/bbatsov)
