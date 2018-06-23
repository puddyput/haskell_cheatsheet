#### Basics
##### Paranthesis
$ and .
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


##### Terms
 - "Ausdrucksstarkes Typsystem"
 - Parametric Polymorphism
 - Ad-Hoc Polymorphism
 - higher order function
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
 - currying
   `curry fst 1 2` (two parameters instead of tuple as parameter)
   uncurry converts a curried function to a function on pairs.
   `uncurry (+) (1,2)` (one tuple instead of two parameter)

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
> This module describes a structure intermediate between a functor and a monad: it provides pure expressions and sequencing, but no binding. (Technically, a strong lax monoidal functor.)

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
#### Foldable
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
#### QuickCheck

#### RedBlackTree

#### GADTS


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

### Footnotes
[^cis552monads]: http://www.seas.upenn.edu/~cis552/13fa/lectures/stub/Monads.html
