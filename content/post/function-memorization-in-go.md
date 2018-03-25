---
title: "Function Memorization in Go"
date: 2018-03-23T22:20:14+04:30
draft: false
tags: [Go,Functional Programming,Performance]
---
[Memoization](http://en.wikipedia.org/wiki/Memoization) is an optimization technique used to increase performance by storing the results of expensive function calls and returning the cached result when the same input occurs again. In this post I show how function memoization can be implemented in Go, in a pure functional manner.

Based on [wikipedia definition](https://en.wikipedia.org/wiki/Pure_function), a function may be considered a pure function if both of the following statements about the function hold:  
**1.** The function always evaluates the same result value given the same argument value(s). The function result value cannot depend on any hidden information or state that may change while program execution proceeds or between different executions of the program, nor can it depend on any external input from I/O devices (usually—see below).  
**2.** Evaluation of the result does not cause any semantically observable side effect or output, such as mutation of mutable objects or output to I/O devices (usually—see below).

Pure functions are more easily optimized. Let's perform optimizations using a caching technique. Assume we have a factorial function like this:
```
func fact(n int) int {
	result := 1
	for i:=2;i<=n;i++ {
		result *= n
	}
	return result
}
```
One way to speed things up is to cache result values and to look them up on subsequent invocations of **fact** function.This can be done either by the function itself or by the caller. The first approach has the drawback that each and every function has to implement caching and that clients have no control over the caching mechanism. The latter approach puts all the burden on the client programmer and produces potential boilerplate. To overcome these issues we need a way to construct a memorized function having the same type as a given function. Let's define **memorized** function that turns any function which accepts **int** as input and returns **int** as output to a memorized capable function:

```
type FuncIntInt func(int) int

func memorized(fn FuncIntInt) FuncIntInt {
	cache := make(map[int]int)

	return func(input int) int {
		if val, found := cache[input]; found {
			log.Println("Read from cache")
			return val
		}

		result := fn(input)
		cache[input] = result
		return result
	}
}
```
It returns a simple function that checks input value against cache first. If it can find it in cache, it returns the value otherwise it calls the input function and store the result into the cache for subsequent calls.
Now lets convert our factorial function into a memorized one by calling:
```
factMem := memorized(fact)
println( factMem(5) )
println( factMem(5) )
println( factMem(5) )
```
Second and third call of factMem would give us the result from the cache. But wait a minute. Does this technique work well for
recursive factorial function as well? I mean this function:
```
func fact(n int) int {
    if n <= 1 {
        return 1
    }
    return n * fact(n-1)
}
```
If you call this function like this:
```
factMem := memorized(fact)
println( factMem(5) )
println( factMem(4) )
println( factMem(3) )
```
And if you expect **factMem(4)** and **factMem(3)** gets their result from cache you are wrong, because n * **fact(n-1)** is not a memorized function.
You can tackle this problem by passing anonymous function of factorial to **memorized** function and calling memorized version in the recursive call:
```
var factMem FuncIntInt
factMem = memorized(func(n int) int {
		if n <= 1 {
			return 1
		}
		return n * factMem(n-1)
	})
println( factMem(5) )
println( factMem(4) )
println( factMem(3) )
```
Thanks to closure and anonymous function we can workaround this problem. You can find the source code on [github](https://gist.github.com/mostafa-asg/cbe12409e480a7ff92db9f46e3e8cf17).

