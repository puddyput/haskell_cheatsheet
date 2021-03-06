# Haskell Cheatsheet
*v1.a, 24.06.2018, [GitHub](https://github.com/puddyput/haskell_cheatshee)*

In preparation of my exam I assembled this file with notes. It is not exhaustive, some of it is barely explained but I'm almost certain nothing is wrong. It begins with some **basics** after which there is a list of **technical terms**, a glossary of sorts. The **topics** cover functors, applicative functors, monads, parsing, monoids, foldables, quickcheck and GADTs. In the end there are some code **examples** and **references** for further reading.
I tried to cite my sources when I copied something but may have missed some :/

### Basics
This section is a reference for basics. The more complex, theoretical topics come after the technical terms.
##### Paranthesis
`$` is to get rid of paranthesis: `putStrLn (show (1 + 1))` == `putStrLn (show $ 1 + 1)`
`.` chains functions: `putStrLn (show (1 + 1))` == `(putStrLn . show) (1 + 1)`
Input type of putStrLn equals output type of show (`String`)
##### Guards
```haskell
sign :: Integer -> Integer
sign z  | z < 0       = -1
        | z > 0       = 1
        | otherwise   = 0
```
##### Anonymous function
`sign' :: Integer -> Integer`
`sign' = \z -> vorzeichen z`

##### Quicksort
```
qsort :: Ord a => [a] -> [a]
qsort []     = []
qsort (x:xs) = qsort (smaller xs) ++ [x] ++ qsort (greater xs)
          where
            smaller xs = [ z | z <- xs, z <= x ]
            greater xs = [ z | z <- xs, z > x]
```
##### Declarations
###### Type
```haskell
data Tree a = Leaf  a
            | Node a (Tree a) (Tree a)
```
###### Instance
```
> instance Eq a => Eq ( Tree a ) where
Leaf x     == Leaf y        = x == y
Node x l r == Node x' l' r' = x == x' && l == l' && r == r'
_          == _             = False
```
##### List Comprehension
`pairs xs ys = [ (x,y) | x <- xs, y <- ys ]`


##### Technical Terms
 - "Ausdrucksstarkes Typsystem"
   "a correctly typed program will not produce errors during runtime"
 - Type                                 
   Describes Terms
 - Kind
   Describe Types. Used for GADTs
 - Constructor
   can be used both for pattern matches to deconstruct values and as functions to construct values
 - Class
   Bunch of functions that are defined using the same name on all types of this class
   Eq (`==`, `/=`), Ord (`<`, `>`, `>=`, ..), Num (`+`, `-`, ..)
 - Instance
   When adding a type to a class, those methods must be implemented
   `instance Tree [] where ..`
 - Parametric Polymorphism
   Polymorph functions can be applied to any data if the data's type can be generalized to the function's type. 
   A List doesn't need to know it's content to be able to tell it's length. In `length :: [a] -> Int` *a* is polymorph
 - Ad-Hoc Polymorphism
   A function with a polymorph parameter BUT the actual type of the parameter decides the implementation of the function (+ for addition or + for concatting strings). Might need it if you want Eq (==) for a type
 - Inheritance
   `class Eq a => Ord a where`.. *Ord* extends *Eq*
 - Higher order function
    Functions as parameters
 - Pure
   Functions without side effects
 - Category
   Categories are things that behave like functions, i.e. you can put one after another with a generalized version of (.)
   generalizes `(.)`
 - Monoid
   Monoid is a class of associative binary(2 params) operations with an identity (neutral element).
   generalizes `(++)`
 - Functor
   Translate "foreign" function for use in this "category" (e.g. container)
   generalizes `map`
 - Applicative Functor
   generalizes `zip` or `zipWith`
 - Monad
   generalizes `concat`

 - Infix
   prefix the infix operator *+* `(+) 2 3`
   infix the prefix function *concatPrint* ``"a" `concatPrint` "b"``
 - Currying
   `curry fst 1 2` (two parameters instead of tuple as parameter)
   uncurry converts a curried function to a function on pairs.
   `uncurry (+) (1,2)` (one tuple instead of two parameter)

## Topics

#### Functor
Translate "foreign" function for use in this "category" (e.g. container)
These laws have to be true:
```haskell
fmap id = id
fmap (g . h) = fmap g . fmap h

id :: a -> a
id x = x 
```

Define functor instances:
```haskell
instance Functor Tree where
  fmap' = maptree

maptree :: (a -> b) -> Tree a -> Tree b
maptree _ Leaf         = Leaf
maptree f (Node l n r) = Node (maptree f l)
                               (f n)
                               (maptree f r)
instance Functor [] where
  fmap' = map

instance Functor Maybe where
  fmap' _ Nothing  = Nothing
  fmap' g (Just x) = Just (g x)

```
#### Applicative Functor
> This module describes a structure intermediate between a functor and a monad: it provides pure expressions and sequencing, but no binding. (Technically, a strong lax monoidal functor.) [C. A. McCann](https://stackoverflow.com/questions/3242361/haskell-how-is-pronounced)

Functors map over general structures one at the time, but with an Applicative functor you can zip together two or more structures.
`pure (+) <*> Just 1 <*> Just 2` gives Just 3
`pure (+) <*> Nothing <*> Just 2` gives Nothing`. The structure affects the result

This is what we do without them:
```haskell
listadd :: [Int] -> [Int] -> [Int]
listadd xs ys = [ x + y | x <- xs, y <- ys ]
```
This is what we want: `fmap2 (+) xs ys` / `zipWith (+) [1] [2]`
But with any number of arguments!
Spaceship operator to the rescue:
```haskell
pure :: a -> f a
(<*>) :: f (a -> b) -> f a -> f b
```
How do you pronounce `<*>` ? [Stack Overflow](https://stackoverflow.com/questions/3242361/haskell-how-is-pronounced) says "apply" or "zip functors with" or not at all:
> In fact, I encourage liberal use, and non-pronunciation, of all lifted application operators:
>   `(<$>)`, which lifts a single-argument function into a Functor
>   `(<*>)`, which chains a multi-argument function through an Applicative
>   `(=<<)`, which binds a function that enters a Monad onto an existing computation
> All three are, at heart, just regular function application, spiced up a little bit.

Back to the `listadd` we want:
```haskell
instance Applicative [] where
  -- pure :: a -> [a]
  pure x    = [x]
  -- (<*>) :: [a -> b] -> [a] -> [b]
  gs <*> xs = [ g x | g <- gs, x <- xs ]

listadd' :: [Int] -> [Int] -> [Int]
listadd' xs ys = pure (+) <*> xs <*> ys 
```
With `<*>` we can go as long as we want! No need for "fmap, fmap2, fmap3, .."
#### Monad
*This entire section is pretty much stolen from [Prof. Weirich ](http://www.seas.upenn.edu/~cis552/13fa/lectures/stub/Monads.html)*
By default, Haskell functions are pure. Sometimes we need impure functions, like IO.
Monads integrate "impure" programming into Haskell, but they are *much* more general than that.

Monads generalize the function `concat :: [[a]] -> [a]` to work with many other sorts of structures besides lists. As a simple example, the monadic operation join can be used to flatten nested Maybe values:
`join (Just (Just 42))` -- gives Just 42
`join (Just (Nothing))` -- gives Nothing

Consider this function, it's zipTree but with Maybe so we can catch errors.
```
zipTree :: Tree a -> Tree b -> Maybe (Tree (a,b))
zipTree (Leaf a) (Leaf b) = 
    Just (Leaf (a,b))   <-------------------- this is how we    
zipTree (Branch l r) (Branch l' r') =          return a value
   ~~~~~~~~~~~~~~~~~~~~~~                              |
   case zipTree l l' of                                |
     Nothing -> Nothing                                |
     Just l'' ->           <------- This is how we     |
   ~~~~~~~~~~~~~~~~~~~~             use a value        |
       case zipTree r r' of         |                  |
         Nothing -> Nothing   <-----|                  |
         Just r'' ->                                   |
        ~~~~~~~~~~~~~~~~~~~~~~~                        |
            Just (Branch l'' r'')   <------------------|
zipTree _ _ = Nothing
```
Some patterns repeat. 
Returning a value:
```
  Just (Leaf(a,b))
  Just x 
```
Let's make a pattern out of it and name it "return":
```
return :: a -> Maybe a                 
return x = Just x
```

For using a value we have:
```
case zipTree l l' of                 
  Nothing -> Nothing
  Just l'' -> ... 
    do something with l''

case zipTree r r' of                 
  Nothing -> Nothing
  Just r'' -> ... 
    do something with r''
```
General pattern:
```
case x of 
  Nothing -> Nothing
  Just y -> f y
```
Name that pattern ("bind") or ("use x in f"):
```
(>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b
x >>= f = 
  case x of 
    Nothing -> Nothing
    Just y  -> f y
```
If the first argument is `Nothing` then the second argument is ignored and `Nothing` is returned as the result. Otherwise, if the first argument is of the form `Just x`, then the second argument is applied to `x` to give a result of type `Maybe b`.

Now we can refactor the code from the beginning!
```
zipTree :: Tree a -> Tree b -> Maybe (Tree (a,b))
zipTree (Leaf a) (Leaf b)           = return (Leaf (a,b))
zipTree (Branch l r) (Branch l' r') = do
   l'' <- (zipTree l l')
   r'' <- (zipTree r r') 
   return (Branch l'' r'')
zipTree _ _ = Nothing
```
List as monad:
```
instance Monad [] where
  -- return x = [x]
  -- bind :: [a] -> (a -> [b]) -> [b]
  bind xs f = [ y | x <- xs, y <- f x ] 
  
instance Functor [] where
  fmap' = map

instance Applicative [] where
  -- pure :: a -> [a]
  pure x    = [x]
  -- (<*>) :: [a -> b] -> [a] -> [b]
  gs <*> xs = [ g x | g <- gs, x <- xs ]

```

#### Parser
`' ' <$ char '+'` = replace + with space
`<|>` =  If the left succeeds, its result becomes the result of `<|>`. If it fails, the result becomes the result of the right parser

1. newtype 
`newtype Parser a = P (String -> [ (a, String) ] )`
2. parse

`parse :: Parser a -> String ->  [ (a, String) ]`
`parse (P p) input = p input`
3. parse a single char
```haskell
> item :: Parser Char
> item = P (\input -> case input of
>                       []     -> []
>                       (x:xs) -> [ (x, xs) ])
```
4. applicative functor for chaining with `<*>`
```haskell
instance Functor Parser where
  -- fmap :: (a -> b) -> Parser a -> Parser b
  fmap f p = P (\input -> case parse p input of
                            []        -> []
                            [(x, xs)] -> [(f x, xs)])
                            
instance Applicative Parser where
  -- pure  :: a -> Parser a
  pure x = P (\input -> [(x, input)])
  -- (<*>) :: Parser (a -> b) -> Parser a -> Parser b
  pf <*> px = P (\input -> case parse pf input of
                           []            -> []
                           [(f, output)] -> parse (fmap f px) output)
```
5. now we can do this:
```haskell
take1and3 :: Parser (Char, Char)
take1and3 = pure f <*> item <*> item <*> item
  where
    x y z = (x, z)
```
6. let's go monad
```haskell
instance Monad Parser where
  -- (>>=) :: Parser a -> (a -> Parser b) -> Parser b
  p >>= f = P (\input -> case parse p input of
                        []            -> []
                        [(x, output)] -> parse (f x) output)
```
7. so we can do this:
```haskell
mTake1and3 :: Parser (Char, Char)
mTake1and3 = do
  x <-  item
        item 
  z <-  item
  return (x, z)
```
Useful functions for parsing:
`sat :: (Char -> Bool) -> Parser Char`
`many` Parses zero or more occurrences of the given parser.
```haskell
int :: Parser Int
int = do
  char '-'
  m <- nat
  return (-m)
 <|>
  nat
```
Parse C Style line comment
```haskell
linecomment :: Parser ()
linecomment = do string "//"
              many (sat (/= '\n'))
              char '\n'
              return ()
```
##### Monoid
Monoid is a class of associative binary operations with an identity. 
Instances should satisfy the following laws:
- identity element exists
    -   `x <> mempty = x`
    - `mempty <> x = x`
- associativity `x <> (y <> z) = (x <> y) <> z ` 
note: `(<>) = mappend`

Monoids: 
- set of integers, the element 0, and `(+)`. 
- set of positive integers, the element 1, and `(*)`
- set of all lists, the empty list [], and `(++)`
- Flip of a pair

Monoids are used to 'combine' and accumulate things. Like reduce a list of sums to single sum.
[read more](https://www.schoolofhaskell.com/user/mgsloan/monoids-tour)
##### Foldable
Class of data structures that can be folded to a summary value.
A Foldable must support `foldr` ->  it has a list representation: `toList = foldr (:) []`
```haskell
instance Foldable Tree where
  -- fold :: Monoid a => Tree a -> a
  fold (Leaf x) = x
  fold (Node l r) = fold l `mappend` fold r
  -- foldMap :: Monoid b => (a -> b) -> Tree a -> b
  foldMap f (Leaf x)   = f x
  foldMap f (Node l r) = foldMap f l `mappend` foldMap f r
  -- foldr (a -> b -> b) -> b -> Tree a -> b
  foldr f v (Leaf x)   = f x v
  foldr f v (Node l r) = foldr f (foldr f v r) l
  -- foldl (a -> b -> a) -> a -> Tree b -> a
  foldl f v (Leaf x)   = f v x
  foldl f v (Node l r) = foldl f (foldl f v l) r
```
#### QuickCheck
> The key idea on which QuickCheck is founded is property-based testing. That is, instead of writing individual test cases (eg unit tests corresponding to input-output pairs for particular functions) one should write properties that are desired of the functions, and then automatically generate random tests which can be run to verify (or rather, falsify) the property. -- [Weirich](https://www.seas.upenn.edu/~cis552/13fa/lectures/QuickCheck1.html)

Useful functions for building Generators:
- **`choose`** takes an interval and returns an random element from that interval:
   `choose :: (System.Random.Random a) => (a, a) -> Gen a`
   `*Main> sample $ choose (0, 3)`
- **`elements`** returns a generator that produces values drawn from the input list
   `elements :: [a] -> Gen a`
   `*Main> sample $ elements [10, 20..100]`
- **`oneof`** allows us to randomly choose between multiple generators
   `oneof :: [Gen a] -> Gen a`
   `*Main> sample $ oneof [elements [10,20,30], choose (0,3)]`
- **`frequency`** allows us to build weighted combinations of individual generators
   `frequency :: [(Int, Gen a)] -> Gen a`

Generator for lists
```haskell
genList ::  (Arbitrary a) => Gen [a]
genList = frequency [ (1, return [])                        -- 1:7 chance to terminate
                     , (7, liftM2 (:) arbitrary genList) ]  -- add one arbitrary
                     
genOrdList :: (Arbitrary a, Ord a) => Gen [a]
genOrdList = genList >>= return . sort             -- guarantee ordered list by sorting afterwards

*Main> sample (genOrdList :: Gen [Int])
```                     
#### RedBlackTree
```haskell
data Color = Red | Black deriving (Eq, Show)
data RBT a = Blatt | Knoten Color (RBT a) a (RBT a) deriving (Eq, Show)
```
meh, let's just hope the test won't cover it.
#### GADTS
Generalized Algebraic Data Types allows us to restrict the return types of constructors and thus enable us to take advantage of Haskell's type system for our domain specific languages.
This section is taken from [wikibooks](https://en.wikibooks.org/wiki/Haskell/GADT)
```haskell
data Expr = I Int
          | B Bool           -- boolean constants, for equality test
          | Add Expr Expr
          | Mul Expr Expr
          | Eq  Expr Expr    -- equality test

eval :: Expr -> Maybe (Either Int Bool)
-- implementation here
```
But having to use Maybe is no fun. We want the typechecker to check the types for us. Let's add a phantom type a:
```haskell
data Expr a = I Int
            | B Bool
            | Add (Expr a) (Expr a)
            | Mul (Expr a) (Expr a)
            | Eq  (Expr a) (Expr a)
-- now we can restrict the parameters of add with a smart constructor:
add :: Expr Int -> Expr Int -> Expr Int
add = Add
-- and the other constructors:
i :: Int  -> Expr Int
i = I
b :: Bool -> Expr Bool
b = B
```  
Note that an expression Expr a does not contain a value a at all; that's why a is called a phantom type, it's just a dummy variable. Compare that with, say, a list [a] which does contain a bunch of as. The key idea is that we're going to use a to track the type of the expression for us. Now the type checker forbids ``b True `add` i 5``!
By only exporting the smart constructors, the user cannot create expressions with incorrect types.
But we still cannot declare eval as `eval :: Expr a -> a` and implement `eval (I n) = n`. How would the compiler know that encountering the constructor `I` means that `a = Int`?
The obvious notation for restricting the type of a constructor is to write down its type, and that's exactly how GADTs are defined:
```haskell
data Expr a where
    I   :: Int  -> Expr Int
    B   :: Bool -> Expr Bool
    Add :: Expr Int -> Expr Int -> Expr Int
    Mul :: Expr Int -> Expr Int -> Expr Int
    Eq  :: Eq a => Expr a -> Expr a -> Expr Bool
```
Thanks to GADTs we now can implement an evaluation function that takes advantage of the type marker:
```haskell
eval :: Expr a -> a
eval (I n) = n
eval (B b) = b
eval (Add e1 e2) = eval e1 + eval e2
eval (Mul e1 e2) = eval e1 * eval e2
eval (Eq  e1 e2) = eval e1 == eval e2
```
In the case `eval (I n) = n` the compiler is now able to infer that `a=Int` when we encounter a constructor `I` and that it is legal to return the `n :: Int`

```haskell
data Empty
data NonEmpty
data SafeList a b where
  Nil  :: SafeList a Empty                          -- return SafeList a Empty on Nil constructor
  Cons :: a -> SafeList a b -> SafeList a NonEmpty  -- return SafeList a NonEmpty on Cons constructor
-- Constructor returning something else but just "SafeList a" not possible in Haskell98 

safeHead :: SafeList a NonEmpty -> a
safeHead (Cons x _) = x
```

### Examples
This section contains some examples that might be useful to know.
##### Insertion Sort
```haskell
insert x []                 = [x]
insert x (y:ys) | x > y     = x : y : ys
                | otherwise = y : insert x ys
                
isort :: Ord a => [a] -> [a]
isort = foldr insert []           
```
##### Identity
There is only one function with type `id :: a -> a`: `id x = x`
Even more generic `f :: a -> b` -- sole implementation: `f x = f x`
### Ressources
###### references
https://hackage.haskell.org/package/base-4.11.1.0/docs/Prelude.html
[Category, Monoid, (Applicative) Functor, Monad](https://stackoverflow.com/questions/15726733/simple-examples-to-illustrate-category-monoid-and-monad)

###### courses
https://www.seas.upenn.edu/~cis552/13fa/schedule.html
https://www.schoolofhaskell.com
[Best introduction into Monads](http://blog.sigfpe.com/2006/08/you-could-have-invented-monads-and.html)
###### practice
https://vowi.fsinf.at/wiki/TU_Wien:Funktionale_Programmierung_VU_(Knoop)/Pr%C3%BCfung_2018-03-02

