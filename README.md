# Where Lions Roam: Haskell & Hardware on VELDT

## Table of Contents
1. [Section 1: Introduction & Setup](https://github.com/standardsemiconductor/VELDT-getting-started#section-1-introduction--setup)
2. [Section 2: Fiat Lux](https://github.com/standardsemiconductor/VELDT-getting-started#section-2-fiat-lux)
   1. [Learning to Count](https://github.com/standardsemiconductor/VELDT-getting-started#learning-to-count)
   2. [Its a Vibe: PWM](https://github.com/standardsemiconductor/VELDT-getting-started#its-a-vibe-pwm)
   3. [Drive: RGB Primitive](https://github.com/standardsemiconductor/VELDT-getting-started#drive-rgb-primitive)
   4. [Fiat Lux: Blinker](https://github.com/standardsemiconductor/VELDT-getting-started#fiat-lux-blinker)
3. [Section 3: Roar](https://github.com/standardsemiconductor/VELDT-getting-started#section-3-roar)
   1. [Serial for Breakfast](https://github.com/standardsemiconductor/VELDT-getting-started#serial-for-breakfast)
   2. [UART My Art](https://github.com/standardsemiconductor/VELDT-getting-started#uart-my-art)
   3. [Roar: Echo](https://github.com/standardsemiconductor/VELDT-getting-started#roar-echo)
4. [Section 4: Pride](https://github.com/standardsemiconductor/VELDT-getting-started#section-4-pride)
5. [Section 5: Where Lions Roam](https://github.com/standardsemiconductor/VELDT-getting-started#section-5-where-lions-roam)
   
**Clicking on any header within this document will return to Table of Contents** 

## [Section 1: Introduction & Setup](https://github.com/standardsemiconductor/VELDT-getting-started#table-of-contents)
This is an opinionated guide to hardware design from first principles using Haskell and VELDT. However, much of the information within this document can be useful independent of the HDL or FPGA development platform. We assume you have VELDT FPGA development board; if not, go order one from [Amazon](https://www.amazon.com/dp/B08F9T8DFT?ref=myi_title_dp). We also assume you are using Linux, but this is only for getting the tools setup and running the examples. 
  
Much of the code included in the examples is written in Haskell and compiled to Verilog using [Clash](https://clash-lang.org/). We find desiging hardware with Haskell to be an enriching experience, and if you are experimenting with HDLs or just starting out with hardware, give it a shot; over time the dividends pay well. Visit the [VELDT-info](https://github.com/standardsemiconductor/VELDT-info#clash) repo for instructions on installation and setup of Haskell and Clash tools. If you design hardware in a language other than Haskell, feel free to skip over the language specific aspects. We hope to translate the examples to other HDLs as this guide develops.
  
We use the Project IceStorm flow for synthesis, routing, and programming. These are excellent, well-maintained open source tools. For installation and setup instructions visit the [VELDT-info](https://github.com/standardsemiconductor/VELDT-info#project-icestorm) repo.

This guide is split into several sections. Each section begins with construction of sub-components then culminates with an application which utilizes the sub-components. [Section 2](https://github.com/standardsemiconductor/VELDT-getting-started#section-2-fiat-lux) constructs a simple blinker, the "hello-world" of FPGAs. [Section 3](https://github.com/standardsemiconductor/VELDT-getting-started#section-3-roar) covers serializers and deserializers which are used to construct a UART. In [Section 4](https://github.com/standardsemiconductor/VELDT-getting-started#section-4-pride) we learn how to interact with the memory provided by VELDT. Finally, in [Section 5](https://github.com/standardsemiconductor/VELDT-getting-started#section-5-where-lions-roam) we design a simple CPU with a custom ISA, then integrate our LED, UART, and memory peripherals to create a stored-program computer. By the end of the guide, you will have a library of commonly used sub-components along with a directory of applications demonstrating their usage. The library and demos explained in this guide are available in this repo, see the [veldt](https://github.com/standardsemiconductor/VELDT-getting-started/tree/master/veldt) and [demo](https://github.com/standardsemiconductor/VELDT-getting-started/tree/master/demo) directories.

Finally, if you have any suggestions, comments, discussions, edits additions etc. please open an issue in this repo. We value any and all contributions. Let's get started!
## [Section 2: Fiat Lux](https://github.com/standardsemiconductor/VELDT-getting-started#table-of-contents)
In this section we start by building a counter. Then using the counter, construct a PWM. Equipped with our counter and PWM, we use the RGB LED Driver IP to create our first running application on VELDT; a blinker!
### [Learning to Count](https://github.com/standardsemiconductor/VELDT-getting-started#table-of-contents)
We begin by creating a haskell library:
```console
foo@bar:~/VELDT-getting-started$ mkdir veldt && cd veldt
foo@bar:~/VELDT-getting-started/veldt$ cabal init
Guessing dependencies...

Generating LICENSE...
Warning: unknown license type, you must put a copy in LICENSE yourself.
Generating Setup.hs...
Generating CHANGELOG.md...
Generating Main.hs...
Generating veldt.cabal...

Warning: no synopsis given. You should edit the .cabal file and add one.
You may want to edit the .cabal file and add a Description field.
```
We can remove the `Main.hs` file:
```console
foo@bar:~/VELDT-getting-started/veldt$ rm Main.hs
```
Next, edit the `veldt.cabal`. We remove the executable section and add our own library. We leave everything else untouched. You may customize attributes like `synopsys` but it is not necessary. Your `veldt.cabal` file should look similar:
```
cabal-version:       >=1.10
-- Initial package description 'veldt.cabal' generated by 'cabal init'.
-- For further documentation, see http://haskell.org/cabal/users-guide/
                                                                                                         
name:                veldt
version:             0.1.0.0
-- synopsis:
-- description:
-- bug-reports:
-- license:
license-file:        LICENSE
author:              Standard Semiconductor
maintainer:          standard.semiconductor@gmail.com
-- copyright:
-- category:
build-type:          Simple
extra-source-files:  CHANGELOG.md

library
	exposed-modules: Veldt.Counter
        build-depends: clash-prelude,
        	       mtl,
                       lens,
		       interpolate,
                       base,
		       ghc-typelits-extra,
		       ghc-typelits-knownnat,
                       ghc-typelits-natnormalise
       	default-extensions: NoImplicitPrelude,
                            DeriveGeneric,
                            DeriveAnyClass,
                            DataKinds,
                            TemplateHaskell,
                            LambdaCase,
                            TupleSections,
                            TypeOperators,
			    QuasiQuotes,
			    ViewPatterns,
			    BinaryLiterals
        default-language: Haskell2010
	ghc-options: -Wall -fexpose-all-unfoldings -fno-worker-wrapper -fplugin=GHC.TypeLits.Extra.Solve\
r -fplugin=GHC.TypeLits.KnownNat.Solver	-fplugin=GHC.TypeLits.Normalise
```
We won't go through everything about this cabal file, but here are the highlights. `exposed-modules` are the modules we export from the library to be used in our demos. So far we see `Veldt.Counter`, we will create a directory `Veldt` with a file `Counter.hs`. This will have our counter source code. The `build-depends` section lists our library dependencies. Notably the `clash-prelude` package provides important functions and types that are crucial for hardware. We use `lens` to zoom and mutate substates. `interpolate` is used for inline primitives when we need Yosys to infer hardware IP. `base` provides standard haskell functions and types. The `ghc-typelits...` packages are plugins to help the clash compiler infer types. The next section is `default-extensions`, these help us reduce boilerplate and clean up syntax. `NoImplicitPrelude` is especially important, it says we don't want the standard Haskell prelude imported implicitly, we want to explicitly import the Clash prelude. `ghc-options` turns on warnings and activates plugins.

Create a directory `Veldt` with a file `Counter.hs`.
```console
foo@bar:~/VELDT-getting-started/veldt$ mkdir Veldt && cd Veldt
foo@bar:~/VELDT-getting-started/veldt/Veldt$ touch Counter.hs
```

Open `Counter.hs` in your favorite editor. Let's name the module, list the exports and import some useful packages:
```haskell
module Veldt.Counter
  ( Counter
  , mkCounter
  , increment
  , incrementWhen
  , incrementUnless
  , decrement
  , set
  , get
  , gets
  ) where

import Clash.Prelude
import Control.Monad.RWS (RWST)
import qualified Control.Monad.RWS as RWS
```
The exported types and functions define the API for our counter. We want to be able to `increment`, `decrement`, `set`, or `get` the counter value. Additionally, we provide `gets` (`get` with a projection) and conditional increment functions `incrementWhen` and `incrementUnless`.

Let's define the counter type, it will be polymorphic so we may use the counter in different contexts with different underlying counter types e.g. `Counter (Index 25)`, `Counter (Unsigned 32)`, or `Counter (BitVector 8)`
```haskell
newtype Counter a = Counter { unCounter :: a }
  deriving (NFDataX, Generic)
```
Clash requires we derive `NFDataX` and `Generic` for any type which will be stored as state in a register.

Next we define a function which constructs a counter. Although very simple, it highlights a style we use throughout the library; functions prefixed with "mk" are state type constructors, sometimes called "smart constructors".
```haskell
mkCounter :: a -> Counter a
mkCounter = Counter
```

The rest of the API is monadic. This is an opinion, you don't have to use monads to make a counter, but this guide does. We advise you to explore other styles, techniques and representations. That being said, we choose to represent mealy machines as Reader-Writer-State monads or RWS for short. In order to easily compose monadic functions we use the concrete RWS transformer from [mtl](http://hackage.haskell.org/package/mtl). It has the type `RWST r w s m a`. Although the counter does not use the Reader or Writer types `r` and `w`, other components in the library will and this eases composition, e.g. when we want to make a UART with both a counter and serializer. Let's define some simple functions to access or mutate our counter.
```haskell
set :: (Monoid w, Monad m) => a -> RWST r w (Counter a) m ()
set = RWS.put . Counter

get :: (Monoid w, Monad m) => RWST r w (Counter a) m a
get = RWS.gets unCounter

gets :: (Monoid w, Monad m) => (a -> b) -> RWST r w (Counter a) m b
gets f = f <$> get
```
They wrap/unwrap the newtype and use the RWST `RWS.get` and `RWS.put` functions to provide access to the underlying counter value with type `a`. We use these accessors to implement the `increment` and `decrement` functions.
```haskell
increment :: (Monoid w, Monad m, Bounded a, Enum a, Eq a) => RWST r w (Counter a) m ()
increment = do
  c <- get
  set $ if c == maxBound
    then minBound
    else succ c

decrement :: (Monoid w, Monad m, Bounded a, Enum a, Eq a) => RWST r w (Counter a) m ()
decrement = do
  c <- get
  set $ if c == minBound
    then maxBound
    else pred c
```
Here's the gist: first `get` the current value of the counter. If the value is equal to its maximum (minimum) bound then set the counter to the minimum (maximum) bound. Otherwise, `set` the counter to the value's successor (predecessor). 

The typeclass constraint `Bounded` says our counter has a minimum and maximum value which gives us `minBound` and `maxBound`. Likewise `Eq` lets us compare equality `==` and `Enum` provides `succ` (successor) and `pred` (predecessor) functions on our polymorphic type `a`. Without these constraints the compiler would complain that it could not deduce the required typeclass. Additionally, the RWS Monad `RWST r w s m a` requires `w` to be a `Monoid` and `m` a `Monad`, this will be important later when we "run" our monadic action.

When designing your own counters be careful when using `succ` or `pred`. For example `succ 0 == (1 :: BitVector 8)` and `pred 4 == (3 :: Index 6)`, but `succ (4 :: Index 5)` is undefined and out of bounds (**DO NOT DO THIS**) because the type `Index 5` only has inhabitants `0`,`1`,`2`,`3`, and `4`; that is why we check for `maxBound` and `minBound` states in `increment` and `decrement`.

Finally, we use our new `increment` function to implement a conditional increment `incrementWhen` and `incrementUnless`. The former will increment when a predicate is `True`, the latter when `False`.
```haskell
incrementWhen
  :: (Monoid w, Monad m, Bounded a, Enum a, Eq a)
  => (a -> Bool)
  -> RWST r w (Counter a) m ()
incrementWhen p = do
  b <- gets p
  if b
    then increment
    else set minBound

incrementUnless
  :: (Monoid w, Monad m, Bounded a, Enum a, Eq a)
  => (a -> Bool)
  -> RWST r w (Counter a) m ()
incrementUnless p = incrementWhen (not . p)
```
Within `incrementWhen`, we get the counter value and apply our predicate. If the predicate evaluates to `True`, `b` is bound to `True` and we increment the counter. Otherwise, `b` is bound to `False` and we set the value of the counter to its minimum bound. To reduce and reuse code, we implement `incrementUnless` using `incrementWhen` and post-compose `not` to our predicate. Suppose we have `incrementUnless (== 3) :: RWST r w (Counter (Index 8)) m ()`, then the states of the counter would be: ... 0 1 2 3 0 1 2 3 0 1 2 3 ...

Here is our completed counter:
```haskell
module Veldt.Counter
  ( Counter
  , mkCounter
  , increment
  , incrementWhen
  , incrementUnless
  , decrement
  , set
  , get
  , gets
  ) where

import Clash.Prelude
import Control.Monad.RWS (RWST)
import qualified Control.Monad.RWS as RWS

-------------
-- Counter --
-------------
newtype Counter a = Counter { unCounter :: a }
  deriving (NFDataX, Generic)

mkCounter :: a -> Counter a
mkCounter = Counter

set :: (Monoid w, Monad m) => a -> RWST r w (Counter a) m ()
set = RWS.put . Counter

get :: (Monoid w, Monad m) => RWST r w (Counter a) m a
get = RWS.gets unCounter

gets :: (Monoid w, Monad m) => (a -> b) -> RWST r w (Counter a) m b
gets f = f <$> get

increment :: (Monoid w, Monad m, Bounded a, Enum a, Eq a) => RWST r w (Counter a) m ()
increment = do
  c <- get
  set $ if c == maxBound
    then minBound
    else succ c

decrement :: (Monoid w, Monad m, Bounded a, Enum a, Eq a) => RWST r w (Counter a) m ()
decrement = do
  c <- get
  set $ if c == minBound
    then maxBound
    else pred c

incrementWhen
  :: (Monoid w, Monad m, Bounded a, Enum a, Eq a)
  => (a -> Bool)
  -> RWST r w (Counter a) m ()
incrementWhen p = do
  b <- gets p
  if b
    then increment
    else set minBound

incrementUnless
  :: (Monoid w, Monad m, Bounded a, Enum a, Eq a)
  => (a -> Bool)
  -> RWST r w (Counter a) m ()
incrementUnless p = incrementWhen (not . p)
```
To end this part, we clean and rebuild the library. You should not see any errors.
```console
foo@bar:~/VELDT-getting-started/veldt$ cabal clean
foo@bar:~/VELDT-getting-started/veldt$ cabal build
Resolving dependencies...
Build profile: -w ghc-8.8.3 -O1
In order, the following will be built (use -v for more details):
 - veldt-0.1.0.0 (lib) (configuration changed)
Configuring library for veldt-0.1.0.0..
Warning: The 'license-file' field refers to the file 'LICENSE' which does not
exist.
Preprocessing library for veldt-0.1.0.0..
Building library for veldt-0.1.0.0..
[1 of 1] Compiling Veldt.Counter    ( Veldt/Counter.hs, /home/foo/VELDT-getting-started/veldt/dist-newstyle/build/x86_64-linux/ghc-8.8.3/veldt-0.1.0.0/build/Veldt/Counter.o ) [flags changed]
```
You can find the full counter source code [here](https://github.com/standardsemiconductor/VELDT-getting-started/blob/master/veldt/Veldt/Counter.hs). We can now use our counter to create a PWM.
### [Its a Vibe: PWM](https://github.com/standardsemiconductor/VELDT-getting-started#table-of-contents)
Pulse Width Modulation or PWM is used to drive our LED. We use a technique called [time proportioning](https://en.wikipedia.org/wiki/Pulse-width_modulation#Time_proportioning) to generate the PWM signal with our counter. To begin let's create a `PWM.hs` file in the `Veldt` directory.
```console
foo@bar:~/VELDT-getting-started/veldt/Veldt$ touch PWM.hs
```
We also need to expose the PWM module with cabal by editing the `exposed-modules` section of `veldt.cabal` to include `Veldt.PWM`.
```
......
library
        exposed-modules: Veldt.Counter,
	                 Veldt.PWM
......
```
Now begin editing the `PWM.hs` file. We start by naming the module, defining our exports, and importing useful packages.
```haskell
module Veldt.PWM
  ( PWM
  , mkPWM
  , pwm
  , setDuty
  ) where

import Clash.Prelude
import Control.Lens
import Control.Monad.RWS
import qualified Veldt.Counter as C
```
We export the type `PWM` and its smart constructor `mkPWM`. The monadic API consists of `pwm`, a PWM action, and a setter `setDuty` to mutate the duty cycle. In this module we will be using [lens](https://hackage.haskell.org/package/lens) to mutate and zoom sub-states. Again, we use the RWS monad. Finally we import our counter as qualified so as not to conflict `RWS`'s `get` with `Counter`'s `get`. This means whenever we want to use an exported function from `Veldt.Counter` we must prefix it with `C.`.

Next we define the `PWM` type and its constructor. Note how we use `makeLenses` to automatically create lenses for our `PWM` type.
```haskell
data PWM a = PWM
  { _ctr  :: C.Counter a
  , _duty :: a
  } deriving (NFDataX, Generic)
makeLenses ''PWM

mkPWM :: Bounded a => a -> PWM a
mkPWM d = PWM (C.mkCounter minBound) d
```
The PWM state consists of a counter and a value used to control the duty cycle. Also, note that we keep `PWM` polymorphic, just like our counter. Our smart constructor creates a PWM with an initial duty cycle and a counter with an initial value set to the minimum bound. 

Let's define and implement `setDuty` which will update the `duty` cycle and reset the counter.
```haskell
setDuty :: (Monoid w, Monad m, Bounded a) => a -> RWST r w (PWM a) m ()
setDuty d = do
  duty .= d
  zoom ctr $ C.set minBound
```
We use the `.=` lens operator to set the `duty` cycle, then we use `zoom` to `set` the counter to its minimum bound. We use `setDuty` to change the duty cycle of the PWM. For example, suppose we have `setDuty 25 :: RWST r w (PWM (Index 100)) m ()`, then the PWM will operate at 25% duty cycle.

Finally, we tackle the `pwm` function.
```haskell
pwm :: (Monoid w, Monad m, Ord a, Bounded a, Enum a) => RWST r w (PWM a) m Bit
pwm = do
  d <- use duty
  c <- zoom ctr C.get
  zoom ctr C.increment
  return $ boolToBit $ c < d
```
First we bind `duty` to `d` and the counter value to `c`. Next we `increment` the counter. Last, we compare `c < d`, convert the `boolToBit`, and `return` the bit. `boolToBit` simply maps `True` to `1 :: Bit` and `False` to `0 :: Bit`. Because we compare the `duty` `d` to the counter `c` with `<`, our type signature requires the underlying counter type `a` to be a member of the `Ord` typeclass. For example, if we have `pwm :: RWST r w (PWM (Index 4)) m Bit` and `duty` is bound to `3 :: Index 4`, (75% duty cycle, remeber `Index 4` has inhabitants 0, 1, 2, 3), the output of `pwm` when run as a mealy machine would be: ... 1, 1, 1, 0, 1, 1, 1, 0, ... .

Here is the complete `PWM.hs` source code:
```haskell
module Veldt.PWM
  ( PWM
  , mkPWM
  , pwm
  , setDuty
  ) where

import Clash.Prelude
import Control.Lens
import Control.Monad.RWS
import qualified Veldt.Counter as C

---------
-- PWM --
---------
data PWM a = PWM
  { _ctr  :: C.Counter a
  , _duty :: a
  } deriving (NFDataX, Generic)
makeLenses ''PWM

mkPWM :: Bounded a => a -> PWM a
mkPWM d = PWM (C.mkCounter minBound) d

setDuty :: (Monoid w, Monad m, Bounded a) => a -> RWST r w (PWM a) m ()
setDuty d = do
  duty .= d
  zoom ctr $ C.set minBound

pwm :: (Monoid w, Monad m, Ord a, Bounded a, Enum a) => RWST r w (PWM a) m Bit
pwm = do
  d <- use duty
  c <- zoom ctr C.get
  zoom ctr C.increment
  return $ boolToBit $ c < d
```
To end this part, we clean and rebuild the library. You should not see any errors.
```console
foo@bar:~/VELDT-getting-started/veldt$ cabal clean && cabal build
Resolving dependencies...
Build profile: -w ghc-8.8.3 -O1
In order, the following will be built (use -v for more details):
 - veldt-0.1.0.0 (lib) (first run)
Configuring library for veldt-0.1.0.0..
Warning: The 'license-file' field refers to the file 'LICENSE' which does not
exist.
Preprocessing library for veldt-0.1.0.0..
Building library for veldt-0.1.0.0..
[1 of 2] Compiling Veldt.Counter    ( Veldt/Counter.hs, /home/foo/VELDT-getting-started/veldt/dist-newstyle/build/x86_64-linux/ghc-8.8.3/veldt-0.1.0.0/build/Veldt/Counter.o )
[2 of 2] Compiling Veldt.PWM        ( Veldt/PWM.hs, /home/foo/VELDT-getting-started/veldt/dist-newstyle/build/x86_64-linux/ghc-8.8.3/veldt-0.1.0.0/build/Veldt/PWM.o )
```
You can find the full PWM source code [here](https://github.com/standardsemiconductor/VELDT-getting-started/blob/master/veldt/Veldt/PWM.hs). In the next part, we use a Clash primitive to infer Lattice RGB Driver IP.
### [Drive: RGB Primitive](https://github.com/standardsemiconductor/VELDT-getting-started#table-of-contents)
We need one more component before starting our demo. This component is a RGB LED Driver; it takes 3 PWM signals (R, G, B) to drive the LED. Because the RGB Driver is a Lattice IP block, we need our compiled Haskell code to take a certain form in Verilog. When we synthesize the demo, Yosys will infer the Lattice Ice40 RGB Driver IP (SB_RGBA_DRV) from the Verilog code. In order to have Clash use a certain Verilog (or VHDL) code, we write a primitive. This primitive tells the Clash compiler to insert Verilog (or VHDL) instead of compiling our function. Let's begin by creating a directory `Ice40` for our Lattice primitives. This will be within the `Veldt` directory. Then we create a `Rgb.hs` file which will be our RGB Driver primitive.
```console
foo@bar:~/VELDT-getting-started/veldt$ mkdir Veldt/Ice40 && touch Veldt/Ice40/Rgb.hs
```
Next add the `Veldt.Ice40.Rgb` to our `veldt.cabal` `exposed-modules` list.
```
...
exposed-modules: Veldt.Counter,
                 Veldt.PWM,
                 Veldt.Ice40.Rgb
...
```
Now edit `Rgb.hs`. We inline the Verilog primitive (meaning we have Verilog and Haskell in the same module), and then wrap it with a function to ease usage. Let's start by naming the module, its exports, and its imports.
```haskell
module Veldt.Ice40.Rgb
  ( Rgb
  , rgbDriver
  ) where

import Clash.Prelude
import Clash.Annotations.Primitive
import Data.String.Interpolate (i)
import Data.String.Interpolate.Util (unindent)
```
We export the `Rgb` type which is the input/output type of our primitive and a wrapper function `rgbDriver` for the primitive. Additionally we import `Clash.Annotations.Primitive` which supplies code for writing primitives. Since the primtive will be inlined we use the [interpolate](https://hackage.haskell.org/package/interpolate) package for string interpolation.

Now we create the primtive.
```haskell
{-# ANN rgbPrim (InlinePrimitive [Verilog] $ unindent [i|
  [ { "BlackBox" :
      { "name" : "Veldt.Ice40.Rgb.rgbPrim"
      , "kind" : "Declaration"
      , "type" :
  "rgbPrim
  :: String         -- current_mode ARG[0]
  -> String         -- rgb0_current ARG[1]
  -> String         -- rgb1_current ARG[2]
  -> String         -- rgb2_current ARG[3]
  -> Signal dom Bit -- pwm_r        ARG[4]
  -> Signal dom Bit -- pwm_g        ARG[5]
  -> Signal dom Bit -- pwm_b        ARG[6]
  -> Signal dom (Bit, Bit, Bit)"
      , "template" :
  "//SB_RGBA_DRV begin
  wire ~GENSYM[RED][0];
  wire ~GENSYM[GREEN][1];
  wire ~GENSYM[BLUE][2];

  SB_RGBA_DRV #(
     .CURRENT_MODE(~ARG[0]),
     .RGB0_CURRENT(~ARG[1]),
     .RGB1_CURRENT(~ARG[2]),
     .RGB2_CURRENT(~ARG[3])
  ) RGBA_DRIVER (
     .CURREN(1'b1),
     .RGBLEDEN(1'b1),
     .RGB0PWM(~ARG[4]),
     .RGB1PWM(~ARG[5]),
     .RGB2PWM(~ARG[6]),
     .RGB0(~SYM[0]),
     .RGB1(~SYM[1]),
     .RGB2(~SYM[2])
  );
 
  assign ~RESULT = {~SYM[0], ~SYM[1], ~SYM[2]};
  //SB_RGBA_DRV end"
      }
    } 
  ]
  |]) #-}
```
When writing primitives be sure the function name, module name, and black box name all match. The template is Verilog from the Lattice documentation [iCE40 LED Driver Usage Guide](https://github.com/standardsemiconductor/VELDT-info/blob/master/ICE40LEDDriverUsageGuide.pdf). The documentation for writing primitives is on the [clash-prelude](https://hackage.haskell.org/package/clash-prelude) hackage page in the `Clash.Annotations.Primitive` module. Basically, the `SB_RGBA_DRV` module takes 3 PWM input signals and outputs 3 LED driver signals. We adopt the style to prefix any primitive functions with `Prim`. Let's give a Haskell function stub for the primitive.
```haskell
{-# NOINLINE rgbDriverPrim #-}
rgbPrim
  :: String
  -> String
  -> String
  -> String
  -> Signal dom Bit
  -> Signal dom Bit
  -> Signal dom Bit
  -> Signal dom (Bit, Bit, Bit)
rgbPrim _ _ _ _ _ _ _ = pure (0, 0, 0)
```
Although we do not provide a real implementation for the the primitive in Haskell, it is good practice to do so and helps when testing and modeling. Also, note the type of `rgbPrim` matches exactly to the inlined primitive type and has a `NOINLINE` annotation.

Instead of constantly writing `(Bit, Bit, Bit)` for our RGB tuple, let's define a type synonym with some tags which are useful when constraining pins.
```haskell
type Rgb = ("red" ::: Bit, "green" ::: Bit, "blue" ::: Bit)
```
Finally, using our `Rgb` type, we wrap the primitive and give it some default parameters.
```haskell
rgb :: Signal dom Rgb -> Signal dom Rgb
rgb rgbPWM = let (r, g, b) = unbundle rgb
             in rgbPrim "0b0" "0b111111" "0b111111" "0b111111" r g b
```
`unbundle` is part of a `Signal` isomorphism, the other part being `bundle`. In this case, `unbundle` maps the type `Signal dom (Bit, Bit, Bit)` to `(Signal dom Bit, Signal dom Bit, Signal dom Bit)`. The `String` parameters we give to `rgbPrim` define the current and mode outputs for the driver. It may be prudent to adjust these parameters depending on the power requirements of your application. It is a good exercise to define a custom current/mode data type and use that in the wrapper `rgb` for easy usage.

Here is the complete `Rgb.hs` source code:
```haskell
module Veldt.Ice40.Rgb
  ( Rgb
  , rgb
  ) where

import Clash.Prelude
import Clash.Annotations.Primitive
import Data.String.Interpolate (i)
import Data.String.Interpolate.Util (unindent)

{-# ANN rgbPrim (InlinePrimitive [Verilog] $ unindent [i|
  [ { "BlackBox" :
      { "name" : "Veldt.Ice40.Rgb.rgbPrim"
      , "kind" : "Declaration"
      , "type" :
  "rgbPrim
  :: String         -- current_mode ARG[0]
  -> String         -- rgb0_current ARG[1]
  -> String         -- rgb1_current ARG[2]
  -> String         -- rgb2_current ARG[3]
  -> Signal dom Bit -- pwm_r        ARG[4]
  -> Signal dom Bit -- pwm_g        ARG[5]
  -> Signal dom Bit -- pwm_b        ARG[6]
  -> Signal dom (Bit, Bit, Bit)"
      , "template" :
  "//SB_RGBA_DRV begin
  wire ~GENSYM[RED][0];
  wire ~GENSYM[GREEN][1];
  wire ~GENSYM[BLUE][2];

  SB_RGBA_DRV #(
     .CURRENT_MODE(~ARG[0]),
     .RGB0_CURRENT(~ARG[1]),
     .RGB1_CURRENT(~ARG[2]),
     .RGB2_CURRENT(~ARG[3])
  ) RGBA_DRIVER (
     .CURREN(1'b1),
     .RGBLEDEN(1'b1),
     .RGB0PWM(~ARG[4]),
     .RGB1PWM(~ARG[5]),
     .RGB2PWM(~ARG[6]),
     .RGB0(~SYM[0]),
     .RGB1(~SYM[1]),
     .RGB2(~SYM[2])
  );
 
  assign ~RESULT = {~SYM[0], ~SYM[1], ~SYM[2]};
  //SB_RGBA_DRV end"
      }
    } 
  ]
  |]) #-}

{-# NOINLINE rgbPrim #-}
rgbPrim
  :: String
  -> String
  -> String
  -> String
  -> Signal dom Bit
  -> Signal dom Bit
  -> Signal dom Bit
  -> Signal dom (Bit, Bit, Bit)
rgbPrim _ _ _ _ _ _ _ = pure (0, 0, 0)

type Rgb = ("red" ::: Bit, "green" ::: Bit, "blue" ::: Bit)

rgb :: Signal dom Rgb -> Signal dom Rgb
rgb rgbPWM = let (r, g, b) = unbundle rgbPWM
             in rgbPrim "0b0" "0b111111" "0b111111" "0b111111" r g b
```

To end this part, we clean and rebuild the library. You should not see any errors.
```console
foo@bar:~/VELDT-getting-started/veldt$ cabal clean && cabal build
Resolving dependencies...
Build profile: -w ghc-8.8.3 -O1
In order, the following will be built (use -v for more details):
 - veldt-0.1.0.0 (lib) (first run)
Configuring library for veldt-0.1.0.0..
Warning: The 'license-file' field refers to the file 'LICENSE' which does not
exist.
Preprocessing library for veldt-0.1.0.0..
Building library for veldt-0.1.0.0..
[1 of 3] Compiling Veldt.Counter    ( Veldt/Counter.hs, /home/foo/VELDT-getting-started/veldt/dist-newstyle/build/x86_64-linux/ghc-8.8.3/veldt-0.1.0.0/build/Veldt/Counter.o )
[2 of 3] Compiling Veldt.Ice40.Rgb ( Veldt/Ice40/Rgb.hs, /home/foo/VELDT-getting-started/veldt/dist-newstyle/build/x86_64-linux/ghc-8.8.3/veldt-0.1.0.0/build/Veldt/Ice40/Rgb.o )
[3 of 3] Compiling Veldt.PWM        ( Veldt/PWM.hs, /home/foo/VELDT-getting-started/veldt/dist-newstyle/build/x86_64-linux/ghc-8.8.3/veldt-0.1.0.0/build/Veldt/PWM.o )
```
You can find the full RGB Driver source code [here](https://github.com/standardsemiconductor/VELDT-getting-started/blob/master/veldt/Veldt/Ice40/Rgb.hs). We now move onto creating a blinker.
### [Fiat Lux: Blinker](https://github.com/standardsemiconductor/VELDT-getting-started#table-of-contents)
This is our first demo, we will use our PWM to blink an LED; starting with the LED off, it will light up red, green, then blue and cycle back to off before repeating. Let's begin by setting up a directory for our demos, then setup a blinker demo with cabal.
```console
foo@bar:~/VELDT-getting-started$ mkdir -p demo/blinker && cd demo/blinker && cabal init
```
The `blinker.cabal` file will look similar to `veldt.cabal`, except our top-level module `Blinker` is the only exposed module, and we add `clash-ghc` and `veldt` to our `build-depends` list. Here is the complete `blinker.cabal`.
```
cabal-version:       >=1.10
-- Initial package description 'blinker.cabal' generated by 'cabal init'.
-- For further documentation, see http://haskell.org/cabal/users-guide/
                                                                                            
name:                blinker
version:             0.1.0.0
-- synopsis:
-- description:
-- bug-reports:
-- license:
license-file:        LICENSE
author:              Standard Semiconductor
maintainer:          standard.semiconductor@gmail.com
-- copyright:
-- category:
build-type:          Simple
extra-source-files:  CHANGELOG.md

library
        exposed-modules: Blinker
        build-depends: clash-prelude,
                       clash-ghc,
                       veldt,
                       mtl,
                       lens,
                       interpolate,
                       base,
                       ghc-typelits-extra,
                       ghc-typelits-knownnat,
                       ghc-typelits-natnormalise
        default-extensions: NoImplicitPrelude,
                            DeriveGeneric,
                            DeriveAnyClass,
                            DataKinds,
                            TemplateHaskell,
                            LambdaCase,
                            TupleSections,
                            TypeOperators,
                            QuasiQuotes
        default-language: Haskell2010
        ghc-options: -Wall -fexpose-all-unfoldings -fno-worker-wrapper -fplugin=GHC.TypeLits.Extra.Solver -fplugin=GHC.TypeLits.KnownNat.Solver -fplugin=GHC.TypeLits.Normalise
```
Additionally, we tell cabal where to find our `veldt` library with a `cabal.project` file.
```
packages: ./blinker.cabal,
          ../../veldt/veldt.cabal
```
Note, you may need to change the filepath to `veldt.cabal` depending on your file locations.
With that out of the way, let's create a `Blinker.hs` file and open the file with a text editor.
```console
foo@bar:~/VELDT-getting-started/demo/blinker$ touch Blinker.hs
```

We start by naming our module and importing dependencies.
```haskell
module Blinker where

import Clash.Prelude
import Clash.Annotations.TH
import Control.Monad.RWS
import Control.Lens
import qualified Veldt.Counter   as C
import qualified Veldt.PWM       as P
import qualified Veldt.Ice40.Rgb as R
```
We find that using qualified imports helps to quickly determine which modules functions and types originate from when reading source code or looking up type signatures. `Clash.Annotations.TH` includes functions to name the top entity module which is useful for synthesis.

Let's define some types to get a feel for the state space.
```haskell
type Byte = BitVector 8

data Color = Off | Red | Green | Blue | White
  deriving (NFDataX, Generic, Enum)

data Blinker = Blinker
  { _color    :: Color
  , _redPWM   :: P.PWM Byte
  , _greenPWM :: P.PWM Byte
  , _bluePWM  :: P.PWM Byte
  , _timer    :: C.Counter (Unsigned 25)
  } deriving (NFDataX, Generic)
makeLenses ''Blinker

mkBlinker :: Blinker
mkBlinker = Blinker
  { _color    = Off
  , _redPWM   = P.mkPWM 0
  , _greenPWM = P.mkPWM 0
  , _bluePWM  = P.mkPWM 0
  , _timer    = C.mkCounter 0
  } 
```
The blinker needs a color, three PWMs (one to drive each RGB signal), and a timer which will indicate when the color should change. We also create the `mkBlinker` smart constructor which initializes the color to `Off` and sets each PWM duty cycle to `0` and the timer to `0`. We derive `Enum` for `Color` so we can use `succ`, e.g. `succ Red == Green`.

Next, we create a `toPWM` function to convert a `Color` into its RGB triple which we use to set the PWM duty cycles.
```haskell
toPWM :: Color -> (Byte, Byte, Byte)
toPWM Off   = (0,    0,    0   )
toPWM Red   = (0xFF, 0,    0   )
toPWM Green = (0,    0xFF, 0   )
toPWM Blue  = (0,    0,    0xFF)
toPWM White = (0xFF, 0xFF, 0xFF)
```
The next function `blinkerM` is the core of our demo. Here is the implementation.
```haskell
blinkerM :: RWS r () Blinker R.Rgb
blinkerM = do
  r <- zoom redPWM   P.pwm
  g <- zoom greenPWM P.pwm
  b <- zoom bluePWM  P.pwm
  timerDone <- zoom timer $ C.gets isTwoSeconds
  zoom timer $ C.incrementUnless isTwoSeconds
  when timerDone $ do
    c' <- color <%= nextColor
    let (redDuty', greenDuty', blueDuty') = toPWM c'
    zoom redPWM   $ P.setDuty redDuty'
    zoom greenPWM $ P.setDuty greenDuty'
    zoom bluePWM  $ P.setDuty blueDuty'
  return (r, g, b)
  where
    isTwoSeconds = (== 24000000)
    nextColor White = Off
    nextColor c     = succ c
```
First we run each PWM and bind the output `Bit` to `r`, `g`, and `b`. Next, we get the current timer value and check if it equals 24,000,000 and bind the `Bool` to `timerDone`. Because the clock has a frequency of 12Mhz and the timer increments every cycle, counting to 24,000,000 takes two seconds. Having checked the timer, we use `incrementUnless`. Remember, this checks if the timer equals 24,000,000 in which case the timer resets to 0, otherwise it increments. When `timerDone` is bound to `True`, we change the `color` and then update each PWM's duty cycle. To update the color we use the `<%=` operator from the lens library. It modifies the value in focus and returns the new value which we bind to `c'`. Next we apply `toPWM` and bind the updated duty cycles. Then, we update each PWM duty cycle using `setDuty`. Finally we `return` the PWM outputs that were bound at the start of `blinkerM`. The `nextColor` function takes advantage of the fact that `Color` derives `Enum`, we just need to manually map `White` to `Off` to avoid going out of bounds.

Now we need to run `blinkerM` as a mealy machine. This requires the use of [`mealy`](http://hackage.haskell.org/package/clash-prelude-1.2.4/docs/Clash-Prelude.html#v:mealy) from the Clash Prelude. [`mealy`](http://hackage.haskell.org/package/clash-prelude-1.2.4/docs/Clash-Prelude.html#v:mealy) takes a transfer function of type `s -> i -> (s, o)` and an initial state then produces a function of type `HiddenClockResetEnable dom => Signal dom i -> Signal dom o`.
```haskell
blinker :: HiddenClockResetEnable dom => Signal dom R.Rgb
blinker = R.rgb $ mealy blinkerMealy mkBlinker $ pure ()
  where
    blinkerMealy s i = let (a, s', _) = runRWS blinkerM i s
		       in (s', a)
```
First, we transform our `blinkerM :: RWS r () Blinker R.Rgb` into a transfer function `blinkerMealy` with type `Blinker -> () -> (Blinker, R.Rgb)` using `runRWS`. We use the unit `()` to describe no input. Then we use `mkBlinker` to construct the initial state. Finally, we apply a unit signal as input and apply the mealy output directly to the RGB Driver IP.

Finally, we define the `topEntity` function which takes a clock as input and outputs a `Signal` of RGB LED driver.
```haskell
{-# NOINLINE topEntity #-}
topEntity
  :: "clk" ::: Clock XilinxSystem
  -> "led" ::: Signal XilinxSystem R.Rgb
topEntity clk = withClockResetEnable clk rst enableGen blinker
  where
    rst = unsafeFromHighPolarity $ pure False
makeTopEntityWithName 'topEntity "Blinker"
```
First, every top entity function has the `NOINLINE` annotation. Although this is a Lattice FPGA, it just so happens that the `XilinxSystem` domain also works. Domains describe things such as reset polarity and clock period and active edge. More information about domains is found in the `Clash.Signal` module. `XilinxSystem` specifies active-high resets, therefore we define a `rst` signal which is always inactive by inputting `False` to [`unsafeFromHighPolarity`](http://hackage.haskell.org/package/clash-prelude-1.2.4/docs/Clash-Signal.html#v:withClockResetEnable). `blinker` has a `HiddenClockResetEnable` constraint so we use [`withClockResetEnable`](http://hackage.haskell.org/package/clash-prelude-1.2.4/docs/Clash-Signal.html#v:withClockResetEnable) to expose them and provide clock, reset, and enable signals. We use the template haskell function [`makeTopEntityWithName`](http://hackage.haskell.org/package/clash-prelude-1.2.4/docs/Clash-Annotations-TH.html#v:makeTopEntityWithName) which will generate synthesis boilerplate and name the top module and its ports in Verilog. The inputs and outputs of the `topEntity` function will be constrained by the `.pcf`, or pin constraint file.

Here is the complete `Blinker.hs` source code:
```haskell
module Blinker where

import Clash.Prelude
import Clash.Annotations.TH
import Control.Monad.RWS
import Control.Lens
import qualified Veldt.Counter   as C
import qualified Veldt.PWM       as P
import qualified Veldt.Ice40.Rgb as R

type Byte = BitVector 8

data Color = Off | Red | Green | Blue | White
  deriving (NFDataX, Generic, Enum)

data Blinker = Blinker
  { _color    :: Color
  , _redPWM   :: P.PWM Byte
  , _greenPWM :: P.PWM Byte
  , _bluePWM  :: P.PWM Byte
  , _timer    :: C.Counter (Unsigned 25)
  } deriving (NFDataX, Generic)
makeLenses ''Blinker

mkBlinker :: Blinker
mkBlinker = Blinker
  { _color    = Off
  , _redPWM   = P.mkPWM 0
  , _greenPWM = P.mkPWM 0
  , _bluePWM  = P.mkPWM 0
  , _timer    = C.mkCounter 0
  }

toPWM :: Color -> (Byte, Byte, Byte)
toPWM Off   = (0,    0,    0   )
toPWM Red   = (0xFF, 0,    0   )
toPWM Green = (0,    0xFF, 0   )
toPWM Blue  = (0,    0,    0xFF)
toPWM White = (0xFF, 0xFF, 0xFF)

blinkerM :: RWS r () Blinker R.Rgb
blinkerM = do
  r <- zoom redPWM   P.pwm
  g <- zoom greenPWM P.pwm
  b <- zoom bluePWM  P.pwm
  timerDone <- zoom timer $ C.gets isTwoSeconds
  zoom timer $ C.incrementUnless isTwoSeconds
  when timerDone $ do
    c' <- color <%= nextColor
    let (redDuty', greenDuty', blueDuty') = toPWM c'
    zoom redPWM   $ P.setDuty redDuty'
    zoom greenPWM $ P.setDuty greenDuty'
    zoom bluePWM  $ P.setDuty blueDuty'
  return (r, g, b)
  where
    isTwoSeconds = (== 24000000)
    nextColor White = Off
    nextColor c     = succ c

blinker :: HiddenClockResetEnable dom => Signal dom R.Rgb
blinker = R.rgb $ mealy blinkerMealy mkBlinker $ pure ()
  where
    blinkerMealy s i = let (a, s', ()) = runRWS blinkerM i s
                       in (s', a)

{-# NOINLINE topEntity #-}
topEntity
  :: "clk" ::: Clock XilinxSystem
  -> "led" ::: Signal XilinxSystem R.Rgb
topEntity clk = withClockResetEnable clk rst enableGen blinker
  where
    rst = unsafeFromHighPolarity $ pure False
makeTopEntityWithName 'topEntity "Blinker"
```

We need a `.pcf` file to connect the FPGA ports to our design ports. Keep in mind that `Rgb` is annotated with `red`, `green`, and `blue`. Thus, our only input is `clk`, and our three outputs are `led_red`, `led_green`, `led_blue`. Here is the [Blinker.pcf](https://github.com/standardsemiconductor/VELDT-getting-started/blob/master/demo/blinker/Blinker.pcf).
```
set_io clk 35 # iot_46b_g0 12Mhz Xtal

set_io led_blue  41 # rgb2 blue
set_io led_green 40 # rgb1 green
set_io led_red   39 # rgb0 red
```
The `#` indicates anything after it is a comment. We provide a [default pin constraint file](https://github.com/standardsemiconductor/VELDT-getting-started/blob/master/demo/pcf_generic.pcf) with helpful comments in the [demo](https://github.com/standardsemiconductor/VELDT-getting-started/tree/master/demo) directory; just remove the first `#` and change the pin name to suit your design.

Finally, we provide a [Makefile](https://github.com/standardsemiconductor/VELDT-getting-started/blob/master/demo/blinker/Makefile) along with a [generic version](https://github.com/standardsemiconductor/VELDT-getting-started/blob/master/demo/Makefile_generic) in the [demo](https://github.com/standardsemiconductor/VELDT-getting-started/tree/master/demo) directory. This automates building the Haskell code with cabal, compiling with Clash, synthesizing with Yosys, place-and-route with NextPNR, bitstream packing with icepack, and bitstream programming with iceprog. Specifically, `make build` just calls `cabal build`, `make` will build with cabal, synthesize, and place-and-route. `make prog` will program the bitstream to VELDT. `make clean` cleans all build files. Information about automatic variables such as `$<` and `$@` can be found [here](https://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html). Be sure `TOP` is assigned the same value as provided to `makeTopEntityWithName`.
```make
TOP := Blinker

all: $(TOP).bin

$(TOP).bin: $(TOP).asc
        icepack $< $@

$(TOP).asc: $(TOP).json $(TOP).pcf
        nextpnr-ice40 --up5k --package sg48 --pcf $(TOP).pcf --asc $@ --json $<

$(TOP).json: $(TOP).hs
        cabal build $<
        cabal exec -- clash --verilog $<
        yosys -q -p "synth_ice40 -top $(TOP) -json $@ -dsp -abc2" verilog/$(TOP)/$(TOP)/*.v

prog: $(TOP).bin
        iceprog $<

build: $(TOP).hs
        cabal build $<

clean:
        rm -rf verilog/
        rm -f $(TOP).json
        rm -f $(TOP).asc
        rm -f $(TOP).bin
        rm -f *~
        rm -f *.hi
        rm -f *.o
        cabal clean

.PHONY: all clean prog build
```

To end this section, we build, synthesize, place-and-route, pack, and program VELDT. There should be no build errors. Verify your device utilisation looks similar, including usage of SB_RGBA_DRV. Before programming, make sure VELDT is connected to your computer, the power switch is ON, and the mode switch is set to FLASH. After programming, make sure the LED blinks with the correct color order with the intended 2 second period. If the CDONE LED is not illuminated blue, try pressing the reset button and/or toggling the power switch. If you have any issues, questions, or suggestions please open a public issue in this repository or contact us privately at standard.semiconductor@gmail.com.
```console
foo@bar:~/VELDT-getting-started/demo/blinker$ make clean && make prog
.....
Info: Device utilisation:
Info: 	         ICESTORM_LC:   198/ 5280     3%
Info: 	        ICESTORM_RAM:     0/   30     0%
Info: 	               SB_IO:     1/   96     1%
Info: 	               SB_GB:     3/    8    37%
Info: 	        ICESTORM_PLL:     0/    1     0%
Info: 	         SB_WARMBOOT:     0/    1     0%
Info: 	        ICESTORM_DSP:     0/    8     0%
Info: 	      ICESTORM_HFOSC:     0/    1     0%
Info: 	      ICESTORM_LFOSC:     0/    1     0%
Info: 	              SB_I2C:     0/    2     0%
Info: 	              SB_SPI:     0/    2     0%
Info: 	              IO_I3C:     0/    2     0%
Info: 	         SB_LEDDA_IP:     0/    1     0%
Info: 	         SB_RGBA_DRV:     1/    1   100%
Info: 	      ICESTORM_SPRAM:     0/    4     0%
.....
```
## [Section 3: Roar](https://github.com/standardsemiconductor/VELDT-getting-started#table-of-contents)
In this section we start by building a serializer and deserializer. Then, with a serializer and deserializer along with a counter we construct a UART. Equipped with our UART, we create a demo which echoes its input.
### [Serial for Breakfast](https://github.com/standardsemiconductor/VELDT-getting-started#table-of-contents)
Let's begin by creating a file `Serial.hs` in the `Veldt` directory.
```console
foo@bar:~/VELDT-getting-started/veldt$ touch Veldt/Serial.hs
```
Now expose the module with `veldt.cabal`. Your `exposed-modules` section should look similar.
```
.....
exposed-modules: Veldt.Counter,
		 Veldt.PWM,
		 Veldt.Serial,
		 Veldt.Ice40.Rgb
.....
```
Let's begin editing the `Serial.hs` file. Fundamentally, we represent serializers and deserializers with a counter and a `Vec` from [Clash.Sized.Vector](http://hackage.haskell.org/package/clash-prelude-1.2.4/docs/Clash-Sized-Vector.html). This means we will be able to serialize or deserialize in two directions say left or right e.g. for a deserializer we could add elements at the beginning (left) or end (right) of the `Vec`. Additionally, we use a flag to indicate whether a deserializer is full or a serializer is empty.
```haskell
module Veldt.Serial
  ( Direction(..)
  -- Deserializer                                                                             
  , Deserializer
  , mkDeserializer
  , isFull
  , deserialize
  , get
  , clear
  -- Serializer
  , Serializer
  , mkSerializer
  , isEmpty
  , serialize	
  , peek
  , give
  ) where

import Clash.Prelude
import Control.Monad.RWS (RWST)
import Control.Lens hiding (Index)
import qualified Veldt.Counter as C
```
With a deserializer we are able to:
  1. construct it with `mkDeserializer`
  2. check if it is full
  3. `deserialize` data, adding it to either the front or back of the vector depending on the direction and incrementing the counter.
  4. `get` the `Vec` of elements of the deserializer
  5. `clear` the full flag and reset the counter

Similarly with a serializer we are able to:
  1. construct it with `mkSerializer`
  2. check if it is empty
  3. `serialize` data, shifting either left or right depending on the direction and incrementing the counter
  4. `peek` at the element to serialize
  5. `give` new data to the serializer and reset the counter
### [UART My Art](https://github.com/standardsemiconductor/VELDT-getting-started#table-of-contents)
### [Roar: Echo](https://github.com/standardsemiconductor/VELDT-getting-started#table-of-contents)
## [Section 4: Pride](https://github.com/standardsemiconductor/VELDT-getting-started#table-of-contents)
## [Section 5: Where Lions Roam](https://github.com/standardsemiconductor/VELDT-getting-started#table-of-contents)
