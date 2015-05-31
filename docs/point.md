# Custom predicates

The library comes with a lot [predefined predicates][provided-predicates]
but also allows to define your own. This example shows how to add predicates
for a simple type representing a point in a two-dimensional Cartesian
coordinate system. We start by defining a `Point` class that represents a
point in our coordinate system:

```scala
scala> case class Point(x: Int, y: Int)
defined class Point
```

The axes of a two-dimensional Cartesian coordinate system divide the plane into
four infinite regions, called quadrants, which are often numbered 1st to 4th.
Suppose we want to refine `Point`s with the quadrant they are lying in.
So let's create simple types that represent the four quadrants:

```scala
scala> trait Quadrant1
defined trait Quadrant1

scala> trait Quadrant2
defined trait Quadrant2

scala> trait Quadrant3
defined trait Quadrant3

scala> trait Quadrant4
defined trait Quadrant4
```

We now have type-level predicates and a type that we want to refine with these
predicates. The next step is to define instances of the `Predicate` type class
for `Point` that are indexed by the corresponding quadrant predicate. We use
the `Predicate.instance` function to create the instances from two functions,
one that checks if a given `Point` lies in the corresponding quadrant and one
that provides a string representation for the predicate that is used for error
messages:

```scala
scala> import eu.timepit.refined.Predicate
import eu.timepit.refined.Predicate

scala> implicit val quadrant1Predicate: Predicate[Quadrant1, Point] =
     |   Predicate.instance(p => p.x >= 0 && p.y >= 0, p => s"($p is in quadrant 1)")
quadrant1Predicate: eu.timepit.refined.Predicate[Quadrant1,Point] = eu.timepit.refined.Predicate$$anon$2@5136071c

scala> implicit val quadrant2Predicate: Predicate[Quadrant2, Point] =
     |   Predicate.instance(p => p.x < 0 && p.y >= 0, p => s"($p is in quadrant 2)")
quadrant2Predicate: eu.timepit.refined.Predicate[Quadrant2,Point] = eu.timepit.refined.Predicate$$anon$2@71a0e244

scala> implicit val quadrant3Predicate: Predicate[Quadrant3, Point] =
     |   Predicate.instance(p => p.x < 0 && p.y < 0, p => s"($p is in quadrant 3)")
quadrant3Predicate: eu.timepit.refined.Predicate[Quadrant3,Point] = eu.timepit.refined.Predicate$$anon$2@3c35f8c9

scala> implicit val quadrant4Predicate: Predicate[Quadrant4, Point] =
     |   Predicate.instance(p => p.x >= 0 && p.y < 0, p => s"($p is in quadrant 4)")
quadrant4Predicate: eu.timepit.refined.Predicate[Quadrant4,Point] = eu.timepit.refined.Predicate$$anon$2@660c0c15
```

We have now everything in place to refine our values:

```scala
scala> import eu.timepit.refined.refine
import eu.timepit.refined.refine

scala> refine[Quadrant1](Point(1, 3))
res0: Either[String,shapeless.tag.@@[Point,Quadrant1]] = Right(Point(1,3))

scala> refine[Quadrant1](Point(3, -2))
res1: Either[String,shapeless.tag.@@[Point,Quadrant1]] = Left(Predicate failed: (Point(3,-2) is in quadrant 1).)

scala> refine[Quadrant4](Point(3, -2))
res2: Either[String,shapeless.tag.@@[Point,Quadrant4]] = Right(Point(3,-2))
```

```scala
scala> import eu.timepit.refined.boolean.Or
import eu.timepit.refined.boolean.Or

scala> type Quadrant1Or3 = Quadrant1 Or Quadrant3
defined type alias Quadrant1Or3

scala> refine[Quadrant1Or3](Point(1, 3))
res3: Either[String,shapeless.tag.@@[Point,Quadrant1Or3]] = Right(Point(1,3))

scala> refine[Quadrant1Or3](Point(-3, -2))
res4: Either[String,shapeless.tag.@@[Point,Quadrant1Or3]] = Right(Point(-3,-2))

scala> refine[Quadrant1Or3](Point(3, -2))
res5: Either[String,shapeless.tag.@@[Point,Quadrant1Or3]] = Left(Both predicates of ((Point(3,-2) is in quadrant 1) || (Point(3,-2) is in quadrant 3)) failed. Left: Predicate failed: (Point(3,-2) is in quadrant 1). Right: Predicate failed: (Point(3,-2) is in quadrant 3).)
```

[provided-predicates]: https://github.com/fthomas/refined#provided-predicates