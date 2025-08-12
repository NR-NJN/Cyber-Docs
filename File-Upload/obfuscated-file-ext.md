# Web shell upload via obfuscated file extension

**Vulnerability:** Vulnerability in the upload avatar function of the user account

---

### Objective
The goal of this lab upload shell code that can access the website's filesystem and print the file contents present in `/home/carlos/secret` from the image upload button. Send this text to the solution box in the same website

---

### Methodology

#### 1. Reconnaissance
First we have to login as the specified user password. Upload a regular image, and return back to your account, track this `GET` request to the repeater and keep it there, note the file path by opening the image in a new link 

#### 2. Discovery & Exploitation

*   **Step 1:** 
Upload the same shellcode which worked on the previous labs and send the `POST` request to the repeater
```
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
Since this file is not allowed, you can check the `POST` request and in the `filetype` section use different encoding types to check if it accepts anything. I added `.php.jpg` to the extension and it was accepted. However, the response still accepted it as `shell.php.jpg` which is not what we need.
*   **Step 2:**  
In the same filter, add an encoded NULL terminating character. Which means the path should look lie this 
```
shell.php%00.jpg
```
Once sent, the request should interpret it as `shell.php`, as the remaining characters are stripped

*   **Step 3:** 
Go back to the `GET` request and add the shellcode file to the path instead of the image file. Send this request to spit out the submission code




#### 3. Result
Once you get the text, just feed it to the solution tab on the website. 




---


