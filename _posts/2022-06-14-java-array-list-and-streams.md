---
title: "Java Arrays Lists & Streams Reference"
excerpt_separator: "<!--more-->"
categories:
  - Reference
tags:
  - Java
  - Array
  - List
  - Stream
---

```
// Initialize Arrays
Integer[] integerArray = {1, 2, 3, 4, 5};
System.out.println(Arrays.toString(integerArray));

// Initialize List
List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5);
System.out.println(integerList);

// Convert Array to List
List<Integer> convertedList = Arrays.asList(integerArray);
System.out.println(convertedList);

// Convert List to Array
Integer[] convertedArray = integerList.toArray(new Integer[0]);
System.out.println(Arrays.toString(convertedArray));

// Create Stream from List
Stream<Integer> integerStream = integerList.stream();
System.out.println(Arrays.toString(integerStream.toArray()));

// Get Array from Stream
integerArray = IntStream.of(1, 2, 3, 4, 5).boxed().toArray(Integer[]::new);
System.out.println(Arrays.toString(integerArray));

// Get List from Stream
integerList = IntStream.of(1, 2, 3, 4, 5).boxed().collect(Collectors.toList());
System.out.println(Arrays.toString(integerArray));
```

[https://code.sololearn.com/cNcLp5EaoiRV](https://code.sololearn.com/cNcLp5EaoiRV)