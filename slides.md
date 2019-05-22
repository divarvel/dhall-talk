% Config As Code? Yup, but properly. Have some Dhall
% Clément Delafargue
% web2day 2019-06-05

---

## Config is solved, we have yaml

<details role="note">
if you agree with this, you won't like the rest
</details>

---

## ToDo big yaml example

---

## Config is solved, we have toml*

<details role="note">
toml solves issues with yaml,  
but these are not the ones I'm interested in
</details>

---

## Configuration is complicated and repetitive

<details role="note">
especially with infra as code and software-defined everything
</details>

---

## Config languages are simple*

<details role="note">
Configuration languages are designed to be simple. no abstraction, no indirection
(* yaml is an error)
</details>


---

## The case for config as code

<details role="note">
- great abstraction support
- familiar syntax (and semantics)
- no separate process
</details>

---

## Webpack

<details role="note">
webpack config file: module export
</details>

---

## Webpack - pros

<details role="note">
plain objects, you can use variables, ternaries, maybe functions
</details>
---

## "it's <i>just</i> JavaScript"

<details role="note">
not a new language to learn (who knows the nginx config file format?)
</details>

---

## simple abstraction

<details role="note">
you can split your config in several parts,  
use properly named variables  
maybe use functions
</details>

---

## imports

<details role="note">
you can split your config file is several files  
and use import to join them
</details>

---

## Webpack - cons

<details role="note">
</details>

---

## "it's just JavaScript"

<details role="note">
I don't want my config files to turn into angular 1.2  
`new HtmlWebpackPlugin` <- what does it do?
</details>

---

## `npm install`

<details role="note">
now the config file has deps, so you need to npm install before  
looking at what's in it
</details>

---

## <small>side effects</small>

<details role="note">
when is webpack run?  
</details>

---

## side effects

<details role="note">
does it modify other config files?  
</details>

---

## <big>side effects</big>

<details role="note">
can I cache the webpack config?  
</details>

---

## <big><big>side effects</big></big>

<details role="note">
does it depend on software that's not in package.json?
</details>

---

## <big><big><big>😱 side effects 😱</big></big></big>

<details role="note">
does reading the config drop my database?
</details>


---

## no caching

<details role="note">
</details>

---

## no automated security audit

<details role="note">
</details>

---

## just a black box

<details role="note">
that may or may not have visible effects
</details>

---

## Pretty printing a JSON does not trash your hard drive

<details role="note">
config files should just describe standalone values (eg more or less the JSON AST)
</details>

---

## Config is not code

<details role="note">
config has a different lifecycle from code, and different requirements  
boundary that's important to respect  
examples: xmonad, things that you have to recompile to configure
</details>

---

## Desirable properties

<details role="note">
not talking about syntax bikeshedding
</details>

---

## Clear syntax

<details role="note">
won't bikeshed here, but the syntax has to be unambiguous  
avoid yaml's mistakes (whitespace sensitivity, ambiguous literals)
</details>

---

## Clear semantics

<details role="note">
it should reduce to more or less an AST equivalent to JSON  
records, lists, scalars (bools, strings, numbers)
</details>

---

## Imports

<details role="note">
we should be able to split config files as we please  
in a structured, and well defined way  
(eg, we compose values, not concat files)
</details>

---

## Some abstraction

<details role="note">
at least variables, restricted functions are ok  
(eg to build records)
</details>

---

## No turing completeness

<details role="note">
a turing-complete interpreter can be turned into anything  
huge security risks (or just DoS)
</details>

---

## No side* effects

<details role="note">
Evaluating a config file should do nothing except evaluating it  
</details>

---

## Some side effects are ok, though

<details role="note">
env vars are so useful, it'd be a shame to forego them
</details>

---

## Dhall

<details role="note">
made just for this (and some more!)
</details>

---

# Clear syntax

<small>
```



{- Comments -}
{ firstKey = "string value"
, otherKey = 42
, nested = [ { otherValue = True } ]
, multiline = ''
              I can do
              as well''
}
```
</small>

<details role="note">
not whitespace significant
records are defined with `{` / `=` / `,`
records are defined with `[` / `,`
multiline strings, naturals, booleans
</details>

---

# Clear semantics

<small>
```



{- Comments -}
{ firstKey = "string value"
, otherKey = 42
, nested = [ { otherValue = True } ]
, multiline = ''
              I can do
              as well''
}
```
</small>

<details role="note">
the semantics is: reduce the file to this AST. Nothing more
</details>

---

# Imports

```
let otherFile = ./other.dhall
in otherFile



```

<details role="note">
we should be able to split config files as we please  
in a structured, and well defined way  
(eg, we compose values, not concat files)
</details>

---

# Some abstraction

<small>
```




{- Don't repeat yourself -}
let user = "bill"
in  { home       = "/home/${user}"
    , privateKey = "/home/${user}/id_ed25519"
    , publicKey  = "/home/${user}/id_ed25519.pub"
    }
```
</small>

<details role="note">
variables and functions (what else?)
</details>

---

# Some abstraction

<small>
```




let
  makeUser = \(user : Text) ->
    { home       = "/home/${user}"
    , privateKey = "/home/${user}/id_ed25519"
    , publicKey  = "/home/${user}/id_ed25519.pub"
    }
in makeUser "bill"
```
</small>

