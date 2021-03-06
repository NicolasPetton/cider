## Configuration

You can certainly use CIDER without configuring it any further,
but here are some ways other folks are adjusting their CIDER
experience.

### Basic configuration

* Enable `eldoc` in Clojure buffers:

```el
(add-hook 'cider-mode-hook #'eldoc-mode)
```

* Suppress auto-enabling of `cider-mode` in `clojure-mode` buffers, when starting
  CIDER:

```el
(setq cider-auto-mode nil)
```

By default CIDER will enable `cider-mode` in all `clojure-mode` buffers when the
first CIDER connection is established. It will also add a `clojure-mode` hook to
enable it on newly created `clojure-mode` buffers. The configuration snippet
above allows you to override this (somewhat non-standard) behavior.

* By default, interactive commands that require a symbol (e.g. `cider-doc`) will
  prompt for the symbol, with the prompt defaulting to the symbol at point. You
  can set `cider-prompt-for-symbol` to nil to instead try the command with the
  symbol at point first, and only prompt if that fails (this used to be the
  default behavior in older CIDER releases).

```el
(setq cider-prompt-for-symbol nil)
```

* Don't log communication with the nREPL server:

```el
(setq nrepl-log-messages nil)
```

Basically, this will dispose of buffers like `*nrepl-messages
connection-name*`. The communication log is invaluable for debugging CIDER
issues, so you're generally advised to keep it around.

* You can hide the `*nrepl-connection*` and `*nrepl-server*` buffers
from appearing in some buffer switching commands like
`switch-to-buffer`(<kbd>C-x b</kbd>) like this:

```el
(setq nrepl-hide-special-buffers t)
```

When using `switch-to-buffer`, pressing <kbd>SPC</kbd> after the command will
make the hidden buffers visible. They'll always be visible in
`list-buffers` (<kbd>C-x C-b</kbd>).

* To prefer local resources to remote (tramp) ones when both are available:

```el
(setq cider-prefer-local-resources t)
```

* Prevent the auto-display of the REPL buffer in a separate window
  after connection is established:

```el
(setq cider-repl-pop-to-buffer-on-connect nil)
```

* Make <kbd>C-c C-z</kbd> switch to the CIDER REPL buffer in the current window:

```el
(setq cider-repl-display-in-current-window t)
```

* Prevent <kbd>C-c C-k</kbd> from prompting to save the file corresponding to
  the buffer being loaded, if it's modified:

```el
;; Don't prompt and don't save
(setq cider-prompt-save-file-on-load nil)
;; Just save without prompting
(setq cider-prompt-save-file-on-load 'always-save)
```

