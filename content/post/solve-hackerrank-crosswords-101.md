---
title: Case Study - Solve Crosswords Puzzle
date: 2018-07-08T20:47:51+04:30
draft: false
tags: [Scala,hackerrank]
---
As a software engineer, I always enjoy solving problems and that's why
I sometimes go to the coding challenge websites like [hackerrank](https://www.hackerrank.com/) and
pick one problem to solve. This time I searched in **functional programming** category and
[crosswords-101](https://www.hackerrank.com/challenges/crosswords-101/problem) caught my eye and I decided to solve
it in Scala in a very clean and understandable way and not in a contest style way of programming!

This article is going to reveal how I solved this puzzle. But before continue, please read the problem definition
and try to solve it by yourself or at least think about the way of solving it and then continue reading.

## Model
Whenever you write software the good model is crucial part for your application. If your model is good
then your program is very clean, easy to understand and also maintainable. That's why I started the solution with
defining modeles. The first model that I wrote was [Placeholder](https://github.com/mostafa-asg/crosswords/blob/master/src/main/scala/com/github/Placeholder.scala).

### Placeholder
Placeholder is a place for inserting words. For instance below you can find two placeholders:
{{< fluid_imgs
        "center|/static/crosswords/1.png|Placeholders"
>}}
And of course you can model it by **case classes**:
```
case class Placeholder(size: Int, direction: Direction)
```
For placeholders that have assigned a word, there is a **currentValue**
property that you can set. It accepts **Option[String]**:
{{< fluid_imgs
        "center|/static/crosswords/2.png|Placeholders"
>}}
And because there may be more than words with 7 characters or 3 characters there is another
property called **candidateValues** that accepts only words which match placeholder's size and
you can generate all possibilities by calling **allPossibilities**:
```
val ph = Placeholder(size = 7,
                     direction = Horizontal,
                     candidateValue = Set("England", "IRELAND"))
val all: List[Placeholder] = ph.allPossibilities
```
*all* contains these placeholders:
{{< fluid_imgs
        "center|/static/crosswords/3.png|Placeholders"
>}}

### Crosswords
No one likes a single placeholder only. That's why we need another model for a
collection of placeholders. We call this model **Crosswords**. Crosswords is a
collection of placeholders with additional property like number of rows and columns:
```
case class Crosswords(rows: Int, cols: Int, placeholders: List[Placeholder]) {
    // code removed for brevity
}
```
For instance for defining this crosswords that has 2 placeholders:
{{< fluid_imgs
        "center|/static/crosswords/4.png|Placeholders"
>}}

We can declare:
```
val cw = Crosswords(rows = 5, cols = 9, placeholders = List(
    Placeholder(size = 7, direction = Horizontal, startPosition = Point(1,1)),
    Placeholder(size = 3, direction = Vertical), startPosition = Point(1,1))
)
```
Notice about **startPosition**. It specifies the position of placeholder in the crosswords. It will
be used by rendering the crosswords later. But what about the intersection? As you can see there is intersection between these two
placeholders at first character. How can we define these intersections? We need a way to
select two placeholder somehow and tell that at which index there is an intersection with other
placeholder's index. So we need an **id** for each placeholder for refereeing it later:
```
val cw = Crosswords(rows = 5, cols = 9, placeholders = List(
    Placeholder(id = "ph1", size = 7, direction = Horizontal, ...),
    Placeholder(id = "ph2", size = 3, direction = Vertical, ...),
))
```
now we can define intersections:
```
cw.addIntersection(from = ("ph1",0), to = ("ph2",0))
// order does not matter
// you can add ph2 first!
// for instance:
// cw.addIntersection(from = ("ph2",0), to = ("ph1",0))
```
Now that we have all the building blocks for creating a crosswords, how we can solve it?

### How to Solve
Imagine we have two placeholders without any intersection in the crosswords. The picture below
show them with their candidate values. How many possible valid solution exists?
{{< fluid_imgs
        "center|/static/crosswords/5.png|Placeholders"
>}}
Yes, there are **2 * 4 = 8** valid solution exist. To get all valid solution you can call
[solveAllPossibilities](https://github.com/mostafa-asg/crosswords/blob/0946a6898e8de8b5b22f4e89ae0bd933a5470922/src/main/scala/com/github/Crosswords.scala#L43) that
returns all the valid solution:
```
val solutions: List[Crosswords] = cw.solveAllPossibilities
// assert(solutions.size == 8)
```
To get each crosswords output representation, you can call [stringRepresentation](https://github.com/mostafa-asg/crosswords/blob/28912d6abe78b94de6787723186e6cfc33a74c88/src/main/scala/com/github/Crosswords.scala#L87),
so to render all the solutions to the standard output, you can write:
```
solutions.foreach { crosswords =>
    println(crosswords.stringRepresentation)
    println("------------------------------"
}
```
### With Intersection
But what about crosswords with intersection? Imaging we have a crosswords with 2 placeholders: **ph1** and **ph2** and
there exists an intersection:
```
cw.addIntersection(from = ("ph1",2), to = ("ph2",1))
```
The picture below depicts the idea:
{{< fluid_imgs
        "center|/static/crosswords/6.png|Placeholders"
>}}
How many valid solution exists? This time 3 valid solution exist not 8 because there are some combination
that violate the intersection. For example combination of **TRACK** and **MOON** is not valid since **A != O**.
In code, [isValidSolution](https://github.com/mostafa-asg/crosswords/blob/28912d6abe78b94de6787723186e6cfc33a74c88/src/main/scala/com/github/Crosswords.scala#L69)
is responsible to check that solution is valid.

## Build crosswords from string
Now that we have our models, how we can build crosswords from input string? [CrosswordsBuilder.buildFrom](https://github.com/mostafa-asg/crosswords/blob/28912d6abe78b94de6787723186e6cfc33a74c88/src/main/scala/com/github/CrosswordsBuilder.scala#L14)
comes to rescue.if we have a string like **+- - - - -++++**, we know that there exists a horizontal placeholder with size 5.
[findAllPlaceholders](https://github.com/mostafa-asg/crosswords/blob/28912d6abe78b94de6787723186e6cfc33a74c88/src/main/scala/com/github/CrosswordsBuilder.scala#L90) is a method
that accepts a string and find all placeholders. But what about columns like:
```
+
+
-
-
-
+
```
If we can rotate it, then it becomes something like **++- - - +** and now we
can pass it to **findAllPlaceholders** method, but this time with paramter **Vertical**. That's why 
there is a method for swaping rows with columns called [swapRowsWithColumns](https://github.com/mostafa-asg/crosswords/blob/28912d6abe78b94de6787723186e6cfc33a74c88/src/main/scala/com/github/CrosswordsBuilder.scala#L21).

So far we have found all horizontal and vertical placeholders but we have no idea
about intersections between them. Since intersection can only happens between
horizontal and vertical placeholders, I 've defined a method called [findIntersectionsBetween](https://github.com/mostafa-asg/crosswords/blob/28912d6abe78b94de6787723186e6cfc33a74c88/src/main/scala/com/github/CrosswordsBuilder.scala#L64)
that accepts a list of vertical placeholders, a list of horizontal placeholders
and returns all intersection between them.

### Find intersections
Given two placeholder, how can you find intersection between them? For instance:
```
val ph1 = Placeholder(size = 5, direction = Horizontal, startPosition = (2,1))
val ph2 = Placeholder(size = 4, direction = Vertical, startPosition = (1,3))
```
The clue is **startPosition**. Each placeholder has [getPositionPoints](https://github.com/mostafa-asg/crosswords/blob/28912d6abe78b94de6787723186e6cfc33a74c88/src/main/scala/com/github/Placeholder.scala#L51)
that returns all the cell position that this placeholder has occupied:
```
ph1.getPositionPoints
// returns List(Point(2,1), Point(2,2), Point(2,3), Point(2,4), Point(2,5))
ph2.getPositionPoints
// returns List(Point(1,3), Point(2,3), Point(3,3), Point(4,3))
```
And if
```
(ph1.getPositionPoints intersect ph2.getPositionPoints).size > 0
```
We are sure that there is an intersection. But we are not done yet.
After finding an intersection, we should call **Crosswords.addIntersection** and this method does not
accept **(row,col)** index, instead it accepts placeholder's local index. The picture below depicts the idea:
{{< fluid_imgs
        "center|/static/crosswords/7.png|Placeholders"
>}}
We need a function that accept crosswords's index in the form of **(row,col)** and
returns placeholder's local index. This function is called [pointToIndex](https://github.com/mostafa-asg/crosswords/blob/28912d6abe78b94de6787723186e6cfc33a74c88/src/main/scala/com/github/Placeholder.scala#L102):
{{< fluid_imgs
        "center|/static/crosswords/8.png|Placeholders"
>}}

## Done!
That was the most important aspects of the code. Of course there are some details that I didn't
mention in this article. You can find the full source code on [my github repository](https://github.com/mostafa-asg/crosswords/).
