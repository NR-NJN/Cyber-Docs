# Blind SQL injection with conditional statements

**Vulnerability:** SQL injection in the cookie tracker ID

---

### Objective
The goal of this lab is to make use of the blind SQL injection vulnerability present in the cookie ID of the website, and use this to inject a query to retrieve all the password of the `administrator` and then login.

---

### Methodology

#### 1. Reconnaissance
My initial step was to check the cookie ID section and feed it a boolean statement resulting in `TRUE` to retrieve a `Welcome back!` message. This was to confirm if I could control the cookie message through my command injection, and sure enough I could.

#### 2. Discovery & Exploitation
I opened up a link on the website and intercepted the request using BurpSuite to send it to the repeater.

*   **Step 1:** Since I had to query each substring of the password using a series of boolean ouputs, I needed to find out the database type and version used. 
Thankfully, my first guess ended up being correct as I used this query to get a `TRUE` response (Welcome back) on the PostgreSQL database.

```
' AND (SELECT SUBSTRING(version(),1,10))='PostgreSQL'--
```

*   **Step 2:** The point was to brute force the password using boolean outputs and finding out if each character in the password matched the one sent by my query. However this can be a long process even with a binary search approach. So I decided to find out the length of the password using a similar tactic. The command `' AND (SELECT 'a' FROM users WHERE username = 'administrator' AND LENGTH(password)=20) ='a'--`. This was the one that returned `TRUE`.
*   **Step 3:** This is where I make use of Burpe's Intruder functionality. I sent an SQL command that compares each character of the password with each letter of the alphabet and numbers from 0 to 9. This was the command
```
' AND SUBSTRING((SELECT password FROM users WHERE username = 'administrator'), 1, 1) ='r'--
```
I added payloads to the positions `'administrator'), 1,` and the comparison character `='r'`, so it can compare each character from the alphanumeric set to each of the 20 password characters. The first payload is a `Numbers` payload type from 1 to 20, the second payload is a `Simple List` with each character and number added to the list. 

Finally in the settings tab, under the `Grep-Match` section I added the string `Welcome back!` to confirm if the payload returns `TRUE` or `FALSE`, as this is what the website displays on `TRUE`

#### 3. Result
This took an enormous amount of time to brute force all 720 combinations, and I'm sure I could have made it way quicker by using a faster method filter on Burp, however I was able to build a password using the characters, sorting by all of the hits on the `TRUE` signal which puts them all at the top of the table, and then sorting by `PAYLOAD 1` which shows the postions.
I was able to login as admin using the extracted password



---


