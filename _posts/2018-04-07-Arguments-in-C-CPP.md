---
layout: post
title: How to implement program arguments and how they are parsed in C/C++
---
###Abstract

C and C++ offer a big level of freedom to the programmer. That is efficiently dangerous, as the programmer has to know what he is doing.
Computers are deterministic machines. They do what they are told to do, and if what they're told is wrong, they will most likely
proceed anyways. While C and C++ compilers do warn programmers and in some cases even refuse to compile, that only happens if the grammar errors (as in programming language grammar) are found.
Don't expect the compiler to try to guess what you try to do. It will assume you know what you try to implement and will not check your code logic for anything other than grammar errors or type errors.
In this post, I will go to the lengths of how you can implement arguments in your program and how they work.

###Implementation

You may scratch your head saying "gee, what has that abstract to do with the topic?". Well, as the compiler assumes you know what you are doing, it will also assume you programmed whatever functions you need.
By default, starting a program `program` with the arguments `-debug -toFile` will do absolutely nothing to the program because a computer is a deterministic machine and it will not implement argument parsing in your program for you.
And even when you do remember that all the functions in a program must be created by a programmer, you can still fail spectacularly.
Arguments in a program may not seem like a big deal, they're pretty beginner things, right? right??? guys??

We often implement the `main()` function something like this, but why?
``
int main(int argc, char *argv[]){
  return 0;
}
``
What means `argc` and why is it an `int`? What means `argv` and why is it a `pointer` to a `char array`?
See? You may know how to make your program to accept arguments at load-time but you don't know the inner workings.
