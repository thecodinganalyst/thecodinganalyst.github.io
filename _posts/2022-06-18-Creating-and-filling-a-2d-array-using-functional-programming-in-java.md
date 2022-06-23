---
title: "Creating and filling a 2D array using Functional Programming in Java"
excerpt_separator: "<!--more-->"
categories:
  - Reference
tags:
  - Java
  - Functional Programming
  - Stream
---

Java stream is a wonderful addition since Java 8 to create and manipulate arrays/lists with ease. However, as other new programming languages can do all the stream functions directly in the lists, it kind of make java a bit more clunky to implement, though it still makes the code much simpler to read than normal loops. Here is an example of how to fill a 2D array.


```
import java.util.stream.IntStream;
import java.util.stream.Stream;
import java.util.function.IntSupplier;
import java.util.Arrays;

public class Program
{
    public static void main(String[] args) {
        int size = 3;
        int value = 1;

        IntSupplier valueSupplier = () -> value;
        int[] arr = IntStream.generate(valueSupplier).limit(size).toArray();
        
        int[][] arr2d = IntStream.range(0, size)
                                    .boxed()
                                    .map(i -> arr.clone())
                                    .toArray(i -> new int[i][]);
        
        for(int[] arr1d: arr2d){
            System.out.println(Arrays.toString(arr1d));
        }
  }
}
```

<!--more-->

We begin by looping the first dimension. Instead of writing `for(int i=0; i<size; i++)`, a neater way is to make use of `IntStream`. `IntStream` is a just a `stream` of `int`, and we can create it with `IntStream.range(0, size)`. We then `boxed()` it, so that we can get a `Stream<Integer>`, which is a proper stream, so that we can use `map` to produce a stream of the type we want, which is an array of `int` - `int[]`, instead of a `IntStream`. 

To create an `int[]` of the same value, we use the static `generate` method of the `IntStream` to generate an infinite size stream of the value we want, then `limit` the size, and simply convert it to an array with `toArray()`. You can see that we outsource the `() -> value` to a `IntSupplier`, just to make the code looks cleaner. A `Supplier` is just a Functional Interface to generate some value.

Back to our 2D array creation, we will map the stream to the clones of `int[]`. Do note that the `clone()` method is very important here, else the 3 arrays will all be pointing to the same address in memory, and anything you do to 1 of the arrays will be applied to the others as well. After that, we use the `toArray(IntFunction<A[]> generator)` to create an array of our type (`int[]`), and that will be a `int[][]`. The IntFunction will take the size of the array required as a parameter to create the array. So here, we need the size of the 1st dimension, hence the `new int[i][]`. 

[https://code.sololearn.com/ckBysSQ9o16H](https://code.sololearn.com/ckBysSQ9o16H)