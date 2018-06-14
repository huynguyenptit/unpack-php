# Giải nén dữ liệu nhị phân trong PHP

Làm việc với các file nhị phân trong PHP hiếm khi được yêu cầu. Tuy nhiên khi cần thiết các hàm 'pack' và 'unpack' của PHP có thể hỗ trợ bạn rất nhiều. Để thiết lập bước này chúng ta sẽ bắt đầu với một vấn đề về lập trình, điều này sẽ giữ cho bài viết được nhất quán liền mạch về ngữ cảnh của bài viết. Vấn đề đó là: Chúng ta muốn viết một hàm đầu vào là một file ảnh và nó sẽ cho chúng ta biết liệu rằng file đó có phải là ảnh GIF hay không; không liên quan với bất kỳ phần mở rộng nào mà file có thể có. Chúng ta không được sử dụng bất kì function nào của thư viện GD.

### Một header của file GIF

Với yêu cầu rằng không được sử dụng bất kì function nào trong thư viện đồ họa, để giải quyết vấn đề này ta cần lấy những dữ liệu liên quan từ file GIF. Không giống như HTML hay XML hoặc các loại file text khác, một file GIF và hầy hết các định dạng ảnh khác đều được lưu trữ dữ dạng nhị phân. Hầu hết các file nhị phân chứa header ở trên đầu file cung cấp các thông tin meta về file cụ thể. Chúng ta có thể sử dụng những thông tin này để tìm ra loại file và các thứ khác, giống như chiều cao, chiều rộng trong một file GIF. Một header của file GIF tiêu biểu được trình bày ở duwois đây, sử dụng bộ sửa hex như [WinHex][1].
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
 
Do đó để kiểm tra liệu rằng một file ảnh có phải là ảnh GIF hay không, chúng ta cần phải kiểm tra 3 byte bắt đầu của header, nơi có các dấu hiệu của 'GIF', và 3 byte tiếp theo chưa các phiên bản; hoặc '87a' hoặc '89a'. Đó là các tác vụ như trên mà  function unpack () không thể thiếu. Trước khi chúng ta xem xét giải pháp, xem nhanh hàm unpack ().

### Sử dụng function unpack()

[unpack()][3] là sự bổ sung của [pack()][4] - nó chuyển đổi dữ liệu nhị phân đến với mảng liên kết dựa trên định dạng đã được chỉ định. Điều này giống với hàm _sprintf_, chuyển đổi các chuỗi dữ liệu theo nhưng định đạng cho trước. 2 function này cho phép chúng ta đọc và ghi các buffer của dữ liệu nhị phân theo như định dạng đã chỉ định. Điều này dễ dàng cho phép một lập trình viên chuyển đổi dữ liệu với chương trình đã được viết ở một ngôn ngữ khách hoặc định dạng khác. Một ví dụ nhỏ cho việc này.

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
 
 Trong các ví dụ ở trên tham số thứ nhất là chuỗi định dạng và tham số thứ 2 là dữ liệu thực. Chuỗi định dạng chỉ định rằng làm cách nào để tham số dữ liệu được đưa ra. Trong ví dụ này phần đầu tiên của định dạng 'C', chỉ định rằng chúng ta nên sửa chữ đầu tiên của dữ liệu như 1 kiểu byte nguyên. Phân tiếp theo '*', cho các function áp dụng các định dạng đã được chỉ định từ trước với tất cả các kí tự còn lại
 
 Mặc dù điều này có vẻ khó hiểu, ở phần tiếp theo sẽ cung cấp các ví dụ cụ thể hơn.
 
 #### Lấy dữ liệu header
 
 Dưới đây là giải pháp cho vấn để sử dụng function unpack() cho GIF. Function _is_gif()_ sẽ trả về dữ liệu đúng nếu file được đưa vào là định dạng GIF.

