---
layout: default
permalink: /pa2/submission
title: Submission and Demo
description: Submission Protocol
parent: Programming Assignment 2
grand_parent: Programming Assignment
nav_order:  3
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



If you have reached this stage, you have completed 95% of this assignment, <span style="color:#f7007f;"><b>congratulations</b></span>! One final step is some <span style="color:#f77729;"><b>administrative work</b></span> that will help your instructors grade your assignments seamlessly.

Please read this section very very carefully and watch the given demo with your full attention.
{:.error}

You are expected to <span style="color:#f77729;"><b>save</b></span> <span class="orange-bold">two</span> additional files each time a client sends a file to the server under [PROJ_ROOT_DIR]:

1. Client to save encrypted files to `/send_files_enc` before sending, with the name: `enc_[filename]`, e.g: `enc_image.ppm` if it is sending `image.ppm` to the server.
2. Server to save received encrypted file to client to `/recv_files_enc` before decrypting, with the name `enc_recv_[filename]`, e.g: `enc_recv_image.ppm`.

You should have the following files now:
<img src="{{ site.baseurl }}/docs/Programming Assignment/pa2/images/03-submission/2024-07-25-14-19-48.png"  class="center_fifty no-invert" />

The new files are mainly (<span class="orange-bold">names must be exactly these</span>):

```sh
1. server_signed.crt, server_private_key.pem # in source/auth
2. ClientWithSecurity[AP|CP1|CP2].py # in source/
3. ServerWithSecurity[AP|CP1|CP2].py # in source/
```

All other files inside `[PROJ_ROOT_DIR]/files` directory must **remain** as per the original files from the previous sections.


You should then make the final commit of all tracked files so far:

```sh
git commit -a -m "feat: save to recv_files_enc and send_files_enc before encrypting and before sending"
```

You can also put your own large files at `[PROJ_ROOT_DIR]/files` (>100MB) to <span style="color:#f77729;"><b>test</b></span> internally, but <span style="color:#f77729;"><b>remove</b></span> it before pushing to Github since it doesn't allow you to push any single file >100MB. Please <span class="orange-bold">do not use<span class="orange-bold"></span></span> `git-lfs`.  
{:.note}

# Print Statements

We will not check your program's `stdout` during the demo, so you're <span style="color:#f77729;"><b>free to print</b></span> whatever you need or want during the development of the assignment.

# Following Protocols Closely (1%)

As stated throughout the handout, you need to stick to the protocol <span style="color:#f77729;"><b>strictly</b></span>, that is to implement each `MODE` as specified (`send` or `read` exactly as specified in the `MODE`) so that we can run your server scripts against our <span style="color:#f77729;"><b>answer key</b></span> client scripts and vice versa. <span class="orange-bold">This is where the last 1% of your grade come from.</span>

You may easily ensure this by checking your server against another pair's client, and vice versa. To summarize:  
- Ensure the padding schemes <span style="color:#f77729;"><b>are correct</b></span>:
   - You (client/server) use **either** `OAEP` or `PKCS1v15` for **encryption** and **decryption** of file data when RSA is used 
   - You (server) use `PSS` for **signing** 
   - Client uses `PKCS1v15` to **verify** `server.crt` using cacsertificate's public key because that's the padding scheme used by our bot 

