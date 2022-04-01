# Hibiscus.nvim
> :hibiscus: Highly opinionated macros to elegantly write your neovim config.

Companion library for [tangerine](https://github.com/udayvir-singh/tangerine.nvim),
but it can also be used standalone.

## Rational
- :candy: Syntactic eye candy over hellscape of lua api
- :tanabata_tree: Provides missing features in both fennel and nvim api

# Installation
1. Create file `plugin/tangerine.lua` to bootstrap hibiscus:
```lua
-- ~/.config/nvim/plugin/tangerine.lua

-- pick your plugin manager, default [standalone]
local pack = "tangerine" or "packer" or "paq"

local function bootstrap (name, url, path)
	if vim.fn.empty(vim.fn.glob(path)) > 0 then
		print(name .. ": installing in data dir...")

		vim.fn.system {"git", "clone", url, path}

		vim.cmd [[redraw]]
		print(name .. ": finished installing")
	end
end

bootstrap (
	"hibiscus.nvim",
	"https://github.com/udayvir-singh/hibiscus.nvim",
	vim.fn.stdpath [[data]] .. "/site/pack/" .. pack .. "/start/hibiscus.nvim"
)
```

2. Call the `setup()` function:
```lua
-- NOTE: require before calling tangerine or your compiler

require [[hibiscus]].setup()
```

3. Require a macro library at top of your modules:
```fennel
;; require all macros
(require-macros :hibiscus.core)
(require-macros :hibiscus.vim)

;; require selected macros
(import-macros {:fstring f} :hibiscus.core)
(import-macros {: map!}     :hibiscus.vim)
```

 DONE: now start using these macros in your config

---

#### Packer
You can use packer to manage hibiscus afterwards:

```fennel
(local packer (require :packer))

(packer.startup (fn []
	(use :udayvir-singh/hibiscus.nvim)))
```

#### Paq
```fennel
(local paq (require :paq))

(paq {
	:udayvir-singh/hibiscus.nvim
})
```

# Neovim Macros
Add these lines of top of your modules:
```fennel
(require-macros :hibiscus.vim)
;; or
(import-macros {:augroup! aug!} :hibiscus.vim)
```

## keymaps
#### map!
<pre lang="fennel"><code>(map! {args} {lhs} {rhs})
</pre></code>

Defines vim keymap for the given modes from {lhs} to {rhs}

##### Arguments:
{args} can contain the following values:
```fennel
; modes |                   options                           |
[ nivcx  :buffer :remap :silent :nowait :expr :unique :script ]
```

##### Examples:
- For Vimscript:
```fennel
(map! [n :buffer] :R "echo &rtp")

(let [rhs ":echo hello"]
  (map! [nv] :lhs rhs))
```

- For Fennel Functions:
```fennel
(fn greet []
  (print "Hello World!"))

(map! [nx] :lhs 'greet) ; variables need to be quoted to indicate they are function

(map! [nv] :lhs #(print "inline functions don't require quoting"))
```

## autocmds
#### augroup!
<pre lang="fennel"><code>(augroup! {name} {cmds})
</pre></code>

Defines autocmd group of {name} with {cmds} containing [groups pattern cmd] chunks.

##### Examples:
- For Vimscript:
```fennel
(local clj "clojure")

(augroup! :greet
  [[FileType]           clj   "echo hello"]
  [[BufRead BufNewFile] *.fnl "echo hello"])
```

- For Fennel Functions:
```fennel
(fn hello [] (print :hello))

(augroup! :greet
  [[BufRead] * 'hello] ; remember to quote functions
  [[BufRead] * #(print "HOLLA!")])
```

## commands
#### command!
<pre lang="fennel"><code>(command! {args} {lhs} {rhs})
</pre></code>

Defines user command {lhs} to {rhs}

##### Arguments:
{args} can contain the same opts as `:command`:
```fennel
[
  :bar      true
  :bang     true
  :buffer   true
  :register true
  :range    (or true <string>)
  :addr     <string>
  :count    <string>
  :nargs    <string>
  :complete <string>
]
```

##### RHS Parameters:
`:command` parameters like `<bang>` are translated by hibiscus into following table:
```fennel
{
  :bang  <boolean>
  :qargs <string>
  :count <number>
  :lines [<number> <number>]
}
```
They are passed as first argument to lua function, For example:
```fennel
(fn example [opts]
  (print opts.qargs))

(command! [:nargs "*"] :Lhs 'example)
```

##### Examples:
- For Vimscript:
```fennel
(command! [:nargs "1"] :Lhs "echo 'hello ' . <q-args>")
```

- For Fennel Functions:
```fennel
(fn greet [opts]
  (print :hello opts.qargs))

(command! [:nargs "1"] :Lhs 'greet) ; again remember to quote 

(command! [:bang true] :Lhs '(print opts.bang))
; or
(command! [:bang true] :Lhs (fn [opts] (print opts.bang)))
```

## vim options
#### set!
Works like command `:set`, sets vim option {name} to {val}
```fennel
(set! nobackup)
(set! tabstop 4)

(each [_ opt (ipairs ["number" "rnu"])]
      (set! opt true))
```

#### set+
Appends {val} to string-style option {name}

```fennel
(set+ wildignore "*.foo")
```

#### set^
Prepends {val} to string-style option {name}
```fennel
(set^ wildignore ["*.foo" "*.baz"])
```

#### rem!
Removes {val} from string-style option {name}

```fennel
(rem! wildignore "*.baz")
```

#### color!
Sets vim colorscheme to {name}

```fennel
(color! "desert")
```

## variables
#### g!
Sets global variable {name} to {val}.

```fennel
(g! mapleader " ")
```

#### b!
Sets buffer scoped variable {name} to {val}.

```fennel
(b! gretting "Hello World!")
```

