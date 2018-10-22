---
title: "Dear Haskell it's not you, it's your tooling."
date: 2018-05-22T18:57:56-04:00
---

##### Dear Haskell; yes you have some bad parts. `String` ahem. But for the most part I enjoy learning to write you. See, the problem is your friends; they are extremely agitating.

#### First, your recommended tool Stack
Stack does good work sometimes, but seems to prefer making very unreasonable choices:

##### Config hell
The default project template has three config files: `config-hell.cabal` `package.yaml`, `stack.yaml`.
So if you want to add a dependency, how pray tell would you do it?
Let's look at the config files and see if there's a clear way.

##### stack.yaml
Well the tool we used was stack so let's start in `stack.yaml`.
In `stack.yaml` the obvious choices are `packages` and `extra-deps` array.

The comments above packages read:
```
# User packages to be built.
# Various formats can be used as shown in the example below.
#
# packages:
# - some-directory
# - https://example.com/foo/bar/baz-0.0.2.tar.gz
# - location:
#    git: https://github.com/commercialhaskell/stack.git
#    commit: e7b331f14bcffb8367cd58fbfc8b40ec7642100a
# - location: https://github.com/commercialhaskell/stack/commit/e7b331f14bcffb8367cd58fbfc8b40ec7642100a
#  subdirs:
#  - auto-update
#  - wai
```

So `packages` are local directories and other addressable content. Not what we're looking for.

What about `extra-deps`:

```
# Dependency packages to be pulled from upstream that are not in the resolver
# using the same syntax as the packages field.
# (e.g., acme-missiles-0.3)
# extra-deps: []
```

This looks more like what we're looking for, but what the fuck is a resolver. 
At the top of `stack.yaml` it reads:

```
# Resolver to choose a 'specific' stackage snapshot or a compiler version.
# A snapshot resolver dictates the compiler version and the set of packages
# to be used for project dependencies. For example:
#
# resolver: lts-3.5
# resolver: nightly-2015-09-21
# resolver: ghc-7.10.2
# resolver: ghcjs-0.1.0_ghc-7.10.2
#
# The location of a snapshot can be provided as a file or url. Stack assumes
# a snapshot provided as a file might change, whereas a url resource does not.
#
# resolver: ./custom-snapshot.yaml
# resolver: https://example.com/snapshots/2018-01-01.yaml
resolver: lts-11.9
```
Okay so a resolver version is like a Linux distribution release version.

We still haven't figured out how to add a package as dependency. What's left `stack.yaml`?

```yaml
# Extra package databases containing global packages
# extra-package-dbs: []
```

Is this a like a ppa?\


##### \<off_topic\>
> God I'm having flash backs to my Ubuntu days.\
> And Moses said thou shalt use Arch or a distribution no one has ever heard of,
> and the people were happy until the people realized they were doing a full clone of every `*-git` aur package.\

##### \</off_topic\>

```
# Extra directories used by stack for building
# extra-include-dirs: [/path/to/dir]
# extra-lib-dirs: [/path/to/dir]
```
Non Haskell dependencies?


Well that didn't work, maybe the next config file.

##### package.yaml
**Success** an array called `dependencies` it has config hell in it, but it works!
I can add `- lens` and run `stack ghci` followed by `import Control.Lens` without an error.
```
    dependencies:
    - config-hell
    - lens
```

#### So what is config-hell.cabal?

