# SQL injection with filter bypass via XML encoding

**Vulnerability:** SQL injection in stock check feature of a product

---

### Objective
The goal of this lab is to make use of the SQL injection vulnerability present in the stock check feature of the website, and use this to inject a query to retrieve all the usernames and passwords into a single column and then login as the admin. However, the website has blacklisted a few attack words in its WAF, hence we must encode the command's characters to bypass the filter.

---

### Methodology

#### 1. Reconnaissance
I went to the stock check feature in Burp and entered a malicious request, which was blocked by the website and I got a `403 Forbidden` proving that the filter was indeed at play here.
The most obvious solution to parse the query was to URL encode common blocklisted words such as `UNION` and `SELECT`

#### 2. Discovery & Exploitation

*   **Step 1:** This command was accepted by the webpage, wherein I got a `200 OK` response with no viable output
`%75nion %73elect * from users` 
The s and the u were URL encoded to confuse the WAF.
*   **Step 2:** Next I decided to find out how many columns were present in the output. `%75nion %73elect null`, this just returns `200 OK`. I thought it was a blind SQL injection and hence decided to try using other tactics such as time stalls. However all of them caused the same exact output.
With nothing panning out, I decided to search up encoding techniques for such a scenario and came across the Hackvector extension that auto url encodes all of my commands, and I can just type them as is inside the tags

*   **Step 3:** I enabled and downloaded the extension, and then applied it to my text without any manual encoding
`UNION SELECT NULL`, this just provided an out put with `null` confirming that there is one row being displayed. 
After this I confirmed the users from the table using `UNION SELECT username from users` and then once I found out the `administrator` is the first user, I used `UNION SELECT password from users limit 1` to retrieve the password.


#### 3. Result
The application successfully logged me in as the `administrator` user, which was the main task. This confirmed the presence of a critical SQL injection vulnerability.



---


