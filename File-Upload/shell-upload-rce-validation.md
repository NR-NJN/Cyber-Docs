# Web shell upload via path traversal

**Vulnerability:** Vulnerability in the upload avatar function of the user account

---

### Objective
The goal of this lab upload shell code that can access the website's filesystem and print the file contents present in `/home/carlos/secret` from the image upload button. Send this text to the solution box in the same website

---

### Methodology

#### 1. Reconnaissance
First we have to login as the specified user password. Then upload an arbitrary image to gain access to the GET and POST requests made by the website. Send these requests to the Burp Repeater. When you upload an image, you will see the file path to which the image is uploaded. 

#### 2. Discovery & Exploitation

*   **Step 1:** The file path found on the website is where we can inject the shellcode. This can be done manually by uploading a php script onto the avatar upload button. In the link/GET request you must change the file path to match the shellcode's filepath. This can be viewed by right click inspecting the image avatar after it has been uploaded and you're prompted to return to the logged in page.
*   **Step 2:**  
This shellcode may look like this
```
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

The issue, is that when you upload this php script, it gives you an error telling you to upload jpeg or png files. This is the request you send to the repeater, marked as a `POST` request.

*   **Step 3:** 
Here the `Content-Type` of the file must be changed to `image/jpeg` or png to trick the server into accepting the php shellcode. Send this request and then find the `GET` request from the HTTP history of the proxy. This should now have the contents of the secret file.




#### 3. Result
Once you get the text, just feed it to the solution tab on the website.


---