```py
# OAEP
padding.OAEP(mgf=padding.MGF1(hashes.SHA256()), algorithm=hashes.SHA256(),label=None)`
# PKCS1v15
padding.PKCS1v15()
# PSS
padding.PSS(mgf=padding.MGF1(hashes.SHA256()), salt_length=padding.PSS.MAX_LENGTH)
```

- Ensure that `Fernet` is used to generate symmetric key and utilise it (encryption/decryption)
- **AP**: Ensure that your `MODE 3`: sticks to the protocol
   * Client to send **TWO** messages first upon connection establishment: `M1` (int, indicating the size of incoming message `M2`) and `M2` (the message)
   * Server to respond with **FOUR** messages (two pairs of size `int` and the corresponding message). This marks the **end** of `MODE 3`
   * No other message exchanges allowed in `MODE 3`
   * Whatever "fix" you do in **AP** should <span class="orange-bold">not</span> change these facts and should be incorporated into `MODE 3` protocol
- **CP1**: Ensure that in `MODE 1`, `M1` contains the **total encrypted file length** (sent by client). It is up to you to send `M2` repeatedly as 128-byte chunks *or* to accumulate encrypted 128 byte chunks and then send it all in one single `send()` socket call. 
- Do <span class="orange-bold">not</span> create any new modes. We only expect: `MODE 0 - MODE 4`. 

Not to worry, things will work seamlessly if you stick to these protocols. 
{:.note}

# Sustainability and Inclusivity 

As mentioned in Programming Assignment 1, there have been some ongoing initiatives from SUTD as well as the Engineering Accreditation Board to include more elements of Sustainability, Diversity and Inclusivity. As such, we encourage you to consider the impact of your project on sustainability, as well as any consideration for diversity and inclusion (e.g., different cultures, demographic groups, etc). 

Here are some suggestions, it is sufficient to fulfil **one** from each.  You may apply it as *any* part of your assignment, as long as you're <span class="orange-bold">still</span> following the protocols requested closely. It does **not** have to be part of your code: e.g if you decide to conduct user testing. Simply highlight your attempts clearly in the README file and write future suggestions (if any).
 
{:.error}
Failure to show attempts to incorporate initiatives to support sustainability and inclusivity results in <span class="orange-bold">-3% (penalty)</span> of your overall grades. 

### Sustainability Ideas

1. **Optimized File Transfer**:
   - **Description**: Design the protocol to transfer files efficiently, minimizing energy consumption and network usage.
   - **Task**: Implement features such as file compression before transfer and efficient chunking of data to reduce the amount of data sent over the network while still following the protocols closely

2. **Efficient Encryption Practices**:
   - **Description**: Use energy-efficient practices within the constraints of the Python cryptography modules.
   - **Task**: Choose the most efficient algorithms provided by the Python cryptography library for encryption and decryption to ensure minimal resource usage while maintaining security.

3. **Logging and Monitoring**:
   - **Description**: Include logging and monitoring to track resource usage during file transfers.
   - **Task**: Implement logging of CPU, memory, and network usage during file transfers, and analyze these logs to identify and optimize inefficient parts of the protocol.

4. **Documentation of Efficient Practices**:
   - **Description**: Document best practices for writing efficient and sustainable code.
   - **Task**: Include a section in the README detailing efficient coding practices and optimizations used in the project, encouraging the use of these practices in future development.

### Inclusivity Ideas

1. **Accessible User Interface**:
   - **Description**: Design the client interface to be accessible to users with disabilities.
   - **Task**: Ensure the client application supports accessibility features such as keyboard navigation, screen reader compatibility, and customizable text size and color schemes.

2. **Multilingual Support**:
   - **Description**: Provide support for multiple languages in the client and server applications.
   - **Task**: Implement localization features allowing users to choose their preferred language for interaction with the client and server.

3. **Clear and Inclusive Documentation**:
   - **Description**: Write clear, detailed documentation that is easy to understand for users of all backgrounds.
   - **Task**: Create comprehensive documentation that includes step-by-step instructions, clear explanations of technical terms, and helpful examples, avoiding jargon.

4. **User-Friendly Error Messages**:
   - **Description**: Provide informative and user-friendly error messages.
   - **Task**: Implement error messages that are clear, descriptive, and offer actionable steps to resolve issues, ensuring they are easily understandable for all users.

5. **Inclusivity in Testing**:
   - **Description**: Ensure that the system is tested with a diverse range of users.
   - **Task**: Conduct user testing with individuals from various backgrounds and with different abilities, gathering feedback to improve the inclusivity and usability of the system.


{:.important-title}
> Don't forget!
> 
> Write a README explaining how your implementation considers both sustainability and inclusivity, detailing the specific features and design choices made to address these aspects as part of your submission. Push it to your project repo. 
>
> Do <span class="orange-bold">NOT</span> break protocol (e.g add additional `MODE`).

# Submission

Simply push to your github classroom repo **before the assignment's due date**. <span class="orange-bold">No late submission is allowed</span>

### README 

You are expected to write a <span class="orange-bold">clear</span> and <span class="orange-bold">succint</span> README file as part of your project submission, detailing the following:
1. How to compile and run your programs (server and client)
2. How client can upload multiple files (if you modify the current starter code)
3. How you have considered sustainability and/or inclusivity in your assignment

# Live Demo 

{:.highlight}
You will be asked to clone your project from the github classroom repo (<span class="orange-bold">fresh clone</span>). 

## Sample Demo

The following video shows the expected interaction between Server and Client processes.

<video controls width="100%" class="center_ninety" autoplay>
    <source src="{{ site.baseurl }}/docs/Programming Assignment/pa2/images/index/assets/images/pa2/pa2-demo.mp4" type="video/mp4">
    Sorry, your browser doesn't support embedded videos.
</video>


## Checkoff Procedure

Please be present at your designated slots (refer to course handout) for your PA2 demo. Even though you have submitted your code, we still want to witness your PA2 **in person**. You will <span class="orange-bold">clone</span> your submitted code (fresh) and run it <span class="orange-bold">on a different machine</span>. If you're doing the project alone, our TA/Instructor will be your client. 


{:.highlight}
If all goes well, you should obtain a maximum score of 8% of your overall 50.005 grades. 

Here's the breakdown of PA2 assessment, based on each feature: 
1. Task 1 (3%): Authentication Protocol
2. Task 2 (2%): Confidentiality Protocol 1 (CP1)
3. Task 3 (2%): Confidentiality Protocol 2 (CP2)
4. Following given protocols closely (1%)
5. Incorporate attempts to support sustainability and inclusivity (**failure** to do so results in -3%)

**Total maximum PA2 grades: 8%**. 


{:.important-title}
> Prepare everyone
>
> ALL group members must be present. Failure to turn up during demo grants 0 marks for that person missing the checkoff (unless LOA is granted). You must also be able to answer any questions pertaining to your own code, failure to do so grants us the right to <span class="orange-bold">revoke</span> the grades for any task even though the demo was *successful*. We will ask any group members in random, so ensure that everyone's on the same page before turning up for the demo. <span class="orange-bold">Don't worry, we will ask basic, reasonable questions</span>.
>
> We reserve the right to revoke any marks given for any features even though the demo is working if you <span class="orange-bold">fail</span> to answer some <span class="orange-bold">basic</span> query about it. Ensure that you have submitted your own work. We also reserve the right to give you 0 marks for PA1 and report you for further disciplinary action should we find sufficient evidence of <span class="orange-bold">plagiarism</span> and report you to OSA for further disciplinary action. 