* Change the result prefix for interactive evaluation (by default it's `=> `):

```el
(setq cider-interactive-eval-result-prefix ";; => ")
```

To remove the prefix altogether just set it to an empty string(`""`).

* CIDER can syntax highlight symbols that are known to be defined. By default,
  this is done on symbols from the `clojure.core` namespace as well as macros
  from any namespace. If you'd like CIDER to also colorize usages of functions
  and variables from any namespace, do:

```el
(setq cider-font-lock-dynamically '(macro core function var))
```

* You can configure known endpoints used by the cider command offered via a
completing read. This is useful if you have a list of common host/ports you
want to establish remote nREPL connections to. Using an optional label is
helpful for identifying each host.

```el
(setq cider-known-endpoints '(("host-a" "10.10.10.1" "7888") ("host-b" "7888")))
```

* If you are targeting the JVM and prefer a local copy of the JDK API
  documentation over Oracle's official copy (e.g., for
  [JavaSE 8](http://docs.oracle.com/javase/8/docs/api/)), per nREPL's
  [`javadoc-info` logic (accurate as of 29 Dec 2014)](http://docs.oracle.com/javase/8/docs/api/),
  you can arrange your project to include the **root** path of the local API doc
  (i.e., where the `index.html` is located) to be included on your classpath
  (i.e., where the doc HTML files can be located by
  `clojure.java.io/resource`). For example, for Leiningen, with the local API
  path being `/usr/share/doc/java/api/`, put the following line in
  `project.clj`:

```clj
:dev {:resource-paths ["/usr/share/doc/java/api/"]}
```

**or** the following line in `$HOME/.lein/profiles.clj`:

```clj
:user {:resource-paths ["/usr/share/doc/java/api/"]}
```

More details can be found [here](https://github.com/clojure-emacs/cider/issues/930).

### Overlays

When you evaluate code in Clojure files, the result is displayed in the buffer
itself, in an overlay right after the evaluated code.  If you want this overlay
to be font-locked (syntax-highlighted) like Clojure code, set the following
variable.

```el
(setq cider-overlays-use-font-lock t)
```

You can disable overlays entirely (and display results in the echo-area at the
bottom) with the `cider-use-overlays` variable.

```el
(setq cider-use-overlays nil)
```

### Specifying indentation

It is common for macros to require special indentation mechanisms. This is most
common in macros that start with `do`, `def`, or `with-`.  CIDER has some
heuristics to detect these macros, but it also lets you explicitly specify how
a macro should be indented.

Here's a simple example of how someone would specify the indent spec for a macro
they've written (using an example in core):

```clj
(defmacro with-in-str
  "[DOCSTRING]"
  {:style/indent 1}
  [s & body]
  ...cut for brevity...)
```

And here's a more complex one:

```clj
(defmacro letfn
  "[DOCSTRING]"
  {:style/indent [1 [[:defn]] :form]}
  [fnspecs & body]
  ...cut for brevity...)
```

Don't worry if this looks intimidating. For most macros the indent spec should
be either just a number, or one of the keywords `:defn` or `:form`. A full
description of the spec is provided in [indent_spec.md](doc/indent_spec.md).

If you *don't* want to use this feature, you can disable it by setting
`cider-dynamic-indentation` to `nil` in your Emacs init file.

```el
(setq cider-dynamic-indentation nil)
```

### Minibuffer completion

Out-of-the box CIDER uses the standard `completing-read` Emacs mechanism. While
it's not fancy it certainly gets the job done (just press `TAB`). There are,
however, ways to improve upon the standard completion if you wish to.

#### icomplete

`icomplete` is bundled with Emacs and enhances the default minibuffer completion:

```el
(require 'icomplete)
```

#### ido

`ido` is also bundled with Emacs and offers more features than `icomplete`.
If you are using `ido`, be sure to use both `ido-everywhere`
and [`ido-ubiquitous`](https://github.com/DarwinAwardWinner/ido-ubiquitous).
You might also want to install [`ido-flex`](https://github.com/lewang/flx).

### Auto-completion

CIDER users are advised to use [`company-mode`](http://company-mode.github.io/)
to enable auto-completion inside of source code and REPL buffers.  To install
`company-mode` do:

`M-x package-install <RET> company <RET>`

After installation, company can be turned on  globally, like so --

```el
(global-company-mode)
```

-- or through mode-specific hooks:

```el
(add-hook 'cider-repl-mode-hook #'company-mode)
(add-hook 'cider-mode-hook #'company-mode)
```

When `company-mode` is thus enabled, it will receive completion information
from `cider-complete-at-point`, and requires no additional setup or plugins.

If you'd prefer to trigger completions manually you can add this to you config:

```el
(setq company-idle-delay nil) ; never start completions automatically
(global-set-key (kbd "M-TAB") #'company-complete) ; use M-TAB, a.k.a. C-M-i, as manual trigger
```

To make `TAB` complete, without losing the ability to manually indent, you can
add this to your config:

```el
(global-set-key (kbd "TAB") #'company-indent-or-complete-common)
```

`company-indent-or-complete-common` is available only in `company-mode` 0.9+ (at
the time of this writing it's still in development).

#### Migrating from `auto-complete-mode`

* Disable `ac-cider-setup` or `ac-nrepl-setup` from running on CIDER hooks

* Remove `cider-mode` and `cider-repl-mode` from the `ac-modes` list

#### Completion annotations

Completion candidates will be annotated by default with an abbreviation
corresponding to their type, and (contextually) their ns. The function used to
format the annotation can be configured by `cider-annotate-completion-function.`
The abbreviations used are configured by `cider-completion-annotations-alist`
and the context in which their namespace is included is configured by
`cider-completion-annotations-include-ns.`

Completion annotations can be disabled by setting
`cider-annotate-completion-candidates` to nil.

<p align="center">
  <img src="screenshots/completion-annotations.png" width="400" />
</p>

### Pretty-printing

You can configure the function used by CIDER for pretty-printing evaluation
results and other data using the `cider-pprint-fn` option.

This can be one of three values (defaults to `pprint`):

- `pprint` to use the built-in `clojure.pprint/pprint`.

- `fipp` to use the
  [Fast Idiomatic Pretty-Printer](https://github.com/brandonbloom/fipp). This is
  approximately 5-10x faster than `clojure.core/pprint`.

- `puget` to use [Puget](https://github.com/greglook/puget), which builds on
  Fipp to provide a
  [canonical serialization](https://github.com/greglook/puget#canonical-representation)
  of data, at a slight performance cost.

Alternatively, `cider-pprint-fn` can be set to the namespace-qualified name of a
Clojure function that takes a single argument and will pretty-print the value of
said argument to `*out*`.

``` el
(setq cider-pprint-fn "user/my-pprint")
```

This function must be resolvable by CIDER at the time it is called (i.e. its
containing namespace must have already been required).

CIDER will bind `*print-length*`, `*print-level*`, `*print-meta*`, and
`clojure.pprint/*print-right-margin*` when calling the pretty-printing
function - the function you provide is expected to respect these options.

### Integration with other modes

* Enabling `CamelCase` support for editing commands(like
`forward-word`, `backward-word`, etc) in the REPL is quite useful since
we often have to deal with Java class and method names. The built-in
Emacs minor mode `subword-mode` provides such functionality:

```el
(add-hook 'cider-repl-mode-hook #'subword-mode)
```

* The use of [paredit](http://mumble.net/~campbell/emacs/paredit.html)
when editing Clojure (or any other Lisp) code is highly
recommended.  You're probably using it already in your `clojure-mode`
buffers (if you're not you probably should). You might also want to
enable `paredit` in the REPL buffer as well:

```el
(add-hook 'cider-repl-mode-hook #'paredit-mode)
```

* [smartparens](https://github.com/Fuco1/smartparens) is an excellent
  alternative to paredit. Many Clojure hackers have adopted it
  recently and you might want to give it a try as well. To enable
  `smartparens` in the REPL buffer use the following code:

```el
(add-hook 'cider-repl-mode-hook #'smartparens-strict-mode)
```

* [RainbowDelimiters](https://github.com/Fanael/rainbow-delimiters) is a
  minor mode which highlights parentheses, brackets, and braces
  according to their depth. Each successive level is highlighted in a
  different color. This makes it easy to spot matching delimiters,
  orient yourself in the code, and tell which statements are at a
  given depth. Assuming you've already installed RainbowDelimiters you can
  enable it in the REPL like this:

```el
(add-hook 'cider-repl-mode-hook #'rainbow-delimiters-mode)
```

* [auto-complete](http://cx4a.org/software/auto-complete/) is a popular Emacs
  interactive auto-completion framework. [ac-cider](https://github.com/clojure-emacs/ac-cider)
  provides a completion source for auto-complete-mode, including, where CIDER provides it,
  pop-up documentation for completed symbols.

* [eval-sexp-fu](https://github.com/hchbaw/eval-sexp-fu.el) provides some visual
feedback when evaluating expressions. [cider-eval-sexp-fu](https://github.com/clojure-emacs/cider-eval-sexp-fu) provides
CIDER integration for `eval-sexp-fu`.

```el
(require 'cider-eval-sexp-fu)
```
