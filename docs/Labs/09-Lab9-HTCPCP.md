---
layout: default
permalink: /labs/09-htcpcp
title: HTCPCP
description:  Understand how HTTP works via HTCPCP Example
parent: Labs
nav_order:  8
---

* TOC
{:toc}

**50.005 Computer System Engineering**
<br>
Information Systems Technology and Design
<br>
Singapore University of Technology and Design
<br>
**Natalie Agus (Summer 2025)**

# Hyper Text Coffee Pot Control Protocol
{:.no_toc}


In NS Module 5, we learnt about the Client-Server and the Web, mainly about basics of socket programming and the HTTP message request/replies.

At the end of this lab exercise, you should be able to:

- **Deploy** web server and client applications
- **Understand** how HTTP works
- **Implement** a toy variant of HTTP called HTCPCP
- Explore **differences** between HTTP and HTTPs using packet sniffer (Wireshark / termshark)
- **Analyse** various captured packets exchanged between the browser, web server, and HTCPCP server



# The Hyper Text Coffee ☕️ Pot Control Protocol

The Hyper Text Coffee Pot Control Protocol (**HTCPCP**) is a whimsical communication protocol for controlling, monitoring, and diagnosing coffee pots that is **based** on [HTTP](https://www.rfc-editor.org/rfc/rfc2616). It is specified in [RFC 2324](https://datatracker.ietf.org/doc/html/rfc2324), published on 1 April 1998 as part of an April Fools prank. In this lab, we are not going to control a real coffee pot (although it is [possible](https://github.com/HyperTextCoffeePot/HyperTextCoffeePot)) but a **virtual** one via the HTCPCP protocol.

This lab is created to give you some kind of understanding on how to deploy fullstack (sort of) application that can be accessible via the network. You will then deploy both the web application and the HTCPCP server and sniff the packets exchanged using Wireshark.

> [The base code for project was originally taken from here](https://jamesg.blog/2021/11/18/hypertext-coffee-pot/), refactored, styled and adapted with more functionalities added to suit our learning experience in the lab. Special thanks to 2023 CSE TAs Cassie and Ryan for the inspiration, ideas, and contribution to create this lab.

# Submission 

Once you have completed all tasks, schedule a checkoff as a Team with your Lab TA by next week Friday 6PM. 

{:.task-title}
> ✅ Checkoff
> 
> Demonstrate the features implemented in Task 1 to our TAs to obtain the checkoff mark for this lab.


### System requirements

You need Python 3.10 (it won't work with Python 3.12) and `uv` to run this project, and Wireshark (or equivalent) installed in your system. You are free to use any CLI or GUI based network protocol analyser. The latter is recommended for beginners. The rest of this lab is written with the assumption that you used **Wireshark**. Other equivalent network protocol analyser should have similar functionalities.

### Source Code

Clone the repository for this lab:

```
git clone https://github.com/natalieagus/lab_htcpcp
```

Then, install the requirements and launch the venv:

```
uv sync
source .venv/bin/activate
```


There are two main processes to run:

1. A HTCPCP compliant Coffee Pot **Server**: implemented in Python using the socket library that accepts requests from `coffee://` URI scheme (instead of http or https).
2. A full-stack web application (using Python Flask) that serves a regular HTTP-based web client and also help you send HTCPCP requests to the coffee pot server using your web browser.

You can spawn both processes from the main file:
```
python main.py 
```

<img src="{{ site.baseurl }}//docs/Labs/images/09-Lab9-HTCPCP/2024-04-09-16-19-38.png"  class="center_full no-invert"/>

Then type the following in your web browser: <http://127.0.0.1:5031>. 

Otherwise, you can also spawn them on two separate terminal sessions. First, spawn the coffee pot server:

```
python server/server_pot.py
```

We assume that `python` is an alias to `python3` in your system.
{:.error}

Then, spawn the web application (`http`):

```
python webapp/webapp_coffee.py
```

<img src="{{ site.baseurl }}//assets/images/lab4_1-intro/2023-06-08-11-22-20.png"  class="center_full no-invert"/>

{:.warning}
The code is now incomplete, so at best you're seeing an empty response. 

<img src="{{ site.baseurl }}//docs/Labs/images/09-Lab9-HTCPCP/2024-04-09-18-51-36.png"  class="center_full no-invert"/>
### Options

The web application can receive two more options: `-https` and `-local` to host the website via `https` or locally only. The coffee pot server can also be set to be hosted locally only using the `-local` option.

### `localhost` vs `127.0.0.1`

You might have heard the term `localhost` or `127.0.0.1` if you just want to spawn a web server locally during development Both "localhost" and "127.0.0.1" are used to refer to the _local host_ or the **loopback interface** of a device.

"localhost" is a **hostname** that is used to refer to the current device itself. It is a standard hostname that resolves to the loopback IP address, which is typically "127.0.0.1" in IPv4 or "::1" in IPv6.

{:.note}
When you access "localhost" in a web browser or any other network application on your device, it is **resolved** to the loopback interface, allowing communication with services running on the same device.

`127.0.0.1` is the loopback IP address assigned to the `localhost`. It is part of the **reserved** IP address block for loopback addresses. When you use `127.0.0.1` directly, you are specifically referring to the loopback interface's IP address. It is the most commonly used loopback address in IPv4.

Since `localhost` is just a hostname, you can change them to be whatever you want. You can do this by editing `/etc/hosts`:

<img src="{{ site.baseurl }}//assets/images/lab4_1-intro/2023-06-08-10-54-42.png"  class="center_full no-invert"/>

And then flush your system's DNS cache to ensure it takes effect:

```
# macOS
sudo dscacheutil -flushcache

# Linux
sudo systemctl restart systemd-resolved
```

### `0.0.0.0` vs `127.0.0.1`

Sometimes you might have read that you can spawn your server on `0.0.0.0` (if you were to deploy it on remote server like aws EC2 and expect it to be publicly reachable). When an IP address is set to "0.0.0.0," it is a special value that represents **all** available network interfaces or all IPv4 addresses on the local machine. It is often used to specify that a service or application should listen on all available network interfaces, meaning it can accept connections from any IP address assigned to the machine.

For example, if a web server is configured to listen on `0.0.0.0:5000`, it will accept incoming connections on all available IP addresses and interfaces of the machine.
{:.info}

This is **different** from when you access `127.0.0.1` or `localhost` on your machine, as you are connecting to services or applications running locally, within the **same** device. In summary, `0.0.0.0` represents **all** available network interfaces and is used for accepting connections from **any** IP address on the machine. `127.0.0.1` is the loopback address used to refer to the local machine itself, enabling communication with services running **locally**.

# The Architecture

The diagram below summarises the architecture of the simple system:

<img src="{{ site.baseurl }}/assets/images/lab4_1-intro/cse-stuffs-htcpcp.drawio.png"  class="center_full"/>

### Task 1

`TASK 1:` **Study** the architecture of the project, then complete all the `TODO` in `./server/server_pot.py`.
{:.task}

Please refer to the [readme](https://github.com/natalieagus/lab_htcpcp/blob/master/README.md) of the project before proceeding to understand further details about the system and answer related questions on eDimension. In particular, you should pay attention to the following:

1. What protocol(s) is/are used to exchange messages between the web application and the web browser?
2. What protocol is used to exchange messages between the web application and the coffee server?
3. Do the browser send messages directly to the coffee server? Why or why not?

There are three `TODO` sections in total to complete. 

#### Create HTCPCP Response Header

```py
    if processing_request and method in ACCEPTED_METHODS:
        current_date = datetime.datetime.now().strftime(
            TIME_STRING_FORMAT
        )

        # TODO: Create response headers 
        headers_to_send = []

        response = create_request_response(
            method, message, additions, pour_milk_start
        )

        final_response = "".join(headers_to_send) + response

        logging.info("Sending response: " + final_response)
```

The HTCPCP response header `headers_to_send` should contain the following <span class="orange-bold">list</span> of string:
1. The **status line**: `HTCPCP/1.1 Status-Code Reason-Phrase`. At this point of our code, our `Status-Code` should be `200`. Find out what `Reason-Phrase` `200` should have. 
2. Subsequent strings: **header fields**. It should have the following header fields: 
   1. `Server` field name with `CoffeePot` as its value 
   2. `Content-Type` field name with the [appropriate media type](https://datatracker.ietf.org/doc/html/rfc2324#section-4) as its value
   3. `Date` field name with `current_date` as its value 
3. Add a line break (`\r\n`) at the end to mark the end of the header and the beginning of the body

Additionally, **each line** in the header should be separated by line break: `\r\n` and each  field name followed by a colon (:) and a leading whitespace.

Here's an example for `HTTP` response message. This is for your **reference** only, adapt the **header fields** for `HTCPCP`: 

```html
HTTP/1.1 200 OK
Content-Type: text/html
Date: Tue, 09 Apr 2024 18:37:30
(empty line)
<html>
<head>
    <title>Example</title>
</head>
<body>
    <p>This is an example of a response body.</p>
</body>
</html>
```

#### Handle Rejected Valid Request 

```py
    elif not processing_request:
        # TODO: Handle other cases that passes ensure_request_is_valid but isn't supported
        # if we reach here, request is valid, but the server doesn't support this feature 
        # e.g: 406
        final_response = ""
```

Construct `final_response` string to complete this `TODO`. It should contain appropriate HTCPCP headers with error code [406](https://datatracker.ietf.org/doc/html/rfc2324#section-2.3.1) because the client requested something unacceptable.  You should write a list of accepted addition as the response body. You can obtain this using: `list(ACCEPTED_ADDITIONS.keys())`. Don't forget to format the response properly like you did in the previous `TODO`.

#### Complete message request checking 

This method is called once at `main()` to ensure that the request format is valid and follows a proper `HTCPCP` protocol.  Complete its implementation. 

If all checks pass, this method should return `True`, otherwise, it should call `send_error_message` which sends the appropriate HTCPCP response back to the client. 

```py
def ensure_request_is_valid(url, content_type, method, connection, requested_pot,
                            accepted_coffee_schemes, accepted_methods, not_found_message):
    # TODO: Basic request checking 
    """
    This method checks if the URL scheme is correct. You shall: 
    
    1. Validate the scheme against accepted_coffee_schemes
    2. Check for correct URL path format <SCHEME>://<HOSTNAME>
    3. Validate the HTTP method: check method against accepted_methods
    4. Check the content type format to conform to "application/coffee-pot-command"
    5. Specific check for "tea" pot request

    If all checks pass, return True, otherwise return False

    For each case 1 to 5 above, call send_error_message(error_message) with an appropriately crafted error message containing status code and reason-phrase. The arg not_found_message gives you a general idea of the format of the expected error message conforming to HTCPCP/1.0 protocol.
    """
    return True
```

{:.note}
Consult `config.py` file for accepted schemes (protocols), hostname, and available pots. The correct URL should anything that conforms to `<SCHEME>://<HOSTNAME>`, where `HOSTNAME` is `ducky`. 

#### Test Invalid Requests

The file `webapp/coffee_app.py` already contains some test routes for you to see if you have implemented `ensure_request_is_valid` properly. Utilize them by checking that your coffee pot server indeed returns the correct statuses given these scenarios.

```py
@app.route("/test-400")
def test_400():
    data = connect_to_server("GET caffeine://ducky HTTP/1.1\r\nContent-Type: application/coffee-pot-command\r\n\r\n")
    status, response = check_response_status(data)
    return craft_error_template(status, response)

@app.route("/test-404")
def test_404():
    data = connect_to_server("GET coffee://psyduck HTTP/1.1\r\nContent-Type: application/coffee-pot-command\r\n\r\n")
    status, response = check_response_status(data)
    return craft_error_template(status, response)

@app.route("/test-501")
def test_501():
    data = connect_to_server("MILK coffee://ducky HTTP/1.1\r\nContent-Type: application/coffee-pot-command\r\n\r\n")
    status, response = check_response_status(data)
    return craft_error_template(status, response)

@app.route("/test-415")
def test_415():
    data = connect_to_server("GET coffee://ducky HTTP/1.1\r\nContent-Type: application/tea-pot-command\r\n\r\n")
    status, response = check_response_status(data)
    return craft_error_template(status, response)
```



{:.note}
You might want to read the rest of the labs first before returning to complete the code, especially this section about [http status code](#http-status-code). 
 

# Inspecting Packets with Wireshark

In the previous lab, you learned how to load and view captured packets with Wireshark. This time around, you will be doing the capturing on your own. Start both `server_pot.py` and `webapp_coffee.py` and open Wireshark.

First, sniff the **loopback** interface:

macOS users need to install `chmodbpf` package before being able to sniff loopback interface. [See here](https://www.wireshark.org/docs/wsug_html_chunked/ChBuildInstallOSXInstall.html).
{: .note}

<img src="{{ site.baseurl }}//assets/images/lab4_2-inspect/2023-06-26-18-00-22.png"  class="center_full no-invert"/>

Then, apply the filter `(tcp.port == 5030) or (tcp.port == 5031)`. Open your web browser and access the homepage of your site `http://127.0.0.1:5031`. You should see some packets captured as follows:

{:.warning}
We assume you have completed Task 1 for this. Otherwise, simply read along and skip to Task 2.  

<img src="{{ site.baseurl }}//assets/images/lab4_2-inspect/2023-06-27-17-44-39.png"  class="center_full no-invert"/>

Notice that there's no `HTCPCP` traffic since Wireshark doesn't have a [dissector](https://wiki.wireshark.org/Lua/Dissectors) for it.

### Task 2

`TASK 2:` Install the `HTPCPCP` dissector for Wireshark.
{:.task}

You can [follow this readme file prepped by your TA](https://github.com/natalieagus/lab_htcpcp/tree/master/dissector). This readme file exists under `dissector/` directory of the project you just cloned for this lab too.

After the dissector is successfully installed, you should see custom `HTCPCP` tag appearing:

<img src="{{ site.baseurl }}//assets/images/lab4_2-inspect/2023-06-27-17-45-12.png"  class="center_full no-invert"/>

You can also filter the traffic by tag `HTCPCP` instead of `port`:

<img src="{{ site.baseurl }}//assets/images/lab4_2-inspect/2023-06-27-17-45-23.png"  class="center_full no-invert"/>

Then, interact with the site: brew some coffee with milk, view your coffee beans, etc to confirm that more `HTCPCP` packets are captured by Wireshark:

<img src="{{ site.baseurl }}//assets/images/lab4_2-inspect/2023-06-27-17-45-37.png"  class="center_full no-invert"/>

Head to eDimension to answer a few questionnaire pertaining to this task.
{:.info}

### Task 3

`TASK 3:` Inspecting transport layer protocol (TCP) and TCP ports used
{:.task}

The file `config/config.py` states the ports used for the webserver and coffee server:

```python
HOST = "0.0.0.0"
LOCALHOST = "localhost"
COFFEE_SERVER_PORT = 5030
WEBSERVER_PORT = 5031
BREW_TIME = 30
ERROR_TEMPLATE = "error.html"
TIME_STRING_FORMAT = "%a, %d %b %Y %H:%M:%S"
```

When we start both processes: the webserver and the coffee server, both servers are binding itself to port `5031` and port `5030` respectively to listen and wait for connection requests. Once a client attempts to `connect` to the socket, a **new** TCP socket is created using 4 identifiers: client IP, client port, server IP and server port.

The web browser is a client to the Flask app (the webserver part), and the Flask app is a client to our coffee webserver. 

Open `sample_capture/homepage_coffee.pcapng` in Wireshark and apply `htcpcp or (tcp.port == 5031)` filter. You will something like this, and use it to answer a few questions on eDimension.


<img src="{{ site.baseurl }}//assets/images/lab4_2-inspect/2023-06-27-17-46-02.png"  class="center_full no-invert"/>

Pay attention to these few things:

1. Can you find SYN, SYN-ACK, and ACK messages? When do we need these handshakes?
2. What are the ports used for communication by the web browser and the Flask app?
3. What are the ports used for communication by the Flask app and the coffee server?
4. How many HTTP response(s) is/are present in the capture? What about request(s)? How can you tell which one(s) is/are the request vs the response?
5. Repeat question 4 but with HTCPCP packets

### Task 4

`TASK 4:` Inspect HTCPCP Messages
{:.task}

Now open `sample_capture/coffee_brew.pcapng`, and add the `htcpcp` filter on it:

<img src="{{ site.baseurl }}//assets/images/lab4_2-inspect/2023-06-27-17-53-51.png"  class="center_full no-invert"/>

We captured these packets when we navigate to our coffee site homepage, brew a coffee, and query the beans currently used to brew the coffee. Inspect its content ans answer a few questions on eDimension. In particular, pay attention to these few things:

1. What are the formats of HTCPCP request and response messages?
2. What are the ports used for communication by the Flask app and the coffee server? Do they remain the same?
3. Read the [HTCPCP RFC](https://datatracker.ietf.org/doc/html/rfc2324), and find the corresponding implementation in `webapp_coffee.py` and `server_pot.py`. There aren't many of them.
4. What are the implemented HTCPCP methods? What headers are accepted on the messages

### Task 5

`TASK 5:` Inspect HTTP Messages
{:.task}

Repeat Task 4, but now you inspect HTTP messages (use `http` filter on `sample_capture/coffee_brew.pcapng`):

<img src="{{ site.baseurl }}//assets/images/lab4_2-inspect/2023-06-27-17-54-34.png"  class="center_full no-invert"/>

In particular, pay attention to these few things:

1. Which packet contains the request made by the browser to get the base HTML page?
2. What are the ports used for communication by the web browser and the Flask app? Do they remain the same?
3. Which HTTP protocol is used? Is the connection persistent? Why and why not?
4. How does the browser ask for more assets?
5. The browser and the Flask app communicates via HTTP, not HTCPCP, but the us (users) wish to **brew coffee** (and indirectly communicate with the coffee server). How do we tell the Flask app the format of the HTCPCP message to be sent to the coffee server? _Hint: see message number 45_

As usual, head to eDimension to answer several questions pertaining to this task.

# HTTP Status Code
In this section, we will explore the concept of status code. In HTTP, status codes are three-digit numbers that indicate the outcome of an HTTP request. Each status code conveys a **specific** **meaning** to help identify and troubleshoot issues during communication between a client and a server.

<img src="{{ site.baseurl }}//assets/images/lab4_3-error/2023-06-27-17-48-11.png"  class="center_fourty no-invert"/>

Our TAs Ryan and Cassie recommend the PG13 version: <https://http.cat> and <https://http.dog> instead.
{: .info}

[HTTP status codes are divided into 5 categories](https://datatracker.ietf.org/doc/html/rfc2616#section-10):

1. `1xx`: Informational, indicates a provisional response consisting only of the Status-Line and optional headers, and is terminated by an empty line
   <img src="https://http.cat/images/100.jpg"  class="center_seventy no-invert"/><br><br>
2. `2xx`: Successful, indicates that the client's request was successfully **received**, **understood**, and **accepted**
   <img src="https://http.dog/201.jpg"  class="center_seventy no-invert"/><br><br>
3. `3xx`: Redirection, indicates that further action needs to be taken by the user agent in order to fulfill the request
   <img src="https://http.dog/303.jpg"  class="center_seventy no-invert"/><br><br>
4. `4xx`: Bad Request, indicates that the server cannot understand the request due to malformed syntax, and client should not resend the request without any modifications
   <img src="https://http.cat/images/429.jpg"  class="center_seventy no-invert"/><br><br>
5. `5xx`: Server Error, indicates cases in which the server is aware that it has erred or is incapable of performing the request
   <img src="https://http.dog/529.jpg"  class="center_seventy no-invert"/><br><br>

When you inspect `coffee_brew.pcapng`, you will see several status code:

<img src="{{ site.baseurl }}//assets/images/lab4_2-inspect/2023-06-27-17-54-34.png"  class="center_full no-invert"/>

### Task 6

`TASK 6:` Study `HTTP` status code 200, 302, 304.
{:.task}

Inspect which request results in those responses with those status code and head to eDimension to answer several questions about it.

# HTCPCP Status Code

[There are two additional HTCPCP status code:](https://datatracker.ietf.org/doc/html/rfc2324#section-2.3)

1. 418: I'm a Teapot
2. 406: Not Acceptable

Both are implemented by our coffee server. Open `sample_capture/coffee_418_406.pcapng` and add `htcpcp` filter on it. In particular, see packet **53** and **129**:

<img src="{{ site.baseurl }}//assets/images/lab4_3-error/2023-06-27-17-57-22.png"  class="center_full no-invert"/>

From the web browser, you can trigger 418 by trying to brew coffee with a teapot:

<img src="{{ site.baseurl }}//assets/images/lab4_3-error/2023-06-27-17-49-11.png"  class="center_fifty no-invert"/>

Which will result in this page being generated by Flask and passed to our web browser:

<img src="{{ site.baseurl }}//assets/images/lab4_3-error/2023-06-27-17-49-27.png"  class="center_full no-invert"/>

You can trigger 406 by trying to brew coffee with **chamomile** option (who would do that? chamomile + caffeine doesn't give you the most pleasant feeling 🤧):

<img src="{{ site.baseurl }}//assets/images/lab4_3-error/2023-06-27-17-59-38.png"  class="center_full no-invert"/>

However we have not implement anything in the webapp to handle that status code 406 (unlike status 418).

### Task 7

`TASK 7:` Confirm handling of `HTCPCP` status code 406, 418, etc (other than 200). You may choose to **embellish** the reason-phrase in the frontend app if you wish.  
{:.task}

Check that you can handle illegal requests properly. For instance, you shall see this page when you attempted to brew coffee with addition of chamomile:

<img src="{{ site.baseurl }}//assets/images/lab4_3-error/2023-06-27-17-50-49.png"  class="center_full no-invert"/>


# HTTPS

Notice that you can inspect the **content** of HTTP messages on Wireshark. This is not desirable since we typically do not want anyone to sniff what we are browsing on the web. HTTPS is a secure way to send data between a web server and a web browser.

## TLS

In HTTPs, the communication protocol is **encrypted** using [Transport Layer Security (TLS)](https://www.cloudflare.com/en-gb/learning/ssl/transport-layer-security-tls/) or, formerly, Secure Sockets Layer. TLS secures communications by using **asymmetric public key infrastructure**, where two keys are used to secure communications between two paraties:

1. **The private key**: this key is controlled by the **owner** of a website and it’s kept private. This key lives on a web server (where the website is hosted) and is used to decrypt information encrypted by the public key.
2. **The public key**: this key is available to everyone who wants to interact with the server in a way that’s secure. As you have already known, information that’s encrypted by the public key can only be decrypted by the private key.

Public keys are typically embedded in a certificate **signed** by a **trusted** Certificate Authority (CA). A certificate is a trusted document that contains a public key and other data of the respective private key owner. Either your browser or your OS **ships** with a set of trusted CA certificates. If a new CA comes into the market, then the browser or the OS will need to ship a software updates **containing** the new CA’s information.

Your browser will typically **complain** if you try to access a site with untrusted Public Key.
{:.error}

### Task 8

`TASK 8:` Enable `HTTPs`
{:.task}

You can enable HTTPs by running `webapp_coffee.py` with the `-https` option:

```
python webapp/webapp_coffee.py -https
```

When you try to access the webpage on your browser, e.g: `https://127.0.0.1:5031`, there might be warning as such:

<img src="{{ site.baseurl }}//assets/images/lab4_4-https/2023-06-28-14-52-39.png"  class="center_seventy no-invert"/>

**Think!** Why does this warning come about? What does it mean by *This server could not prove that it is 127.0.0.1*? 
{:.error}

You can just click **proceed** anyway, and you should be able to view your webpage as per normal. However this time round, when you open Wireshark and sniff your loopback channel while loading the CoffeePot homepage, all packets are **encrypted** and wireshark will not be able to decode `https` packets. It will all appear as `TLS` packets, with encrypted application data:

<img src="{{ site.baseurl }}//assets/images/lab4_4-https/2023-06-28-15-00-29.png"  class="center_full no-invert"/>

With HTTPs enabled, can you still inspect the HTCPCP messages and their content? **Why**? Head to eDimension to answer a few questions pertaining to this task.

### Generate self-signed certificate

Instead of using Flask's adhoc certificate, you can also generate **your own** self-signed certificate (similar to PA2, but PA2 cert is signed by our csesubmitbot). Generate a certificate and a private key using this command.

```sh
openssl req -x509 -newkey rsa:4096 -nodes -out cert.pem -keyout key.pem -days 365
```

You need to have `openssl` installed first. We assume you already have it since it's required for Programming Assignment 2. If you don't have it installed, **google** it.
{: .note}

You will be prompted to key in some details as such, fill it with whatever you want:
<img src="{{ site.baseurl }}/assets/images/lab4_4-https/2023-07-11-17-21-05.png"  class="center_fifty no-invert"/>

You will have two more files in the root folder of the repository: `cert.pem` and `key.pem`. Simply restart the webapp using `-custom` option:

```
python webapp/webapp_coffee.py -https -custom
```

When you inspect the certificate on your browser, you will notice that it will show the details you keyed in earlier:
<img src="{{ site.baseurl }}/assets/images/lab4_4-https/2023-07-11-17-21-38.png"  class="center_fifty no-invert"/>

This is different from the certificates generated by Flask when you use `adhoc` method instead:

<img src="{{ site.baseurl }}/assets/images/lab4_4-https/2023-07-11-17-20-04.png"  class="center_fifty no-invert"/>
## QUIC

Modern browsers at the time of this writing (Jan 2025) have moved to utilise [QUIC](https://www.chromium.org/quic/): a multiplexed transport over UDP. Its goal is to improve user experience by reducing page load times, and it started as an alternative to TCP+TLS+HTTP/2.

<img src="{{ site.baseurl }}//assets/images/lab4_4-https/2023-06-28-15-18-30.png"  class="center_full no-invert"/>

**QUIC** is out of syllabus, but we figured it would be fun to know.
{:.info}

## Summary

In this lab, you have learned several things pertaining to our syllabus:

1. Understand client-server model of network applications
2. Understand how web server works and the HTTP requests and responses
3. Understand how wireshark packet sniffer and analyzer works and its application

{:.note}
It might do you some good to try and draw the space-time diagram of `sample_capture/homepage_coffee.pcapng` as practice. Although it is not formally graded, it might give you a better understanding on TCP handshake and the timeline of HTTP messages exchanged between server/client.