<details role="note">
simple functions
</details>

---

## No turing completeness

<details role="note">
general recursion is not allowed in the language
</details>

---

## State of the art type system

<details role="note">
all the interesting properties of dhall are thanks to the choice of
a good type system.
</details>

---

## No side* effects

<details role="note">
the semantics are: reduce this expression as much as possible.
no side effects are possible to express, this way
</details>

---

# Some side effects are ok

<small>
```



-- read a string (without the quotes)
{ currentHome = env:HOME as Text

-- read any dhall expression
, currentPort = env:PORT }
```
</small>

<details role="note">
Reading env vars is fine
</details>

---

# Some side effects are ok

<small>
```




let map = https://prelude.dhall-lang.org/List/map

in map Natural Bool Natural/even [ 2, 3, 5 ]
```
</small>

<details role="note">
http imports. super easy to reuse code  
stdlib distributed this way
</details>

---

# Some side effects are ok

<small>
```





let map =
  https://prelude.dhall-lang.org/List/map
    sha256:c60cc328c8aab8253e7372ae973ab7fd7b37448dc5bd9602f8e17b52a950d57b

in map Natural Bool Natural/even [ 2, 3, 5 ]
```
</small>

<details role="note">
you can freeze imports to avoid changes *and* use cache
socket(AF_INET6, SOCK_DGRAM|SOCK_CLOEXEC|SOCK_NONBLOCK, IPPROTO_IP) = 0
open("/home/clementd/.cache/dhall/c60cc328c8aab8253e7372ae973ab7fd7b37448dc5bd9602f8e17b52a950d57b", O_RDONLY|O_NOCTTY|O_NONBLOCK) = 0
</details>

---

# Clear semantics

```
cat evenList.dhall \
  | dhall resolve \
  | dhall normalize


```

<details role="note">
first we resolve all imports (http + env)  
then we evaluate (β-reduce)
</details>

---

# Clear semantics

<small>
```





let map =
  https://prelude.dhall-lang.org/List/map
    sha256:c60cc328c8aab8253e7372ae973ab7fd7b37448dc5bd9602f8e17b52a950d57b

in map Natural Bool Natural/even [ 2, 3, 5 ]
```
</small>

<details role="note">
to go from this
</details>

---

# Clear semantics

```
[ True, False, False ]




```

<details role="note">
to this  
for instance: commit the thing above, but the ops team can read  
the generated config, or automated tools / linters can act on  
the generated output.
</details>

---

## How to use it

<details role="note">
if nobody reads the config file, it's not very useful
</details>

---

# from Haskell

<small>
```haskell




data Config = { port :: Natural }

main :: IO ()
main = do
  Config{port} <- input auto "./config.dhall"
  runServer port
```
</small>

<details role="note">
config is type checked by dhall, then translated to haskell
</details>

---

# Language bindings

<ul style="font-size: 1em;">
<li>haskell</li>
<li>nix</li>
<li>ruby</li>
<li>java (via eta)</li>
<li>go (wrapped executable)</li>
</ul>


<details role="note">
complete implementation  
except go, it uses JSON as an intermediate format, so while you  
can use the whole language, you cannot get all the expressible  
typse in go (but normalization is complete)
</details>

---

# Language bindings (in progress)

<ul style="font-size: 1em;">
<li>rust</li>
<li>clojure</li>
<li>purescript</li>
<li>python</li>
</ul>

<details role="note">
work in progress As soon as the rust impl is done, it may be easier  
to have bindings in more languages, like JavaScript
</details>

---

![](./assets/im-not-dead-yet.jpg)

<details role="note">
this is all good and nice, but either your lang doesn't have bindings  
or you don't have power over the config format (eg kuberetes)
</details>

---

# Normalize to json/yaml

```
cat config.dhall \
  | dhall-to-yaml \
  > config.yaml


```

<details role="note">
you can normalize almost everything into json and yaml (except functions):  
primitive types, records, lists, unions
</details>

---

## Types for common software

<details role="note">
model types and helpers for kubernetes, ansible, all that. This allows quick  
and inexpensive checks for big config files before actually running software
</details>

---

## [dhall-kubernetes](https://github.com/dhall-lang/dhall-kubernetes)

<details role="note">
types, default values  
then you get the ability to use imports and functions to modularize your config
</details>

---

## `dhall-to-text` for custom formats

<details role="note">
you can generate any config format (but you have to write the projection function)
</details>

---

## Closing words

<details role="note">
dhall is worth it just for the imports + env vars features  
it's a great equivalent to hocon for instance  
Types and functions can come in handy later
</details>

---

## Don't go overboard

<details role="note">
Even with its limitations, dhall is impressively expressive. You can implement a lot with it  
don't forget this is configuration
</details>

---

![](./assets/kiss.jpg)

<details role="note">
it's still config, some redundancy is desirable (more than in regular programming)  
even though people _can_ normalize the conf and read it, it should stay easy to read and edit  
having all this power is cool, as long as you don't abuse it
</details>

---

## Let's keep it ops-friendly

<details role="note">
try to put you in your ops team shoes  
work with them to find the abstraction level everyone is comfortable in
</details>

