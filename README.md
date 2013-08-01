# venvwrapper.el

A featureful virtualenv mode for Emacs. Emulates
much of the functionality of Doug Hellmann's
[virtualenvwrapper](https://bitbucket.org/dhellmann/virtualenvwrapper/)
for Emacs.

## Features

* Works with the new
  [python.el](https://github.com/fgallina/python.el), which is the
  built in on Emacs 24.3 and up. Does not support the older python
  modes.
* Python shells, interactive shells, eshell, and any other subprocesses can
  be made aware of your virtualenvs.
* Implements a large subset of the functionality of virtualenvwrapper.

## Basic Usage

* Make sure you have [dash.el](https://github.com/magnars/dash.el) and
  [s.el](https://github.com/magnars/s.el) installed.
* Obviously make sure you have
  [virtualenv](http://www.virtualenv.org/en/latest/) installed. You
  don't actually need virtualenvwrapper.
* Copy `virtualenvwrapper.el` to somewhere on your `load-path`
* Put

  ```emacs
  (require 'virtualenvwrapper)
  (venv-initialize-interactive-shells) ;; if you want interactive shell support
  (venv-initialize-eshell) ;; if you want eshell support
  (setq venv-location "/path/to/your/virtualenvs/")
  ```

  in your config somewhere.

* Use `M-x venv-workon` to activate virtualenvs and `M-x
  venv-deactivate` deactivate them.
* If you have your virtualenvs spread around the filesystem rather
  than in one directory, just set venv-location to be a list of
  paths to each virtualenv. For example:
  ```emacs
  (setq venv-location '("/path/to/project1-env/"
                        "/path/to/ptoject2-env/"))
  ```
  Notice that the final directory of each path has a different name.
  The mode uses this fact to disambiguate virtualenvs from each other,
  so for now it is required. *WARNING* Seriously, don't try to use
  virtualenvs with the same name. The tool has no way of telling them
  apart, so horrible things may happen if you, e.g. try to delete one
  and it deletes the other.

## What do activating and deactivating actually do?

Many virtual environment support tools describe their functionality as
"it just works" or "it's so simple". This is not descriptive enough to
figure out what's wrong when something inevitably breaks, so here I
will describe *exactly* what happens when you activate a virtualenv:

1. `python-shell-virtualenv-path` is set to the virtualenv's directory
   so that when you open a new python shell, it is aware of the
   virtual environment's installed packages and modules.
2. The virtualenv's `bin` directory is prepended to the `PATH`
   environment variable so that when a process is launched from Emacs
   it is aware of any executables installed in the virtualenv (such as
   `nosetests`, `pep8`, etc.). This comes in handy because you can do
   `M-! nosetests` to run your tests, for example.
3. The `VIRTUAL_ENV` environment variable is set to the virtualenv's
   directory so that any tools that depend on this variable function
   correctly (one such tool is
   [jedi](http://tkf.github.io/emacs-jedi/))
4. The virtualenv's `bin` directory added to the `exec-path`, so that
   Emacs itself can find the environment's installed variables. This is
   useful, for example, if you want to have Emacs spawn a subprocess
   running an executable installed in a virtualenv.

When you deactivate, all these things are undone. You can safely
modify your `PATH` and `exec-path` while a virtualenv is active and
expect the changes not to be destroyed.

This covers everything except interactive shells, which are
covered in the next section.

## Shells

This thing supports two types of interactive shells, the
[eshell](https://www.gnu.org/software/emacs/manual/html_mono/eshell.html)
and the
[interactive subshell](https://www.gnu.org/software/emacs/manual/html_node/emacs/Interactive-Shell.html)
(what you get when you do `M-x shell`).

### Interactive shell

Support for interactive shell is turned on by calling
`venv-initialize-interactive-shell`. After this is done, whenever you
call `shell`, the shell will start in the correct virtualenv. This
detects whether or not you have virtualenv wrapper installed and does
the right thing in either case.  Note that changing the virtualenv in
Emacs will not affect any running shells and vice-versa, they are
independant processes.

#### WARNINGS

This feature is a pretty big hack and works by
[advising](https://www.gnu.org/software/emacs/manual/html_node/elisp/Advising-Functions.html)
the `shell` function. This works fine if you haven't otherwise tricked
out or advised it, but if this is the case it may break. Please file
an issue if you encounter any bugs with this functionality, I am
interested to see how robust it is.

### Eshell

Support for eshell is turned on by calling `venv-initialize-eshell`.
After doing this, any new eshells you launch will be in the correct
virtualenv and have access to installed executables, etc. The mode
also provides a variety of virtualenvwrapper commands that work
identically to their bash/zsh counterparts (described in detail
below). Note that in contrast to how interactive shells work, Eshell
shares an environment with Emacs, so if you activate or deactivate in
one, the other is affected as well. Note that this requires the
variable `eshell-modify-global-environment` to be set to true. Running
`venv-initialize-shell` causes this to occur. If this doesn't work for
you, open an issue! It's technically possible to separate the two, but
it requires some hacking around with the different namespaces that I'm
haven't got around to to doing yet.

## Command Reference

All of these comamnds can be called from eshell without prefixes or
parens, exactly as you would in bash or zsh. For example:

```
eshell> workon myenv
eshell> deactivate
eshell> cpvirtualenv env copy
eshell> mkvirtualenv newenv
```

#### `venv-workon`

Prompts for the name of a virtualenv and activates it as described
above. Can also be called noninteractively as `(venv-workon "name")`.

#### `venv-deactivate`

Deactivates your current virtualenv, undoing everything that `venv-workon`
did. This can also be called noninteractively as `(venv-deactivate)`.

#### `venv-mkvirtualenv`

Prompt for a name and create a new virtualenv. If your virtualenvs are
all kept in the same directory (i.e. `venv-location` is a string),
then the new virtualenv will be created in that directory. If you keep
for virtualenvs in a variety of places (i.e. `venv-location` is a
list), then the new virtualenv will be created in the current default
directory. Also callable noninteracively as `(venv-mkvirtualenv
"name")`.

#### `venv-rmvirtualenv`

Prompt for the name of a virutalenv and delete it. Also callable
noninteracively as `(venv-rmvirtualenv "name")`.

#### `venv-lsvirtualenv`

Display all virtualenvs in a help buffer. If you want to get use this
noninteractively, use `(venv-list-virtualenvs)`.

#### `venv-cdvirtualenv`

Change the current default directory to the current virtualenv's
directory. If called noninteractively, you can optionally provide an
argument, which is interpreted as a subdirectory. For example, to go
to the `bin` directory of the currently active virtualenv, call
`(venv-cdvirtualenv "bin")`.

#### `venv-cpvirtualenv`

Makes a new virtualenv that is a copy of an existing one. Prompts for
the names of both. *WARNING* This comes with same caveat as the
corresponding command in the original virtualenvwrapper, which is that
some packages hardcode their locations when being installed, so
creating new virtualenvs in this manner may cause them to break. Use
with caution


## Useful Macros

There is a `venv-with-virtualenv` macro, which takes the name of a
virtualenv and then any number of forms and executes those forms with
that virtualenv active, in that virtualenv's directory.  For example:

```emacs
(venv-with-virtualenv "myenv" (message default-directory))
```

Will message the path of `myenv`'s directory. There's also a
`venv-all-virtualenv` macro, which takes a series of forms, activates
each virtualenv env in turn, moves to its directory, and executes the
given forms.

Since its common to want to execute shell commands, there are
convenience macros, `venv-with-virtualenv-shell-command` and
`venv-allvirtualenv-shell-command`, which take a string, interpreted
as a shell command, and do exactly what you'd expect. So for example,
you can do `(venv-allvirtualenv-shell-command "pip install pep8")` to
install `pep8` in all virtualenvs. `venv-allvirtualenv-shell-command`
can also be called interactively and will prompt for a command to run
if so.

The eshell supports using this command just like in bash or zsh, so at
an eshell prompt, you can just do:

```emacs
eshell> allvirtualenv pip install pep8
```

And it will do what you expect.


## Extras

This mode doesn't screw with things you probably have customized
yourself, such as your mode line, keybindings, mode-hooks, etc. in
order to provide stuff like automatically turning on virtualenvs in
certain projects, show the virtualenv on the mode line, etc. Instead,
you can do all these things pretty easily using tools already provided
by Emacs. How to do soem of them are described below.

### Keybindings

This mode doesn't provide any. I don't presume to know how you want
your keybindings, you can bind them to whatever you want! Go crazy!

### Advising

Virtualenvwrapper lets you write shell scripts that run as hooks after you
take certain actions, such as creating or deleting a virtualenv. This mode
doesn't provide something similar, because emacs itself already does in the
form of
[advice](https://www.gnu.org/software/emacs/manual/html_node/elisp/Advising-Functions.html). For example, you could advice the mkvirtualenv function to install
commonly used tools when a new virtualenv is created:

```emacs
(defadvice venv-mkvirtualenv (after install-common-tools)
    "Install commonly used packages in new virtualenvs."
    (shell-command "pip install flake8 nose jedi"))
```

### Automatically activating a virtualenv in a particular project

Its also common to want to have a virtualenv automatically activated
when you open a file in a certain project. This mode provides no
special way to do this because once again Emacs has already done it in
the form of
[Per-Directory Local Variables](https://www.gnu.org/software/emacs/manual/html_node/emacs/Directory-Variables.html)
and
[mode hooks](https://www.gnu.org/software/emacs/manual/html_node/emacs/Hooks.html). In
order to have a virtualenv automatically activated when you open a
python file in a particular project, you could put a `.dir-locals.el` in the
project's root directory with something like:

```
((python-mode . ((project-venv-name . "myproject-env"))))
```

Now whenever python mode is activated, you will have a variable
`project-venv-name`. In order to cause this venv to be activated, we can just add
a python-mode hook:

```emacs
(add-hook 'python-mode-hook (lambda ()
                              (hack-local-variables)
                              (venv-workon project-venv-name)))
```

The call to `hack-local-variables` is necessary beacuse by default
mode-hooks are run before directory local variables are set, so we
have to do that explicitly in the hook in order to have access to
them.

### Displaying the currently active virtualenv on the mode line

The name of the currently active virtualenv is in the variable
`venv-current-name`. If you want to have it displayed on your custom
mode line you can just add `(:exec (list venv-current-name)))`
somewhere in your `mode-line-format`. If you don't customize your mode
line and just want to have the current virtualenv displayed, you can
do:

```emacs
(add-to-list 'mode-line-format '(:exec venv-current-name))
```

### Eshell prompt customization

You also might want to have the name of your current virtualenv appear
on the eshell prompt. You can do this by a pretty similar mechanism,
just include `venv-current-name` in your `eshell-prompt-function`
somewhere. Here is a simple example of a prompt that includes the
current virtualenv name followed by a dollar sign:

```emacs (setq eshell-prompt-function (lambda () (concat
venv-current-name " $ "))) ```

More about customizing the eshell prompt
[on the EmacsWiki](http://www.emacswiki.org/emacs/EshellPrompt).

### Bugs / Comments / Contributions

Open an issue or a PR!

### License

Copyright (C) 2013 James J. Porter

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