As it turns out `package.yaml` is just an alternative format for `config-hell.cabal` provided by [hpack](https://github.com/sol/hpack).
#### Putting It all together
So, we have stack which is like a distribution of Haskell packages.
If you want to use a package from a stack resolver/distribution then you put list it as a dependency in hpack's `package.yaml` or cabal's `config-hell.cabal` file, if your not using hpack.
If you want a package that's not in the resolver/distribution you list it in `extra-deps` in `stack.yaml`.
Hey, that one made sense! You use the `stack` command to manage hpack which manages cabal to manage cabal directly.
Some stack templates use hpack some don't.
Wait what's a template?

#### Templates
##### because it's not enough to have one over complicated build system.

Templates are just lightly configured projects.
Running `stack templates` prints this beautifully documented list.

```
Template                    Description
chrisdone
foundation                - Project based on an alternative prelude with batteries and no dependencies.
franklinchen
ghcjs                     - Haskell to JavaScript compiler, based on GHC
ghcjs-old-base
hakyll-template           - a static website compiler library
haskeleton                - a project skeleton for Haskell packages
hspec                     - a testing framework for Haskell inspired by the Ruby library RSpec
new-template
protolude                 - Project using a custom Prelude based on the Protolude library
quickcheck-test-framework - a library for random testing of program properties
readme-lhs                - small scale, quick start, literate haskell projects
rio
rubik
scotty-hello-world
scotty-hspec-wai
servant                   - a set of packages for declaring web APIs at the type-level
servant-docker
simple
simple-hpack
simple-library
spock                     - a lightweight web framework
tasty-discover            - a project with tasty-discover with setup
tasty-travis
unicode-syntax-exe
unicode-syntax-lib
yesod-minimal
yesod-mongo
yesod-mysql
yesod-postgres
yesod-simple
yesod-sqlite
```

As far as I can tell this is the only documentation for each template.

Running `stack new project-name template-name` constructs a new project using the named template.
Some templates use hpack, some don't.
While I understand where they were going with this, the lack of documentation,
consistent configuration standards, and breath of choice make this a bit of a nightmare for people starting out.
`stack new test` and `stack new test simple` are identical, while `stack new test simple-hpack` is mostly identical.
Both `simple` and `simple-hpack` use hpack.
The assumption that users will realize `simple` is the default template is a pretty good one, but why make people assume at all?
Why not say the default is simple in the list of templates? 

Enough with templates! hspec looked cool. Let's try it out.

## Inconsistent and suppressing errors 
Okay let's say I have a simple project (we'll use the default templates code) and we want to try hspec tests on it.
We will make it with `stack new someFunc`.

```sh
someFunc
├── app
│  └── Main.hs
├── ChangeLog.md
├── LICENSE
├── package.yaml
├── README.md
├── Setup.hs
├── someFunc.cabal
├── src
│  └── Lib.hs
├── stack.yaml
└── test
   └── Spec.hs
```


Our  project `someFunc` has two code files: `someFunc/app/Main.hs`, and `someFunc/src/Lib.hs`.

##### Main.hs
```Haskell
module Main where
import Lib
main :: IO ()
main = someFunc
```

##### Lib.hs
```Haskell
module Lib
    ( someFunc
    ) where
someFunc :: IO ()
someFunc = putStrLn "someFunc"
```

let's take a look at the `hspec` template.
`stack new someFunc-hspec hspec` creates:
```sh
someFunc-hspec
├── app
│  └── Main.hs
├── LICENSE
├── README.md
├── Setup.hs
├── someFunc-hspec.cabal
├── src
│  └── Data
│     └── String
│        └── Strip.hs
├── stack.yaml
└── test
   ├── Data
   │  └── String
   │     └── StripSpec.hs
   └── Spec.hs
```

##### Main.hs
```Haskell
module Main where
import Data.String.Strip
main :: IO ()
main = interact strip
```

##### Strip.hs
```Haskell
module Data.String.Strip (strip)  where
import Data.Char
strip :: String -> String
strip = dropWhile isSpace . reverse . dropWhile isSpace . reverse
```

##### StripSpec.hs
```Haskell
module Data.String.StripSpec (main, spec) where
import Test.Hspec
import Test.QuickCheck
import Data.String.Strip
-- `main` is here so that this module can be run from GHCi on its own.  It is
-- not needed for automatic spec discovery.
main :: IO ()
main = hspec spec
spec :: Spec
spec = do
  describe "strip" $ do
    it "removes leading and trailing whitespace" $ do
      strip "\t  foo bar\n" `shouldBe` "foo bar"
    it "is idempotent" $ property $
      \str -> strip str === strip (strip str)
```

`stack test` produces a successful test.

Cool, but I want to test `someFunc`, so we'll copy the code files from `someFunc` into `someFunc-hspec` and delete the template's code and tests for now.
```sh
cp someFunc/app/Main.hs someFunc-hspec/app/Main.hs
cp someFunc/src/Lib.hs someFunc-hspec/src

rm -rf someFunc-hspec/src/Data
rm -rf someFunc-hspec/test/Data

cd someFunc-hspec
```

To get
```sh
.
├── app
│  └── Main.hs
├── LICENSE
├── README.md
├── Setup.hs
├── someFunc-hspec.cabal
├── src
│  └── Lib.hs
├── stack.yaml
└── test
   └── Spec.hs
```

