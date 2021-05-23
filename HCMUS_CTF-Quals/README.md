# HCMUS_CTF - Quals
<hr />
  
| Challenge name    | Points  | Solvers   |  
| :---------  |    :---: |   ---: |
| [Nothingness](#nothingness) | 32 | 27 |
| [EasyLogin](#easylogin) | 71 | 24 |
| [SimpleCalculator](#simplecalculator) | 94 | 12|
| [GITchee-gitchee-goo](#gitchee-gitchee-goo) | 193 | 9 |  
  
## `Nothingness`  
    
![](./imgs/nothingness/1.png)      
Trang web khi vào sẽ hiển thị ra như trên, nhưng có vẻ là `URL path` được nhắc đến ở đây nên ta thử nhập một `path` bất kỳ nào đấy lên URL để xem như thế nào  
  
![](./imgs/nothingness/2.png)    
Và phần `path` được render lại, theo kinh nghiệm của mình có thể đây là lỗi `Server-side template injection` nên test thử và chính xác là như thế.  
  
![](./imgs/nothingness/3.png)  
Ở đây ta có thể thấy ở phần `response header` là webserver đang sử dụng `Python` để chạy => Rất có thể là `Jinja2`.   
![](./imgs/nothingness/4.png)  
Sau khi đã xác định được `target` và cũng như `vulnerability` thì ta tiến hành tìm cách để đọc `flag`.  
Thử fuzz với template `{{config.items()}}` để đọc được các giá trị cấu hình trên server và rất có thể là flag nằm trong đó.  

![](./imgs/nothingness/5.png)  
Và tất nhiên là đời không như là mơ, giờ tìm cách `RCE` để tìm ra file flag.  
Payload: `{{config.__class__.__init__.__globals__['os'].popen('<command>').read()}}`  
  
Và `flag` nằm ở thư mục root  
![](./imgs/nothingness/6.png)  
  
Đọc flag nà!!  
![](./imgs/nothingness/7.png)  
  
> Flag: HCMUS-CTF{404_teMpl4t3_1njEctIon    
   
## `EasyLogin`   
  
  
## `SimpleCalculator`   
    
Web có chức năng cho ta nhập vào một biểu thức gì gì đó, sau đó tính toán các kiểu rồi trả về result thông qua biến query `equation`. Ta thử nhập vào một mảng xem như thế nào 
  
![](./imgs/simplecalc/1.png)  
Theo như reponse trả về, ta biết được code đằng sau sử dụng hàm eval để thực hiện tính biểu thức đó. Vậy giờ việc cần làm là tìm cách `RCE` thông qua chức năng này!!!  
Thử thực thi hàm `phpinfo()` xem như thế nào  
  
![](./imgs/simplecalc/2.png)   
Ô cê! i'm fine =((. Và tất cả `ký tự chữ cái [a-zA-Z]` và các dấu như ```quote('), double-quote("), backtick(`)``` đều được filter kỹ càng! Hmmm... Liền thử ngay kỹ thuật `XOR string` để bypass filter nhưng lại bị `giới hạn về ký tự (chỉ 19 ký tự)` nhưng theo kiến thức mình biết được thì ta có thể bypass bằng cách sử dụng dấu `~` để lấy phủ định của một chuỗi.  
  
Ví dụ: `~"_GET"` sẽ cho ra các ký tự không nằm trong alphabet nên khi gửi lên server chỉ cần lấy phủ định lại của kết quả đó là có thể bypass được filter. Ngoài ra việc gọi tên một biến theo cách truyền thống là `$variable` thì ta cũng có thể gọi `${'variable'}`.   
  
Payload mà đội mình dùng để đọc file flag: `?equation=${~%A0%B8%BA%AB}[0](~%91%93%DF%D0%D5)&0=system`  
Giải thích sơ qua về payload:
- `${~%A0%B8%BA%AB}[0] = ${'_GET'}[0]` nghĩa là lấy tên hàm qua biến query `0`.
- `~%91%93%DF%D0%D5 = nl /*` là argument đặt trong function trên và thay cho `cat /*`  

![](./imgs/simplecalc/3.png)  
  
> Flag: `HCMUS-CTF{d4ngErous_eVal}`  
## `GITchee-gitchee-goo`   
