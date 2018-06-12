# Giải nén dữ liệu nhị phân trong PHP

Làm việc với các file nhị phân trong PetaHP hiếm khi được yêu cầu. Tuy nhiên khi cần thiết các function 'pack' và 'unpack' của PHP có thể hỗ trợ bạn rất nhiều. Để thiết lập bước này chúng ta sẽ bắt đầu với một vấn đề về lập trình, điều này sẽ giữ cho bài viết được nhất quán liền mạch về ngữ cảnh của bài viết. Vấn đề đó là: Chúng ta muốn viết một function đầu vào là một file ảnh và nó sẽ cho chúng ta biết liệu rằng file đó có phải là ảnh GIF hay không; không liên quan với bất kỳ phần mở rộng nào mà f có thể cóile. Chúng ta không được sử dụng bất kì function nào của thư viện GD.

### Một header của file GIF

Với yêu cầu rằng không được sử dụng bất kì function nào trong thư viện đồ họa, để giải quyết vấn đề này ta cần lấy những dữ liệu liên quan từ file GIF. Không giống như HTML hay XML hoặc các loại file text khác, một file GIF và hầy hết các định dạng ảnh khác đều được lưu trữ dữ dạng nhị phân. Hầu hết các file nhị phân chứa header ở trên đầu file cung cấp các thông tin meta về file cụ thể. Chúng ta có thể sử dụng những thông tin này để tìm ra loại file và các thứ khác, ví dụ nhưng chiều cao, chiều rộng trong trường hợp của file GIF. Một header của file GIF tiêu biểu được trình bày ở duwois đây, sử dụng bộ sửa hex như [WinHex][1].
![][2]

Mô tả chi tiết về header được đưa ra ở dưới đây.

| ----- |
| 
    
    
    Offset   Length   Contents
      0      3 bytes  "GIF"
      3      3 bytes  "87a" or "89a"
      6      2 bytes  
      8      2 bytes  
     10      1 byte   bit 0:    Global Color Table Flag (GCTF)
                      bit 1..3: Color Resolution
                      bit 4:    Sort Flag to Global Color Table
                      bit 5..7: Size of Global Color Table: 2^(1+n)
     11      1 byte   
     12      1 byte   
     13      ? bytes  
             ? bytes  
             1 bytes   (0x3b)

 | 
 
Do đó để kiểm tra liệu rằng một file ảnh có phải là ảnh GIF hay không, chúng ta cần phải kiểm tra 3 byte bắt đầu của header, nơi có các dấu hiệu của 'GIF', và 3 byte tiếp theo chưa các phiên bản; hoặc '87a' hoặc '89a'. Đó là cho các tác vụ như trên cái mà chứa function unpack (). Trước khi chúng ta xem xét giải pháp, xem nhanh hàm unpack ().

### Sử dụng function unpack()

[unpack()][3] là sự hoàn thiện của [pack()][4] - nó vận chuyển dữ liệu nhị phân đến với mảng liên kết dựa trên định dạng đã được chỉ định. Nó là một thứ gì đó xuyên suốt ở các dòng _sprintf_, vận chuyển các chuỗi dữ liệu theo nhưng định đạng cho trước. 2 function này cho phép chúng ta đọc và ghi các buffer của dữ liệu nhị phân theo như định dạng đã chỉ định. Điều này dễ dàng cho phép một lập trình viên chuyển đổi dữ liệu với chương trình đã được viết ở một ngôn ngữ khách hoặc định dạng khác. Một ví dụ nhỏ cho việc này.

| ----- |
| 
   
    
    $data = unpack('C*', 'codediesel');
    var_dump($data);

 | 

Nó sẽ in ra màn hình như dưới đây, mã thập phân 'codediesel' :

| ----- |
| 
    
    
    array
      1 => int 99
      2 => int 111
      3 => int 100
      4 => int 101
      5 => int 100
      6 => int 105
      7 => int 101
      8 => int 115
      9 => int 101
      10 => int 108

 |
 