Now we'll see if everything worked out with `stack ghci`.
```Haskell
someFunc-hspec-0.1.0.0: initial-build-steps (lib + exe)
The following GHC options are incompatible with GHCi and have not been passed to it: -threaded
Configuring GHCi with the following packages: someFunc-hspec
Using main module: 1. Package `someFunc-hspec' component exe:someFunc-hspec with main-is file: /home/host/haskell-hell/someFunc-hspec/app/Main.hs
GHCi, version 8.2.2: http://www.haskell.org/ghc/  :? for help
[1 of 2] Compiling Lib              ( /home/host/haskell-hell/someFunc-hspec/src/Lib.hs, interpreted )
[2 of 2] Compiling Main             ( /home/host/haskell-hell/someFunc-hspec/app/Main.hs, interpreted )
Ok, two modules loaded.
Loaded GHCi configuration from /tmp/haskell-stack-ghci/433033ec/ghci-script
*Main> main
someFunc
*Main>
```
Great, it all worked!

So, now we'll build it with `stack build`
```
someFunc-hspec-0.1.0.0: build (lib + exe)
Preprocessing library for someFunc-hspec-0.1.0.0..
Cabal-simple_mPHDZzAJ_2.0.1.0_ghc-8.2.2: can't find source for
Data/String/Strip in src,
.stack-work/dist/x86_64-linux-tinfo6/Cabal-2.0.1.0/build/autogen,
.stack-work/dist/x86_64-linux-tinfo6/Cabal-2.0.1.0/build/global-autogen
--  While building custom Setup.hs for package someFunc-hspec-0.1.0.0
using:
      /home/host/.stack/setup-exe-cache/x86_64-linux-tinfo6/Cabal-simple_mPHDZzAJ_2.0.1.0_ghc-8.2.2 --builddir=.stack-work/dist/x86_64-linux-tinfo6/Cabal-2.0.1.0 build lib:someFunc-hspec exe:someFunc-hspec --ghc-options " -ddump-hi -ddump-to-file -fdiagnostics-color=always"
          Process exited with code: ExitFailure 1
```

The code works in ghci, but now with stack build.

```sh
rg  'Strip'
someFunc-hspec.cabal
18:  exposed-modules:     Data.String.Strip
```
So ghci doesn't read the cabal config.
If we replace `Data.String.Strip` with `Lib`.
`stack build` is success full.

Let's make a test for `someFunc`

First, we modify `Lib.hs` by making it return a string not IO.
This is need necessary to test it as IO is very difficult to test.
##### Lib.hs
```Haskell
module Lib
    ( someFunc
    ) where

someFunc :: String
someFunc = "someFunc"
```
Then we add `putStrLn` to `Main.hs`
##### Main.hs
```Haskell
module Main where

import Lib

main :: IO ()
main = putStrLn someFunc
```

Now we have a passing test of `someFunc`!

## A Comparison with rust
`cargo` only has two project templates binary, and library.
`cargo new some_func` uses the binary template. 
```sh
some_func
├── Cargo.toml
└── src
   └── main.rs
```

We will add a `lib.rs` file to `src` containing the function and it's test.
##### lib.rs
```rust
/// some_func returns "some_func"
pub fn some_func() -> &'static str {
    "some_func"
}

#[test]
fn test_some_func() {
   assert_eq!(some_func(), "some_func");
}
```

Then we'll modify `main.rs`.
##### main.rs
```rust
mod lib;
use lib::some_func;

fn main() {
    println!("{}", some_func());
}
```

Now `cargo test` passes!

You'll notice there is only one config file.
##### Cargo.toml
```toml
[package]
name = "some_func"
version = "0.1.0"
authors = ["Avi ד <avi.the.coder@gmail.com>"]

[dependencies]
```
When we run `cargo test` or `cargo build` it generates `Cargo.lock`, While this is not as theoretically sound as stack or the cabal solver, in practice, I have doubts that any amount of time I spend on manually configuring package versions will ever match the time it takes to learn cabal and stack.

## Disclaimer
I am not intending to criticize the work done on cabal or stack. Both solve hard problems.
I intend to highlight making stack and cabal the default choice for new Haskell programmers' front loads complexity.
I would venture that the primary reason Haskell is such a niche language is because of the prioritization of theoretical soundness at the expense of ease of use.
If any one, and this is a big "If", actually wants Haskell to become more main stream, perhaps try making easier, slightly less-sound tools like cargo.
I am not a big fan of the adage 'worse is better', but some times the easy way is better than theoretically sound.

Also apologies for my attempt at humor

