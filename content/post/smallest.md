---
title: "An Interesting Question of Stack."
date: 2018-09-20T12:00:00+08:00
weight: 10
keywords: ["golang"]
description: "An Interesting Question of Stack."
tags: ["golang"]
categories: ["Dev"]
author: "Fred"
banner: ""
---

Recently, I read a article on the internet which raise a question about stack. The question is that how to get the smallest number for the best practice when pushing data on a stack and then poping it. I try to write down it in golang. Please have a look.

```
package main

import "fmt"

type Stack struct {
	source []int
	data   []int
	small  int
}

func (s *Stack) push() {
	for _, val := range s.source {
		s.data = append(s.data, val)
		fmt.Printf("Current data: %v. The smallest element: %v. \n", s.data, s.small)
	}
}

func (s *Stack) pop() {
	for length := len(s.data); length > 0; length-- {
		s.data = append(s.data[:length-1])
		fmt.Printf("Current data: %v. The smallest element: %v. \n", s.data, s.small)
	}
}

func main() {
	s := &Stack{}
	s.source = []int{5, 4, 3, 4, 6, 3, 2, 5, 3, 2}
	s.data = []int{}

	s.push()
	s.pop()
}
```
We need to modify the program for the output as below.
```
Current data: [5]. The smallest element: 5.
Current data: [5 4]. The smallest element: 4.
Current data: [5 4 3]. The smallest element: 3.
Current data: [5 4 3 4]. The smallest element: 3.
Current data: [5 4 3 4 6]. The smallest element: 3.
Current data: [5 4 3 4 6 3]. The smallest element: 3.
Current data: [5 4 3 4 6 3 2]. The smallest element: 2.
Current data: [5 4 3 4 6 3 2 5]. The smallest element: 2.
Current data: [5 4 3 4 6 3 2 5 3]. The smallest element: 2.
Current data: [5 4 3 4 6 3 2 5 3 2]. The smallest element: 2.
Current data: [5 4 3 4 6 3 2 5 3]. The smallest element: 2.
Current data: [5 4 3 4 6 3 2 5]. The smallest element: 2.
Current data: [5 4 3 4 6 3 2]. The smallest element: 2.
Current data: [5 4 3 4 6 3]. The smallest element: 3.
Current data: [5 4 3 4 6]. The smallest element: 3.
Current data: [5 4 3 4]. The smallest element: 3.
Current data: [5 4 3]. The smallest element: 3.
Current data: [5 4]. The smallest element: 4.
Current data: [5]. The smallest element: 5.
Current data: []. The smallest element: 0.
```
What's your advice? I think there will be many solutions to get the result. However, we need to consider more about the best practice. Please try your hand to write it.

If you have no idea about the best practice, here is some thinking for your reference.

1. Try to reduce time complexity O(n) of your method. This means try not to traverse the data stack and reduce the time of reading data.

2. Try to reduce space complexity S(n) of your method. This means reduce the memory cost and the space of the auxiliary stack you will use.

The answer is as following.
***
The answer of that article is to use index. Before I read the answer, I can only complete the task "1.". Now, I try to write down my version to complete task "2.". How about yours?

The output with the auxiliary stack of index is as below. That is really awesome.
```
Current data: [5]. The smallest element: 5.
Auxiliary: [0].
Current data: [5 4]. The smallest element: 4.
Auxiliary: [0 1].
Current data: [5 4 3]. The smallest element: 3.
Auxiliary: [0 1 2].
Current data: [5 4 3 4]. The smallest element: 3.
Auxiliary: [0 1 2].
Current data: [5 4 3 4 6]. The smallest element: 3.
Auxiliary: [0 1 2].
Current data: [5 4 3 4 6 3]. The smallest element: 3.
Auxiliary: [0 1 2].
Current data: [5 4 3 4 6 3 2]. The smallest element: 2.
Auxiliary: [0 1 2 6].
Current data: [5 4 3 4 6 3 2 5]. The smallest element: 2.
Auxiliary: [0 1 2 6].
Current data: [5 4 3 4 6 3 2 5 3]. The smallest element: 2.
Auxiliary: [0 1 2 6].
Current data: [5 4 3 4 6 3 2 5 3 2]. The smallest element: 2.
Auxiliary: [0 1 2 6].
Current data: [5 4 3 4 6 3 2 5 3]. The smallest element: 2.
Auxiliary: [0 1 2 6].
Current data: [5 4 3 4 6 3 2 5]. The smallest element: 2.
Auxiliary: [0 1 2 6].
Current data: [5 4 3 4 6 3 2]. The smallest element: 2.
Auxiliary: [0 1 2 6].
Current data: [5 4 3 4 6 3]. The smallest element: 3.
Auxiliary: [0 1 2].
Current data: [5 4 3 4 6]. The smallest element: 3.
Auxiliary: [0 1 2].
Current data: [5 4 3 4]. The smallest element: 3.
Auxiliary: [0 1 2].
Current data: [5 4 3]. The smallest element: 3.
Auxiliary: [0 1 2].
Current data: [5 4]. The smallest element: 4.
Auxiliary: [0 1].
Current data: [5]. The smallest element: 5.
Auxiliary: [0].
Current data: []. The smallest element: 0.
Auxiliary: [].
```

Here is my code.
```
package main

import "fmt"

type Stack struct {
	source    []int
	data      []int
	small     int
	auxiliary []int
}

func (s *Stack) push() {
	for num, val := range s.source {
		s.data = append(s.data, val)
		if num == 0 || val < s.data[s.auxiliary[len(s.auxiliary)-1]] {
			s.auxiliary = append(s.auxiliary, num)
			s.small = s.data[s.auxiliary[len(s.auxiliary)-1]]
		}
		fmt.Printf("Current data: %v. The smallest element: %v. \nAuxiliary: %v. \n", s.data, s.small, s.auxiliary)
	}
}

func (s *Stack) pop() {
	for length := len(s.data); length > 0; length-- {
		if length-1 == s.auxiliary[len(s.auxiliary)-1] {
			s.auxiliary = append(s.auxiliary[:len(s.auxiliary)-1])
			if len(s.auxiliary)-1 > -1 {
				s.small = s.data[s.auxiliary[len(s.auxiliary)-1]]
			} else {
				s.small = 0
			}
		}
		s.data = append(s.data[:length-1])
		fmt.Printf("Current data: %v. The smallest element: %v. \nAuxiliary: %v. \n", s.data, s.small, s.auxiliary)
	}
}

func main() {
	s := &Stack{}
	s.source = []int{5, 4, 3, 4, 6, 3, 2, 5, 3, 2}
	s.data = []int{}

	s.push()
	s.pop()
}
```
