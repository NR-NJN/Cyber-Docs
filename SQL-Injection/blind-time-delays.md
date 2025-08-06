# Blind SQL injection with time delays and information retrieval

**Vulnerability:** SQL injection in the cookie tracker ID

---

### Objective 1
The goal of this lab is to create a 10 second delay on the website using a time delay injection
### Objective 2
The goal of this lab is to make use of the blind SQL injection vulnerability present in the cookie ID of the website, and use this to inject a query to retrieve all the password of the `administrator` and then login.

---

### Methodology 1
This was part 1 of the lab which only required you to perform a time delay

#### 1. Exploitation

This was short and simple, my first guess of the website using PostgreSQL was correct and one command was enough to trigger the delay
```
' ||pg_sleep(10)--
```

This ensured the lab part 1 was completed.


### Methodology

#### 1. Reconnaissance
In the cookie section I had to make sure the database was still the same. However, the unconditional queries weren't working therefore I had to go with a conditional approach
```
'||(SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END)--
```
This worked perfectly and hence I can use this to build a full query

Trying another query to make sure the tables and columns were correct

```
'||(SELECT CASE WHEN (username = 'administrator' AND LENGTH(password)>1) THEN pg_sleep(10) ELSE pg_sleep(0) END from users)--
```

#### 2. Discovery & Exploitation

*   **Step 1:** Now since the tables and columns were accurately giving me a 10 second delay, I had to construct a better query for the password
```
'||(SELECT CASE WHEN (username = 'administrator' AND SUBSTRING(password,1,1)<'m') THEN pg_sleep(10) ELSE pg_sleep(0) END from users)--
```
Sure enough this took 10 seconds to parse and the approach was becoming more clearer
*   **Step 2:** Unlike the previous labs this time I was going to send it to the intruder regardless of the long delays and parsing time as it was going to be tedious for me to do this manually

```
||(SELECT CASE WHEN (username = 'administrator' AND SUBSTRING(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END from users)--
```
The query had to be modified to this to make sure the cluster bomb iterated through all possible outcomes

*   **Step 3:** I sent it to the intruder and added payloads to the second argument of the substring function to iterate 20 times and another payload on the comparison character to iterate through a-z and 0-9. Hopefully it should not take as long as it previously did. Usually the first 60 are instanteous and then it exponentially slows down afterwards.

`'||(SELECT CASE WHEN (username='administrator' AND LENGTH(password)=20) THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users)--`
This was used to find out the length of the password

Before starting the attack I changed the response time to 5 seconds because that was more than enough compared to the other ms time values and would significantly reduce the time taken for illiciting a response. 

(So if you just leave it as is, it stops functioning in the background and then takes exponentially longer to run)

The intruder met some errors around the late stage of brute forcing so I had to rerun it and sit there as it ran. After some more time (not quite as much as before), I reordered the output table to sort by request time, and selected every request above 5000 ms

#### 3. Result
I was able to receive the password from the brute force table and align them based on the payload numbers

---


