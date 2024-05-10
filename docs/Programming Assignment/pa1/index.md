---
layout: default
title: Programming Assignment 1
permalink: /pa1/intro
has_children: true
parent: Programming Assignment
nav_order: 1
---

# Programming Assignment 1 
{: .no_toc}


In this assignment, you are tasked to expand the functionality of a <span style="color:#f77729;"><b>shell</b></span> as well as a <span style="color:#f77729;"><b>daemon</b></span> process, both of which are the common applications of `fork()`.

The assignment is written entirely in C. At the end of this assignment, you should be able to:

- <span style="color:#f77729;"><b>Create</b></span> a shell and wait for user input
- Write several other <span style="color:#f77729;"><b>system programs</b></span> that can be invoked by the shell
- <span style="color:#f77729;"><b>Parse</b></span> user input and invoke `fork()` with the appropriate program
- Create a program that results in a <span style="color:#f77729;"><b>daemon</b></span> process
- Use your shell to keep track of the state of your daemon processes
- Create and maintain environment variables 
- Customise your shell via simple `.rc` file

You may complete this assignment in <span style="color:#f7007f;"><b>groups of 2-3 pax</b></span>. Indicate your partner's name in the google sheet provided in our course handout.
{:.info}

## Starter Code

You might want to run this assignment in your **POSIX compliant OS**, using `gcc` version 10.X or below. **Using version 11.X and above** might be _okay_, but the bot is running `gcc 10` version so if you use some newer fancy stuffs then the bot might not be able to run it. You can try first.
{:.error}

You should have joined the GitHub Classroom and obtain the starter code for this assignment there. The link can be found in the Course Calendar portion of your Course Handout.

## Submission 

Simply push your completed shell code to the github classroom repository by the stipulated due date in the course handout. You will then meet our TAs/instructors for a live demo (checkoff, schedule in course handout). You are to expand the shell to implement various **additional functionalities** that will be elaborated throughout this assignment's handout. Read along to find out more. 

Your shell should <span style="color:#f7007f;"><b>NOT</b></span> crash due to any input from the user or any <span style="color:#f7007f;"><b>ABSENCE</b></span> of input from the user after the given command.
{:.error}