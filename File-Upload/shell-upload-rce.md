# Remote code execution via web shell upload

**Vulnerability:** Vulnerability in the upload avatar function of the user account

---

### Objective
The goal of this lab upload shell code that can access the website's filesystem and print the file contents present in `/home/carlos/secret` from the image upload button. Send this text to the solution box in the same website

---

### Methodology

#### 1. Reconnaissance
First we have to login as the specified user password. Then upload an arbitrary image to gain access to the GET and POST requests made by the website. Send these requests to the Burp Repeater. When you upload an image, you will see the file path to which the image is uploaded. 

#### 2. Discovery & Exploitation

*   **Step 1:** 
The file path found on the website is where we can inject the shellcode. This can either be done manually by uploading a php script onto the avatar upload button, or by erasing the image data completely in the POST request on Burp's repeater, and replacing it with the shell php filename and shellcode. In the link/GET request you must change the file path to match the shellcode's filepath. This can be viewed by right click inspecting the image avatar after it has been uploaded and you're prompted to return to the logged in page.
*   **Step 2:**  
This shellcode may look like this
```
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

or this
```
<?php echo system($_GET['cmd']); ?>     
```

The first one runs the command directly accessing the required files, while the second one allows the user to dynamically input commands in the website link or the GET request.

*   **Step 3:** 
If you upload it directly to the website, you will get the output printed on the webpage directly. With the `$_GET` method, you will have to appent `?cmd=<command>` to the link after the shellcode's uploaded filepath. The command could then just be `cat /home/carlos/secret`

If you inspect the Burp Repeater, then you would have to change the file path in the GET request to the the same file path as the image upload file path with the php filename. First send the POST request which contains the php code, then send the GET request which gives you the contents of the file as part of the original request. 


#### 3. Result
Once you get the text, just feed it to the solution tab on the website. This is no different from a regular CTF payload upload


---

### Post Script
The `$_GET` method gave me a similar flag text as the real flag/file content. The `cat /home/carlos/secret` spat out something else which looked very similar to the real answer. However, when I fed the answer to the solution text box, it wasn't accepting it. There were no whitespaces or copying problems. So be cautious while using that method.

