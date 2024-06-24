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

# Additional Feature

You are then free to implement <span class="orange-bold">any additional feature(s)</span> to your shell as you wish:
1. Decorate the prompt to include useful information like current path, current user, and time
2. Expand interpretation ability beyond setting PATH environment variables and executing startup commands: setting other environment variables or setting theme and colorn scheme 
3. Add command history configuration to the shell 
4. Add a basic tab autocompletion ability when traversing the file system 
5. Anything else you can think of 

{:.task-title}
> Additional Shell Feature 
>
> This accounts for 1.5% of your grade (PA1 in total accounts for 10% of your grades). Be <span class="orange-bold">creative</span>, you are to implement one or two great functionality (there's no need to implement 50 of them). *There's no rubric for this.* It is both objective (in terms of functionality and difficulty of implementation) and subjective (in terms of usefulness and looks).  

# Sustainability and Inclusivity 

There have been some ongoing initiatives from SUTD as well as the Engineering Accreditation Board to include more elements of Sustainability, Diversity and Inclusivity. As such, we encourage you to consider the impact of your project on sustainability, as well as any consideration for diversity and inclusion (e.g., different cultures, demographic groups, etc). 

Here are some suggestions, it is sufficient to fulfil **one** of each category (1 idea for sustainability, 1 idea for inclusivity).  You may apply it in *any* part of your assignment, be it your additional shell feature, enhancing the system programs, or the overall look of your shell. It does **not** have to be part of your code: e.g if you decide to conduct user testing. Simply highlight your attempts clearly in the README file and write future suggestions (if any).

{:.error}
Failure to show attempts to incorporate initiatives to support sustainability and inclusivity results in <span class="orange-bold">-3% (penalty)</span> of your overall grades. 

### Sustainability Ideas

1. **Energy-Efficient Algorithms**:
   - **Description**: Write algorithms that are optimized for energy efficiency.
   - **Task**: Implement an algorithm that sorts data efficiently or processes files with minimal CPU usage. Measure and compare the energy consumption of different approaches.

2. **Resource Usage Feedback**:
   - **Description**: Implement features that provide feedback on resource usage for each command executed.
   - **Task**: Enhance your shell to display statistics such as CPU time, memory usage, and disk I/O for each executed command, promoting awareness of resource consumption.

3. **Idle Resource Management**:
   - **Description**: Implement a feature that reduces resource usage when the shell is idle.
   - **Task**: Add functionality to put the shell into a low-power mode when not in use, waking up only when a new command is entered.

4. **Environment Variable for Efficiency**:
   - **Description**: Utilize environment variables to optimize resource usage based on user preferences.
   - **Task**: Implement and document environment variables that control the shell's behavior, such as limiting the number of concurrent processes or adjusting the shell's polling intervals to save energy.

5. **Lightweight System Programs**:
   - **Description**: Focus on creating system programs that are designed to be lightweight and efficient.
   - **Task**: Develop a lightweight version of tools like `ps` or `neofetch`, ensuring they use minimal system resources and run efficiently.

### Inclusivity Ideas

1. **Enhanced Accessibility Features**:
   - **Description**: Make the shell accessible to users with disabilities by including features that aid in navigation and usage.
   - **Task**: Implement features such as keyboard shortcuts, voice commands, or compatibility with screen readers to ensure the shell is usable by everyone.

2. **Customizable Interface**:
   - **Description**: Allow users to customize the shell interface to meet their individual needs.
   - **Task**: Add options for users to customize the appearance and behavior of the shell, such as changing text size, color schemes, and prompt styles.

3. **Multilingual Support**:
   - **Description**: Provide support for multiple languages to cater to a diverse user base.
   - **Task**: Implement localization features that allow the shell and system programs to operate in different languages, with easy switching between them.

4. **Inclusive Error Messaging**:
   - **Description**: Design error messages to be clear, informative, and user-friendly, avoiding technical jargon.
   - **Task**: Write detailed, plain language error messages that help users understand and fix issues, including links to relevant documentation or tips.

5. **User Profiles and Preferences**:
   - **Description**: Allow users to create profiles that save their preferences and settings.
   - **Task**: Develop a system where users can save their interface customizations, preferred languages, and frequently used commands, ensuring a personalized and inclusive experience every time they use the shell.

{:.important-title}
> Don't forget!
> 
> Write a README explaining how your implementation considers both sustainability and inclusivity, detailing the specific features and design choices made to address these aspects as part of your submission. Push it to your project repo. 


# Submission 

You should commit and push your work to your Github Classroom repository for PA1. You will lose all admin rights to your repo after the due date (check course calendar).

{:.error}
No <span class="orange-bold">late submission</span> will be entertained. You will obtain zero marks if you fail to submit before the due date. 

## README 

You are expected to write a <span class="orange-bold">clear</span> and <span class="orange-bold">succint</span> README file as part of your project submission, detailing the following:
1. How to compile and run your shell
2. Builtin functions supported
3. Additional features supported 
4. How you have considered sustainability and/or inclusivity in your assignment

# Live Demo

{:.highlight}
You will be asked to clone your project from the github classroom repo (<span class="orange-bold">fresh clone</span>). 

## Sample Demo 
You have to be able to demo the workings of your `cseshell` as follows: (note that you're free to implement other system programs, in this example we implement some simple `ps`):

<video controls width="100%" class="center_ninety" autoplay>
    <source src="{{ site.baseurl }}docs/Programming Assignment/pa1/images/pa1-demo.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>

## Checkoff Procedure
You and your entire team should appear in class during your own cohort's lab session (check exact schedule in class calendar) where you shall `clone` the code you submitted and demonstrate it in your computer. We will ask several implementation-related questions, and even ask you to <span class="orange-bold">modify</span> some features on the spot. We will also test your shell by modifying `.cseshellrc`, checking all the builtin commands, check the 4 system programs implemented, and also your additional feature(s). All members must be present and must be able to answer questions. <span class="orange-bold">We will pick who should answer the question</span>. Hence it is important that everyone knows what's going on, at least in theory.


{:.highlight}
If all goes well, you should obtain a maximum score of 10% of your overall 50.005 grades. 

Here's the breakdown of PA1 assessment, based on each feature: 
1. Shell main loop (2%): this is a <span class="orange-bold">requirement</span>, without this, you will automatically obtain 0 marks for PA1. Upon proper bugless completion, you get 2%. Otherwise, if your shell is buggy and only work under ideal condition, you obtain a max of 1%
2. Shell builtin functions (1.5%): `cd` (0.25%), `help` (0.25%), `usage` (0.25%), `env` (0.25%), `setenv` (0.25%), `unsetenv` (0.25%)
3. `.cseshellrc` interpretation (2%): `PATH` setting (0.75%), Command execution (0.75%), able to handle empty lines (0.5%)
4. System programs (3%): `sys` (0.5%), `dspawn` (1%), `dcheck` (0.75%), `backup` (0.75%)
5. Additional functionality (1.5%) 
6. Incorporate attempts to support sustainability and inclusivity (**failure** to do so results in -3%)

**Total maximum PA1 grades: 10%**. 

{:.important-title}
> Prepare everyone
> 
> ALL group members must be present. Failure to turn up during demo grants 0 marks for that person missing the checkoff (unless LOA is granted). You must also be able to answer any questions pertaining to your own code, failure to do so grants us the right to <span class="orange-bold">revoke</span> the grades for any task even though the demo was *successful*. We will ask any group members in random, so ensure that everyone's on the same page before turning up for the demo. <span class="orange-bold">Don't worry, we will ask basic, reasonable questions</span>.
> 
> We reserve the right to revoke any marks given for any features even though the demo is working if you <span class="orange-bold">fail</span> to answer some <span class="orange-bold">basic</span> query about it. Ensure that you have submitted your own work. We also reserve the right to give you 0 marks for PA1 and report you for further disciplinary action should we find sufficient evidence of <span class="orange-bold">plagiarism</span> and report you to OSA for further disciplinary action. 