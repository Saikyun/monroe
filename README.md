_It takes a Miracle to reach Arcadia_

# Miracle - an Arcadia nREPL client
Miracle is a fork of [Monroe](https://github.com/sanel/monroe), which in turn is a nREPL client for Emacs. Miracle is meant to be used with ClojureCLR in general, and Arcadia in particular.

Since [Arcadia](https://github.com/arcadia-unity/Arcadia) runs on CLR there are some things that needed to be changed in Monroe in order for e.g. jump to definiton to work.

If you are not familiar with nREPL, it is protocol for accessing
Clojure [REPL](http://en.wikipedia.org/wiki/Read-eval-print_loop) over
the network.

The name comes from Marilyn Monroe's sister, who's last name is Miracle.

Below follows a description of Monroe, which currently applies to Miracle as well.

# About Monroe

In short, Monroe aims to have:

* easy access to Clojure REPL via nREPL protocol
* simple installation without any dependencies, except Emacs
* REPL with colors and history support
* generic approach - you can use it with other languages than Clojure
* only REPL for interaction with shortcuts - no funny windows or buffers with errors,
  messages and other distractions

On other hand, Monroe is not:

* Clojure IDE like [Cider](https://github.com/clojure-emacs/cider)
* Kitchen sink that will do Clojure work for you

## Installation

Install [Arcadia](https://github.com/arcadia-unity/Arcadia). Might work with other ClojureCLR nREPLs, but Arcadia is Miracle's primary focus.

Make sure you have `clojure-mode.el` installed first. You can get it
from Marmalade repository, melpa or directly from
[here](https://github.com/clojure-emacs/clojure-mode).

Clone this repository into .emacs.d, it's currently not on melpa.
```sh
cd ~/.emacs.d
git clone https://github.com/Saikyun/miracle.git
```

In your [Emacs init file](https://www.gnu.org/software/emacs/manual/html_node/emacs/Init-File.html), put:

```el
(add-to-list 'load-path "~/.emacs.d/miracle")
(require 'miracle)
(add-hook 'clojure-mode-hook 'clojure-enable-miracle)
```

Then either evaluate the rows or restart Emacs.

Start an Arcadia project in Unity. In order for autocomplete to work, put this repo in your `Assets`-folder: https://github.com/sogaiu/clojure-complete
You need to be on the `clr-support` branch.

```
cd Assets
git clone https://github.com/sogaiu/clojure-complete
git branch clr-support
git pull
```

Then, in Emacs:

<kbd>M-x miracle [RET]</kbd>

and follow the question about nREPL server location and port. The defaults are the same as Arcadia's defaults.

## Keys and shortcuts

### Miracle shortcuts for code buffer

These shortcuts are valid from code buffer where you edit Clojure
code and where *miracle-interaction-mode* is activated.

Keys                | Description
--------------------|----------------------------
<kbd>C-c C-c</kbd>  | Evaluate expression at point.
<kbd>C-c C-r</kbd>  | Evaluate region.
<kbd>C-c C-k</kbd>  | Evaluate current buffer contents.
<kbd>C-c C-l</kbd>  | Load current file from disk.
<kbd>C-c C-d</kbd>  | Describe symbol at point, showing documentation in REPL window.
<kbd>C-c C-n</kbd>  | Evaluate namespace.
<kbd>C-c C-b</kbd>  | Interrupt running job.
<kbd>M-.</kbd>      | Jump to definition of var at point.
<kbd>M-,</kbd>      | Jump back to where you were before you did `M-.`

Note the difference between <kbd>C-c C-k</kbd> and <kbd>C-c C-l</kbd>;
the former loads the contents of the buffer and sends them directly
over the wire; this can differ from the state of the namespace on
disk, and doesn't always convey line number information. It loads each
top-level form in the file individually, and if one of them fails it
will continue compiling the rest. The second one tells the server to
load the whole file from its disk, so if you are connected to a remote
server and have made changes to your local copy, they will not be
loaded. However, a single exception will halt the whole thing.

### Miracle shortcuts for REPL buffer

These shortcuts are valid in REPL buffer; also, most of the standard
*comint-mode* shortcuts should work without problems.

Keys                | Description
--------------------|----------------------------
<kbd>C-c C-d</kbd>  | Describe symbol at point, showing documentation in REPL window.
<kbd>C-c C-c</kbd>  | Interrupt running job.
<kbd>M-.</kbd>      | Jump to definition of var at point.
<kbd>C-c C-f</kbd>  | Replaces the last result with a pretty printed version of it. `f` is for formatting.

###  For autocompletion, use the following (thanks sogaiu)
#### Prerequisites

- company-mode -- https://company-mode.github.io/
- clojure-complete (clr branch) -- https://github.com/sogaiu/clojure-complete/tree/clr-support

Put the following in your `init.el`.

```
(with-eval-after-load "miracle"
  (defun miracle-eval-string (s callback)
    (miracle-send-eval-string
     s
     (lambda (response)
             (miracle-dbind-response response (id value status)
                                     (when (member "done" status)
                                       (remhash id miracle-requests))
                                     (when value
                                       (funcall callback nil value))))))

  (defun miracle-get-completions (word callback)
    (interactive)
    (miracle-eval-string
     (format "(do (require '[%s]) (%s/completions \"%s\"))"
             "complete.core" "complete.core" word)
     (lambda (err s)
             (progn
              ;; XXX
              (message (format "received str: %s" s))
              (message (format "err: %s" err))
              (when (not err)
                (funcall callback (read-from-whole-string s)))))))

  (defun company-miracle (command &optional arg &rest ignored)
    (interactive (list 'interactive))
    (cl-case command
             (interactive (company-begin-backend 'company-miracle))
             (prefix (and (or ;;(eq major-mode 'clojurec-mode)
                           ;;(eq major-mode 'clojure-mode)
                           (eq major-mode 'miracle-mode))
                          (get-buffer "*miracle-connection*")
                          (substring-no-properties (company-grab-symbol))))
             (candidates (lexical-let ((arg (substring-no-properties arg)))
                                      (cons :async (lambda (callback)
                                                           (miracle-get-completions arg callback)))))))

  ;; XXX: problems w/o the following when invoking company-grab-symbol
  (setq cider-mode nil)

  (add-to-list 'company-backends 'company-miracle)
```


## Bug reports & patches

Feel free to report any issues you find or you have suggestions for improvements.
