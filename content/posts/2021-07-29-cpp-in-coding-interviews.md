---
title: Feedback on C++ solutions on interview training site AlgoExpert
date: 2021-07-29
aliases:
  - /blog/posts/2021-07-29-cpp-in-coding-interviews/
---

Interviews for software engineer job positions often involve coding interviews testing the basic coding and classical data structure and algorithm knowledge and understanding.

[AlgoExport](https://www.algoexpert.io/) is a commercial site to train for coding interviews (including system design and behavior interviews).
It is actually doing a good job at it.
I have been using it when preparing for my job switch recently.

The coding aspect of the site consists of 155 coding questions.
A list of all questions can be seen [here](https://www.algoexpert.io/questions).
The unique selling point compared to [Leetcode](https://leetcode.com/) are the solutions and videos explaining the solutions in detail.

The license to get access to the training material costs around 100 USD.
So we are not talking about a small hobby project here, but a commercial enterprise.
I think this is important when we are talking about what the expectation on quality should be.

The solution always consists of running code and a comment indicating the runtime complexity.
The solutions are available in 9 programming languages.
And here the problems start.
The C++ solutions have quality problems.
While the solutions (as far as I can tell) produce the right results, there are two patterns of problems.

The usage does not reflect good usage of C++.
For example, you will not find `std::unique_ptr`, and many solutions have memory leaks.
That is common across many interview preparation sites and might warrant its own article.
While this is annoying, I will let this slide here.

What I want to focus on is that often the time complexity stated is plain incorrect.
Given that coding interviews often focus so much on the [Big O notation](https://en.wikipedia.org/wiki/Big_O_notation), this is quite bad.
A solution that is significantly slower than advertised is incorrect.

Often, the reason is accidental copying of vectors and strings.
C++ passes objects by value unless otherwise specified, while other languages, e.g. Python and Java, use reference semantics for some variables and value semantics for others.
That means if the code doesn't state it otherwise, C++ will create a new object as a copy of an existing object, e.g. when passing an object as parameter to a function.

```cpp
// just an example, in practice use std::accumulate
int calculate_sum(std::vector<int> numbers)
{
  int s{};
  for (auto i : numbers) {
    s += i;
  }
  return s;
}

vector<int> numbers{1, 2, 3};
int s = calculate_sum(numbers); // creates a (accidental) copy of numbers
```

Equivalent code in Java would not copy the vector and just pass the object as reference.
In C++, the function should instead be declared as `int calculate_sum(std::vector<int> const & numbers)` being explicit in that it is passed as reference and explicit in that the vector should not be mutated.

I reviewed every solution and around ten solutions have material problems in that the runtime complexity stated in the solution does not match the code.

Let's look at an example of the problem:

Note: I will need to quote the solution to some extent.
I will shorten the examples to limit the publication of solutions developed by AlgoExpert while quoting just as much as needed to illustrate the point.

## Example 1: Question Depth-first Search

The question is about implementing a DFS on a tree structure.
The basic algorithm implementation shouldn't come to the surprise of anybody with a CS degree or equivalent.

The solution contains code that is structurally similar to this:

```cpp
vector<string> Node::depthFirstSearch(vector<string> * array) {
    // [dirk] solution contains appending an element to array
    array->push_back(some_value);
    // [dirk] solution contains recursive calls in a loop
    for (auto node : [removed]) {
        node->depthFirstSearch(array);
    }
    return *array;
}
```

The solution time complexity is stated as `O(v + e)` as the standard DFS is.
I am not sure how much sense it makes to say `O(v + e)` in a tree when, by [definition](https://en.wikipedia.org/wiki/Tree_(graph_theory)), `e == v - 1`, but anyway.

Worse, the runtime is not `O(v + e)`, but `O(v * v)`.
Why?
We call `depthFirstSearch` once for every node and add an entry to `array` (an out parameter) every single time.
Each call, we create a copy of the current state of `array`.
You can see the copy operation in line 103 of the assembly output of this [godbolt link](https://godbolt.org/z/o3qeWsvh5).
On average, the size of `array` is `1/2 * n` leading to an overall runtime of `O(n * n)`.

This can be trivially fixed by changing the return value to `void`.
There is just no reason to create a copy of the array in each and every call.
The resulting code is simpler to understand and faster.

## Example 2: Binary Search

The question is about binary search (as the name of the question indicates).
Again, a classical, well-known algorithm.
Solution 1 contains a recursive solution with a time complexity stated as `O(log n)`.
The solution has the same problem with accidental copying.

```cpp
int binarySearchHelper(vector<int> array, int target, int left, int right) {
    // either a or b in the usual algorithm
    // a)
    return binarySearchHelper(array, target, left, middle-1);
    // b)
    return binarySearchHelper(array, target, middle+1, right);
}
```

How can this be fixed?
a) Changing the first parameter to `const &`.
b) Changing the recursive call to `binarySearchHelper(std::move(array), target, l, r)`
c) Using `std::span` (C++20).

`O(log n)` and `O(n log n)` is quite different in theory and practice.

## Example 3: Product Sum

The question uses a `std::vector<std::any>` and recursive calls.
Every entry is either an `int` or another instance of `std::vector<any>`.

Let's shortly mention the fact that a `std::variant` would be the much preferred way to frame this question.
In fact, I had to look up how to use `std::any`.

The solution contains the following code.

```cpp
int productSum(vector<any> array) {
    for (auto el : array) {
        if (el.type() == typeid(vector<any>)) {
            productSum(any_cast<vector<any>>(el));
        }
    }
}
```

The solution is given as `O(n)` with `n` being the number `int` elements.
The problem is that we have three extra copies of the vector and all its sub-vectors and elements per recursive call.
One in the for loop where `for (auto const & el : array) {` would be better.
In addition, there is an additional copy in the line of the recursive call.
The function [`std::any_cast`](https://en.cppreference.com/w/cpp/utility/any/any_cast) here returns a copy of the contained object.

This [godbolt link](https://godbolt.org/z/fas4dWv8o) contains a variant of the solution that actually does have `O(n)` runtime, compared to the `O(n * n)` runtime of the original solution.
Even getting to this answer required me to explore with different alternatives for some time.
I am very skeptical if it is fair to expect this solution (or even this question) of an engineer in a whiteboard interview.

## Other examples

Similar problems exist in other solutions, too:

- Question: "Palindome Check": Solution 3 where the solution is `O(n * n)` instead of `O(n)` due to repeated copies of `std::string` (assuming a C++11 compliant implementation of `std::string`).
- Question "Min Height BST": All three solution presented have the same problem as the Binary Search solution.
The code presented in Solution 1 for example is `O(n * n * log n)` instead of `O(n * log n)` as claimed.
- Question "Simple Cycle Check": The solution is in `O(n * n)` instead of `O(n)`.
- Question "Subarray Sort": The solution is again `O(n * n)` instead of `O(n)` as claimed.
- Question "Dijkstra's Algorithm": The Solution 1 is `O(v * v + e * e)` instead of `O(v * v + e)` due to the usual copying instead of using references problem.
- Question "Shifted Binary Search": If the solution for the question Binary Search has the problem, it is not surprising the Solution 1 to this question as a similar issue: Again `O(n * log n) != O(log n)`
- Question "Search For Range": If the solution for the question Binary Search has the problem, it is not surprising the Solution 1 to this question as a similar issue: Again `O(n * log n) != O(log n)`

The solutions are correct in that they produce the right result (as far as I can tell).
They are incorrect in that the comment about the time complexity is not matching the code.
When I interviewed recently, I was often asked to state the runtime and space complexity of the code.
These mistakes can easily mislead those who trust these solutions too much.

All these problems were easily fixable [by standard C++ practices](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#f15-prefer-simple-and-conventional-ways-of-passing-information).
Nothing fancy.
Just basic C++ practices.

## Examples with issues where it does not lead to a regression in asymptotic runtime

Sometimes there are extra copies of data structures, but the resulting code does not lead to a regression in the asymptotic behavior.

Examples for that are:

- Question "Generate Document": Solution 1 and 2 have problems, Solution 3 is fine.
- Question "Move Element to End": Solution 1. The solution contains an unnecessary copy of a vector. However, the solution is already `O(n)`, so it doesn't make a difference under that metric.
- Question "Merge Overlapping Intervals": Solution 1. The solution copies the vector `O(n log n)` times.
  The code would be made better by using `std::pair` and faster by having the lambda passed to `std::sort` take a `const` reference to avoid needless copy operations.
  Again, no real difference in the final result, but painful to read.
- Question "Task Assignment". Extra copy of the input vector where a `const` reference or a std::move be the much better choices. However, it doesn't make a difference in the asymptotic runtime.
- Question "Permutations": Sure, in an exponential algorithm another `O(n)` doesn't make a difference. However, interestingly the answer to the question named "Powerset", which is structurally similar, is correct.
  While there is a structural problem with the C++ solution, some are correct. It appears that there are multiple authors of the C++ solutions.
- Question "Sort Stack": The solution to Sort Stack again as a return value forcing a copy of the input and again the code is not using the return value. There is no reason for the extra copy of the input vector in the code.
  Changing the return value to `void` and the solution is faster and clearer. However, the solution is already `O(n * n)`, so no real harm done.
- Question "Longest Palindromic Substring": Again extra copy operations that are not needed with more clear, standard C++ code. In this case, for example, instead of copying the input string, `std::string_view` might be a good solution.
- Question "Group Anagrams": This is worth mentioning because it is a different way how you can create copy operations without realizing it. The solution contains code like this.

  ```cpp
    for (auto index : indexes) {
        string word = words[index];
    }
  ```

  The variable `words` is a `vector<string>`. So each round of the loop creates a copy of the string.
  A more normal usage of C++ would be `string const & word = words[index];`

- Question "Valid IP Addresses": Sure, the solution is `O(1)` because the input string is defined to be of length 12 or less, but that doesn't make it a great idea to copy the string multiple times.
It is `O(1)` with unnecessarily large constants.
- Question "Reverse Words in Strings": Same pattern. Usage of references or `std::move` or `std::string_view` would make clearer better code, but the issues present do not have push the solution beyond the `O(n)`.
- Question "Suffix Trie Construction". Unnecessary copy operations, but no regression in asymptotic runtime.
- Question "Same BSTs": Again the same wasteful copy operations, but no overall regression.
- Question "Calender Matching": A lot of copy operations. A few `std::move` and the flow of ownership would have been much clearer. And the code would be faster.

It is fair to ask if, especially given that the Big O notation is correctly stated in these solutions, this isn't premature optimization.
My argument would be that we are not talking about changes that are out of the ordinary.
We are talking about a couple of basic principles of C++ programming.
Premature optimization might be "evil" IMHO if it makes the code less readable, less maintainable.
The proposed changes to not make the code worse.
Often, they make the code actually clearer AND faster.

In addition, performance is often impacted by 1000 paper cuts.
In most programs, none of these extra copies will show up in a profiler, but they are still wasteful.
Don't do wasteful stuff.
A good C++ program should not have these problems, even if they do not cause a regression in the complexity estimation.
Chandler Carruth's [his 2014 Cppcon talk](https://youtu.be/fHNmRkzxHWs) "Efficiency with Algorithms, Performance with Data Structures" is worth viewing for more information about the topic.

Most cases are caused by the missing usage of references (or `std::move` or `std::string_view`/`std::span`) leading to wasteful copying of large data structures.
This is not a single problem in one question, but given that it is common in many solutions, it is a systematic problem on the site.

It is interesting that it is not the case in all solutions.
I think that the site has multiple authors, and some authors were aware of these kinds of issues and others weren't.
Many solutions use references (correctly).
References (and even const references) are not some obscure element of the language, but a core element to it.

The most likely explanation is that the solutions of other programming languages were translated into C++ without taking the differences between the languages into account.
And this direct translation is the source of the problem.
Given that the solutions and videos are the unique selling point of AlgoExpert, I would have expected more.

I did write the support team about these issues in November 2020.
At this point, eight months later, the solutions have not been fixed, which is sad.

Let me be clear: AlgoExpert is a good service for coding interview preparation.
I am recommending it.
The selection of questions is okay.
The videos are well-made and the topics well explained.
But I am disappointed when statements, comments and code do not match up in such fundamental ways.
I hope that the team eventually goes back to their solutions and fixes these issues.
Until then, I would recommend to not use their C++ solution.