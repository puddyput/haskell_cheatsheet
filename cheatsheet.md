#### Basics
##### Guards
```
sign :: Integer -> Integer
sign z  | z < 0       = -1
        | z > 0       = 1
        | otherwise   = 0
```
##### Anonymous function
```
sign' :: Integer -> Integer
sign' = \z -> vorzeichen z
```
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
```
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


##### Fachbegriffe
 - Ausdrucksstarkes Typsystem
 - Parametrische Polymorphie
 - Ad-Hoc Polymorphie
 - higher order function
    Functions as parameters
 - Pure
   Functions without side effects
 - Functor
   Translate "foreign" function for use in this "category" (e.g. container)
 - Applicative Functor
 - Monad
 - 
#### Monad

#### Functor
Translate "foreign" function for use in this "category" (e.g. container)
These laws have to be true:
```
fmap id = id
fmap (g . h) = fmap g . fmap h

id :: a -> a
id x = x 
```

Define functor instances:
```
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

This is what we do without them:
```
listadd :: [Int] -> [Int] -> [Int]
listadd xs ys = [ x + y | x <- xs, y <- ys ]
```
This is what we want: `fmap2 (+) xs ys`
But with any number of arguments!
Spaceship operator to the rescue:
```
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
```
instance Applicative [] where
  -- pure :: a -> [a]
  pure x    = [x]
  -- (<*>) :: [a -> b] -> [a] -> [b]
  gs <*> xs = [ g x | g <- gs, x <- xs ]

listadd' :: [Int] -> [Int] -> [Int]
listadd' xs ys = pure (+) <*> xs <*> ys 
```
With `<*>` we can go as long as we want! No need for "fmap, fmap2, fmap3, .."
#### Parser

#### Foldable

#### QuickCheck

#### RedBlackTree

#### GADTS

