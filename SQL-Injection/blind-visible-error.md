# Visible error-based SQL injection

**Vulnerability:** SQL injection in the cookie tracker ID

---

### Objective
The goal of this lab is to make use of the blind SQL injection vulnerability present in the cookie ID of the website, and use this to inject a query to retrieve all the password of the `administrator` and then login.

---

### Methodology

#### 1. Reconnaissance
I went to check if the cookie section was still vulnerable, using a `ORDER BY 1--` query, and sure enough it returned a `200 OK` which confirmed the injection point.

#### 2. Discovery & Exploitation
I opened up a link on the website and intercepted the request using BurpSuite to send it to the repeater.

*   **Step 1:** Since this was a visible error based lab, I used the type cast query to draw out a type error using the username column from the users table

```
' cast((select username from users) AS int)-- 
```
which returned an error

```
Unterminated string literal started at position 95 in SQL SELECT * FROM tracking WHERE id = 'xyz' cast((select username from users) AS int)'. Expected  char
```
this was exactly what I needed from the lab, however I was unsure of what to do after this for quite sometime.

*   **Step 2:** I tried other variations of this query such as `username || ':' || password` however this also returned the same error as the function only returns one value not the whole table.

Because of this issue, I could not test out UNION too as that would output a full column and it should match too. Hence I went with the AND clause.

This query `' AND CAST((SELECT 2) AS int)--` returns a boolean expression error, confirming that it is parsed by the database
which means this should not return any error at all
```
 AND 1=CAST((SELECT 1) AS int)--
```

*   **Step 3:** The lingering issue was to only print out the correct value from the users table. The query also had a character limit, so it wouldnt parse the provided query beyond a certain length. Which caused multiple issues. The offset flag used along with the limit flag made it so the query was too long. This was the current query
```
' AND 1=CAST((SELECT username FROM users limit 1) AS int)--
```
This query did work, however before that I had to delete the cookie ID to free up character space, which was something I thought would never work, as I expected the page to stop loading after that.

This showed me a type error but also mentioned the character it was comparing types with, which happened to be `administrator`. Therefore the first user in the table was `administrator`, and the password should correspond to the same user

```
' AND 1=CAST((SELECT password FROM users limit 1) AS int)--
```

#### 3. Result
I was able to receive the password from the single value type error vulnerability, however having to delete the cookie ID to free up space was not something I would have worked

---


