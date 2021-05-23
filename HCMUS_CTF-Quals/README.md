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
Và phần `path` được render lại, theo kinh nghiệm của mình có thể đây là lỗi Server-side template injection nên test thử và chính xác là như thế.  
  
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
  
## `GITchee-gitchee-goo`   
