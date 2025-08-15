# Remote code execution via polyglot web shell upload

**Vulnerability:** Vulnerability in the upload avatar function of the user account

---

### Objective
The goal of this lab upload shell code that can access the website's filesystem and print the file contents present in `/home/carlos/secret` from the image upload button. Send this text to the solution box in the same website

---

### Methodology

#### 1. Reconnaissance
First we have to login as the specified user password. Upload a regular shell file with the same code as previously mentioned and then see as the website doesn't accept it, even after obfuscating the name. This website looks at the metadata and content of the file itself to match a set of predefined types, which is more often than not, jpg or png file data. The php script hiding in your file is not fooling anybody now.

#### 2. Discovery & Exploitation

*   **Step 1:** 
Download exiftools on your specified device. and run a command which is able to change the file contents of a php script to replicate an already existing image file.
```
exiftool -Comment="<?php echo 'START' .  file_get_contents('/home/carlos/secret') .  'END'; ?>" <image name>.<image file extension> -o <output file name>.php

```
This is able to inject the php line script into the image's metadata, whilst maintaining the properties of a malicious php file.
*   **Step 2:**  
The server verifies the file contents to see image data, and lets it pass through the barrier. The code is then run by the server as the filetype is still php which the server is configured to run at this level.

Return back to the profile page after uploading this php file from your device, and capture the `GET` request on the repeater. Check the file location and modify it in the repeater before sending the request again

```
/files/avatars/<filename>.php
```
This should spit out the image file data and the content of the `secret` file between the `START` and `END` text blocks as entered into the line. Everything in between those 2 words is the solution text.


#### 3. Result
Once you get the text, just feed it to the solution tab on the website. 




---


