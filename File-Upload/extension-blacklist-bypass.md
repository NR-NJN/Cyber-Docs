# Web shell upload via extension blacklist bypass

**Vulnerability:** Vulnerability in the upload avatar function of the user account

---

### Objective
The goal of this lab upload shell code that can access the website's filesystem and print the file contents present in `/home/carlos/secret` from the image upload button. Send this text to the solution box in the same website

---

### Methodology

#### 1. Reconnaissance
First we have to login as the specified user password. Upload a regular image and return back to your account, track this `GET` request to the repeater and keep it there. If you upload a php script trying to access `/home/carlos/secret`, it says your're not allowed to do this. 

#### 2. Discovery & Exploitation

*   **Step 1:** 
Upload the same shellcode which worked on the previous labs and send the `POST` request to the repeater
```
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
Since this file is not allowed, you can check the `POST` request and then see the server it is being uploaded to. It most likely will be an Apache server, and hence you can upload a configuration file to make it think php files are some other non blacklisted files.
*   **Step 2:**  
Change the filetype to `.htaccess` this file is accepted as there is no extension to it. Change the `Content-Type` to `text/plain`. Under the contents of the file, add this directive to modify the configuration
```
AddType application/x-httpd-php .jpeg
```
Or any extension that wont be blocked by by the server configuration. (could be literally any word too). Send the request to the server.

*   **Step 3:** 
Go back to the webpage and upload the same shellcode, and catch the `POST` request again. This time in the request change the extension from php to jpeg or whatever extension name you gave in the `.htaccess` file. Send this request which should now be accepted by the server. Open the original `GET` request and then change the file path to the shellcode's filepath taken from the website (files/avatars/shell.jpeg). Send this request and it should spit out the secret file contents.




#### 3. Result
Once you get the text, just feed it to the solution tab on the website. 




---


