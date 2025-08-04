# Blind SQL injection with conditional errors

**Vulnerability:** SQL injection in the cookie tracker ID

---

### Objective
The goal of this lab is to make use of the blind SQL injection vulnerability present in the cookie ID of the website, and use this to inject a query to retrieve all the password of the `administrator` and then login.

---

### Methodology

#### 1. Reconnaissance
I opened up a link on the website and intercepted the request using BurpSuite to send it to the repeater.

My initial step was to check the cookie ID section and feed it a case statement which would return a `200 OK` based on the output.
```
' and (select case when (1=2) then 1/0 else 'a' end)='a--
```
this should evaluate to false within the bracket and move to the else statement evaluating to true, since `a=a`

whereas this should cause the website to perform a divide by 0 error as the bracket evaluates to true
```
' and (select case when (1=1) then 1/0 else 'a' end)='a--
```
the first one did not give me a `200 OK` so I had to modify the query. Oracle checks a divide by 0 error before executing the `CASE` query
and hence you need to convert it into a type conversion error.

```
' and (select case when (1=2) then TO_CHAR(1/0) else 'a' end from dual)='a--
```
this was all I needed to begin
#### 2. Discovery & Exploitation

*   **Step 1:** Now that I knew it was an Oracle SQL database, hence I used this query to get into the database and use its `users` table for login details
```
' and (select case when (LENGTH(password)=20) then TO_CHAR(1/0) else NULL end from users where username ='administrator') IS NULL-- 
```
This evaluated to `500 ERROR` which means this is true, and to make sure the command works, I tried it with other character lengths to make sure I get `200 OK`, hence now we know the length of the password is 20 characters


*   **Step 2:** Next I tried out this query to make sure the same works. 
```
' and (select case when (SUBSTR(password,1,1)>'m') then TO_CHAR(1/0) else NULL end from users where username ='administrator') IS NULL--
```
This evaluated to `200 OK` which means this is false, however the point being that this type of query is working.

*   **Step 3:** This is where I make use of Burpe's Intruder functionality. I sent an SQL command that compares each character of the password with each letter of the alphabet and numbers from 0 to 9. The command used was the same as before
```
' and (select case when (SUBSTR(password,1,1)>'m') then TO_CHAR(1/0) else NULL end from users where username ='administrator') IS NULL--
```
I added payloads to the positions `'administrator'), 1,` and the comparison character `='m'`, so it can compare each character from the alphanumeric set to each of the 20 password characters. The first payload is a `Numbers` payload type from 1 to 20, the second payload is a `Simple List` with each character and number added to the list. 

Finally in the settings tab, under the `Grep-Match` section I added the string `Welcome back!` to confirm if the payload returns `TRUE` or `FALSE`, as this is what the website displays on `TRUE`

Now in the previous lab I used this method and it took a couple hours for Burp to spit out the password, hence instead I just manually found out the password by running this command for every character and using Binary Search tactics as I searched for every condition, which took me a little over ten minutes for 20 characters.

#### 3. Result
Since I used binary search while manually finding matching these characters it hardly took any amount of time, I was able to build a password using the characters one by one and then enter them into the website's login portal.
I was able to login as admin using the extracted password


---