| ----- |
| 
    
    
    function is_gif($image_file)
    {
     
        /* Open the image file in binary mode */
        if(!$fp = fopen ($image_file, 'rb')) return 0;
     
        /* Read 20 bytes from the top of the file */
        if(!$data = fread ($fp, 20)) return 0;
     
        /* Create a format specifier */
        $header_format = 'A6version';  # Get the first 6 bytes
    
        /* Unpack the header data */
        $header = unpack ($header_format, $data);
     
        $ver = $header['version'];
     
        return ($ver == 'GIF87a' || $ver == 'GIF89a')? true : false;
     
    }
     
    /* Run our example */
    echo is_gif("aboutus.gif");

 |
 
 Dòng code quan trọng cần chú ý là phần khai báo định dạng. Kí tự 'A6' chỉ định rằng function unpack() lấy 6 byte đầu của dữ liệu và diễn tả nó như 1 chuỗi. Dữ lệu nhận được sau đó được lưu trữ trong một mảng liên quan với tên key là 'version'.
 
 Một ví dụ khác được đưa ra dưới đây. Nó trả về một số dữ liệu header thêm vào của file GIF, bao gồm chiều rộng và chiều cao ảnh.
 
 | ----- |
 | 
     
     
     function get_gif_header($image_file)
     {
      
         /* Open the image file in binary mode */
         if(!$fp = fopen ($image_file, 'rb')) return 0;
      
         /* Read 20 bytes from the top of the file */
         if(!$data = fread ($fp, 20)) return 0;
      
         /* Create a format specifier */
         $header_format = 
                 'A6Version/' . # Get the first 6 bytes
                 'C2Width/' .   # Get the next 2 bytes
                 'C2Height/' .  # Get the next 2 bytes
                 'C1Flag/' .    # Get the next 1 byte
                 '@11/' .       # Jump to the 12th byte
                 'C1Aspect';    # Get the next 1 byte
     
         /* Unpack the header data */
         $header = unpack ($header_format, $data);
      
         $ver = $header['Version'];
      
         if($ver == 'GIF87a' || $ver == 'GIF89a') {
             return $header;
         } else {
             return 0;
         }
     }
      
     /* Run our example */
     print_r(get_gif_header("aboutus.gif"));
 
  | 
  
  Ví dụ trên sẽ in ra như dưới đây khi chạy
  
  | ----- |
  | 
      
      
      Array
      (
          [Version] => GIF89a
          [Width1] => 97
          [Width2] => 0
          [Height1] => 33
          [Height2] => 0
          [Flag] => 247
          [Aspect] => 0
      )
  
   | 
  
  Dưới đây chúng ta sẽ đi vào chi tiết của một định dạng cụ thể hơn khi chạy. Tôi sẽ chia định dạng, đưa những chi tiết vào trong mỗi kí tự.
  
  | ----- |
  | 
      
      
      $header_format = 'A6Version/C2Width/C2Height/C1Flag/@11/C1Aspect';
  
   | 
  
  | ----- |
  | 
      
      
      A - Read a byte and interpret it as a string. 
          Number of bytes to read is given next
      6 - Read a total of 6 bytes, starting from position 0
      Version - Name of key in the associative array where data 
          retrieved by 'A6' is stored
       
      / - Start a new code format
      C - Interpret the next data as an unsigned byte
      2 - Read a total of 2 bytes
      Width - Key in the associative array
       
      / - Start a new code format
      C - Interpret the data as an unsigned byte
      2 - Read a total of 2 bytes
      Height- Key in the associative array
       
      / - Start a new code format
      C - Interpret the data as an unsigned byte
      1 - Read a total of 2 bytes
      Flag - Key in the associative array
       
      / - Start a new code format
      @ - Move to the byte offset specified by the following number.
            Remember that the first position in the binary string is 0. 
      11 - Move to position 11
       
      / - Start a new code format
      C - Interpret the data as an unsigned byte
      1 - Read a total of 1 bytes
      Aspect - Key in the associative array
  
   | 

Xem thêm các tùy chọn định dạng tại đây [here][4]. Trên đây tôi chỉ trình bày một ví dụ nhỏ, pack/unpack trong thực tế còn có thể làm những việc phức tạp hơn những thứ đã được viết ở trên.
