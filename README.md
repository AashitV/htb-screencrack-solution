<p align="center">
  <a href="https://laravel.com" target="_blank">
    <img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel Logo">
  </a>
</p>

<h1 align="center">ScreenCrack – Laravel SSRF to Redis RCE</h1>
<p align="center">
  A Hack The Box walkthrough and exploit project<br>
  SSRF ➜ Redis ➜ Encrypted Laravel Job ➜ Unserialization ➜ Remote Code Execution
</p>

---

##  About This Project

This repository contains Team 7's walkthrough and proof-of-concept exploit for the Hack The Box machine **ScreenCrack**, which simulates a vulnerable Laravel-based web application.

**ScreenCrack** is a web challenge with Medium difficulty, which functions by taking url as an input and providing the page's Screenshot and Source code.

We explore a full **SSRF-to-RCE** exploit chain by injecting malicious job payloads into Redis via gopher protocol, triggering RCE through Laravel’s job queue mechanism.

 **This project is for educational purposes only.**

---

## Attack Chain Overview (flow)

``` 
  A[User Input Field] --> B[cURL Fetch (SSRF)]
  B --> C[gopher://127.0.0.1:6379] // Redis is on port 6379
  C --> D[Redis Queue]
  D --> E[Laravel Worker]
  E --> F[Unserialize & Execute Payload]
  F --> G[Flag written to web directory]

```
---
## SSRF

-> Connect to the Redis DB via url. HINT: "http:// 127.0.0.1 : <Redis_port>"

-> To make it work, use different protocol other than "http", and would need to bypass "127.0.0.1".

-> Here's the number of bypasses available for the same: 
https://github.com/b4rdia/HackTricks/blob/master/pentesting-web/ssrf-server-side-request-forgery/url-format-bypass.md

-> This is the way to interact with the Server Database and inject the payload through the input field.

---

## Payload generation

**Step 1:** 
We can see default job queues in Redis DB

![1) default keys - job queues](https://github.com/user-attachments/assets/b88c97cd-886e-4404-a6f2-444f73530e75)


**Step2:** 
Through the local Docker instance, we can see the actual job in the default queue (laravel_database_queues:default)

![2) actual job format](https://github.com/user-attachments/assets/fdd513c6-f298-47e0-9986-2af446c9f517)


**Step 3:** 
It becomes clear after proper indents and formatting, and easier to exploit now.

![3) simplified format (json)](https://github.com/user-attachments/assets/d3244c0b-719a-40c7-b99c-8f3ed011a122)


**Step 4:** 
You may need to convert it back to the format actually being executed (Serialize back a specific part properly).


**Step 5:** 
Ultimately, encode the payload with the Redis push command to enter the job inside the default queue based on the **protocol** used.


![submitted and pwned ScreenCrack](https://github.com/user-attachments/assets/3df0deb3-d257-429b-afc4-901d7c73f72d)


------------------------------------------------------------------------------------------------------------------------------------

NOTE: 

~ May have to wait 10 min for the job to be executed because of the design in code (sleep=600).

~ A specific protocol will show error when entered, but it runs and gets executed in the backend and get the job done.

~ Local docker instance will help to see how the queue and job actually works. (Here, we can execute the jobs faster and won't have to wait for 10 min, as we have access to the shell directly to run the command: 'php artisan queue:work --once').

~ You will get flag based on the command you used inside the payload. (Hint based on our payload: By default, the files URL output files are inside 'src' folder = <HTB ip : port>/src/abcdef.txt)

---

## Status
This project is a proof of concept and is **just a hint** and actual RCE payload with push Redis command is kept **hidden** due of HTB policy of not sharing flag/writeup for **Active challenges**.


~by

Cristiansen, Matt

Nguyen, Dinh Cao Dat

Virani, Aashit
