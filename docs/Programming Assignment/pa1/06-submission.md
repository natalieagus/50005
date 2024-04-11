---
layout: default
permalink: /pa1/part6
title: Submission and Demo 
description: Admin details on submission and demo of PA1
parent: Programming Assignment 1
grand_parent: Programming Assignment
nav_order:  6
---


* TOC
{:toc}

**50.005 Computer System Engineering**
<br>
Information Systems Technology and Design
<br>
Singapore University of Technology and Design
<br>
**Natalie Agus (Summer 2024)**

# Submission and Demo 
{: .no_toc}

{:.error-title}
> 🔴 Check
> 
> Ensure that you have implemented all the following tasks before proceeding:
> 1. Implement **shell main loop** 
> 2. Implement all 7 shell builtin commands: `cd, help, exit, usage, env, setenv, unsetenv`
> 3. Implement the functionality to read and interpret `.cseshellrc`  
> 4. Implement 4 more system programs: `sys`, `dspawn`, `dcheck`, and `backup`  

## Additional Feature

You are then free to implement <span class="orange-bold">any additional feature(s)</span> to your shell as you wish:
1. Decorate the prompt to include useful information like current path, current user, and time
2. Expand interpretation ability beyond setting PATH environment variables and executing startup commands: setting other environment variables or setting theme and colorn scheme 
3. Add command history configuration to the shell 
4. Add a basic tab autocompletion ability when traversing the file system 
5. Anything else you can think of 

{:.task-title}
> Additional Shell Feature 
>
> This accounts for 1.5% of your grade (PA1 in total accounts for 10% of your grades). Be <span class="orange-bold">creative</span>, you are to implement one or two great functionality (there's no need to implement 100). *There's no rubric for this.* It is both objective (in terms of functionality and difficulty of implementation) and subjective (in terms of usefulness and looks).  

## Submission 

You should commit and push your work to your Github Classroom repository for PA1. You will lose all admin rights to your repo after the due date (check course calendar).

{:.error}
No <span class="orange-bold">late submission</span> will be entertained. You will obtain zero marks if you fail to submit before the due date. 


## Demo 

You and your team should appear in class during your own cohort's lab session (check exact schedule in class calendar) where you shall `clone` the code you submitted and demonstrate it in your computer. We will ask several implementation-related questions, and even ask you to <span class="orange-bold">modify</span> some features on the spot. We will also test your shell by modifying `.cseshellrc`, checking all the builtin commands, check the 4 system programs implemented, and also your additional feature(s). All members must be present and must be able to answer questions. <span class="orange-bold">We will pick who should answer the question</span>. Hence it is important that everyone knows what's going on, at least in theory.


{:.highlight}
If all goes well, you should obtain a maximum score of 10% of your overall 50.005 grades. 

Here's the breakdown of PA1 assessment, based on each feature: 
1. Shell main loop (2%): this is a <span class="orange-bold">requirement</span>, without this, you will automatically obtain 0 marks for PA1. Upon proper bugless completion, you get 2%. Otherwise, if your shell is buggy and only work under ideal condiiton, you obtain a max of 1%
2. Shell builtin functions (1.5%): `cd` (0.25%), `help` (0.25%), `usage` (0.25%), `env` (0.25%), `setenv` (0.25%), `unsetenv` (0.25%)
3. `.cseshellrc` interpretation (2%): `PATH` setting (0.75%), Command execution (0.75%), able to handle empty lines (0.5%)
4. System programs (3%): `sys` (0.5%), `dspawn` (1%), `dcheck` (0.75%), `backup` (0.75%)
5. Additional functionality (1.5%) 

**Total PA1 grades: 10%**.

{:.important}
We reserve the right to revoke any marks given for any features even though the demo is working if you <span class="orange-bold">fail</span> to answer some <span class="orange-bold">basic</span> query about it. Ensure that you have submitted your own work. We also reserve the right to give you 0 marks for PA1 and report you for further disciplinary action should we find sufficient evidence of <span class="orange-bold">plagiarism</span>.