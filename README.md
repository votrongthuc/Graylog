## Mục lục

------
#### [1 Giới thiệu](#1)
   - [1.1 Thành Phần](#1.1)



-------------------


### <a name="1"><a/>1 Giới thiệu.

- Graylog là một nền tảng quản lí log mã nguồn mở với nhiều tính năng mạnh mẽ. Nó có khả năng gộp chung và giải nén các dữ liệu quan trọng từ server log, thứ thường được gửi sử dụng giao thức Syslog. Bên cạnh đó Graylog cũng hỗ trợ việc tìm kiếm và giúp bạn hình dung cấu trúc của log thông qua một giao diện web.

- Bài viết này sẽ hướng dẫn bạn cài đặt và cấu hình Graylog trên server **Centos7**, đồng thời thiết lập một đầu vào đơn giản dùng để ghi system log.

### <a name="1.1"><a/>1.1 Thành Phần.

   - **MongoDB**: Lưu trữ các cấu hình và thông tin meta
   
   - **Elasticsearch**: Lữu trữ log và cung cấp dữ liệu tìm kiếm, phần chiếm nhiều tài nguyên các các hành động phần lớn sẽ được thự hiện tại đây.
   
   -  **GrayLog**: Phân tích cú pháp log và thu thập log từ các client gửi lên.
   
   - **GrayLog Web interface**: Quản lý bằng giao diện web
   
   
### <a name="2"><a/>2 Cài đặt   
   

### <a name="2.1"><a/>2.1 Cài đặt Packet.
    
        yum -y install epel-release
        
   - Install java 
   
        - Elasticsearch dựa trên nên java nên cần cài đặt JDK
        
                yum install java
                
       ![image](https://user-images.githubusercontent.com/19284401/55300780-6e44aa00-5463-11e9-97bd-2f4ca91521ec.png)
       
       
### <a name="2.2"><a/>2.2 Elasticsearch 

 - Import the GPG key
 
        rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch   
        
 - Tạo repo mới cho **Elasticseach** 
 
         vi /etc/yum.repos.d/elasticsearch.repo  
         
 - Pates nội dung sau vào file
 
 
        [elasticsearch-2.x]
        name=Elasticsearch repository for 2.x packages
        baseurl=https://packages.elastic.co/elasticsearch/2.x/centos
        gpgcheck=1
        gpgkey=https://packages.elastic.co/GPG-KEY-elasticsearch
        enabled=1
        
        
   - Hiện tại **Elasticseach**  phiên bản mới nhất là 6.7.
   
   - Tuy nhiên ở đây mình dùng bản 2.x, cụ thể hơn là 2.4. Vì đơn giản là bản 6.7 nó ngốn quá nhiều tài nguyện luôn.
   
   ![image](https://user-images.githubusercontent.com/19284401/55300976-7ea95480-5464-11e9-8316-75fe7bafa291.png)
   
   - Với cấu hình như trên mình chạy bản 5.x mà được vài phút là **Elasticseach** lăn ra chết vì ko đủ RAM. nhưng với bản 2.4 thì chạy gần 1 năm nay ko bị sao cả.
   
 - Cài đặt elasticsearch

       yum -y install elasticsearch
 
 
 - Cấu hình elasticsearch
 
        systemctl daemon-reload 
        
        systemctl enable elasticsearch.service
        
- Cấu hình elasticsearch

        vi /etc/elasticsearch/elasticsearch.yml        

     - Tìm đến dòng 17 sửa thành **graylog**
     
     ![image](https://user-images.githubusercontent.com/19284401/55301366-7a7e3680-5466-11e9-8628-becbb3463901.png)
     
 - Restart elasticsearch
 
        systemctl restart elasticsearch.service   
        
        
 - Check lại thông tin elasticsearch
        
        curl -X GET http://localhost:9200
        
   ![image](https://user-images.githubusercontent.com/19284401/55301675-edd47800-5467-11e9-900d-a4a10c75afd5.png)
   
 - Kiểm tra tình trạng "sức khỏe" của  elasticsearch 
 
        curl -XGET 'http://localhost:9200/_cluster/health?pretty=true'
        
        
   - Nếu bạn thấy dòng **"status" : "green"** là ngon rồi. 
   
     ![image](https://user-images.githubusercontent.com/19284401/55301816-9c78b880-5468-11e9-80ac-2fc39db144c8.png)
     
     
     
### <a name="2.3"><a/>2.3 Cài đặt Mongodb  

- Creat repo

        vi /etc/yum.repos.d/mongodb-org-3.6.repo
        
- Pates nội dung sau vào file 
        
        [mongodb-org-3.0]
        name=MongoDB Repository
        baseurl=http://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.0/x86_64/
        gpgcheck=0
        enabled=1
        
        
- Cài đặt Mongodb

          yum -y install mongodb-org 
        
- Nếu firewalldđang bật

        yum -y install policycoreutils-python
        
        semanage port -a -t mongod_port_t -p tcp 27017  
        
-  Nếu bạn ko muốn dùng firewalld
        
       systemctl stop firewalld
    
       systemctl disable firewalld
       
- Start và enable Mongodb

        systemctl start mongod
        
        systemctl enable mongod
        
### <a name="2.4"><a/>2.4 Cài đặt Graylog

- Tạo repo
            
            vi /etc/yum.repos.d/graylog.repo

- Pates nội dung sau vào file.

            [graylog]
            name=graylog
            baseurl=https://packages.graylog2.org/repo/el/stable/2.4/$basearch/
            gpgcheck=1
            repo_gpgcheck=0
            gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-graylog    
            
            
- Cài đặt

            yum -y install graylog-server
            
- Cấu hình 

            vi /etc/graylog/server/server.conf
            
- Tìm đến dòng 55 Thêm bất kì những ký tự nào đó trên bàn phím vào sau dâu = cũng được (có có ký tự đặc biệt)

    password_secret =fahiogbfaerogiboierahgowebgaw0eoigbwguoptgbobofrbwepiufbgierubgfwuipfbw
    
    ![image](https://user-images.githubusercontent.com/19284401/55302346-ec587f00-546a-11e9-8959-c8e05b442cf2.png)
    
- Nếu bạn muộn thể hiện sự chuyên nghiệp thì bạn dùng lệnh sau để gen ra 1 chuỗi ngẫu nhiên rồi pates sau dấu =

            yum -y install pwgen
            
            pwgen -N 1 -s 96
            
- Tìm tiếp đến dòng 66 thay đoạn code vào sau dâu = 

        echo -n yourpassword | sha256sum 
        
  - Thay  **yourpassword** bằng password của bạn
        
  ![image](https://user-images.githubusercontent.com/19284401/55302715-a00e3e80-546c-11e9-9f63-995825d93268.png)
        
  - Chú ý bỏ cài dấu - ở cuối dùng đi nhé 
   
  ![image](https://user-images.githubusercontent.com/19284401/55302734-b9af8600-546c-11e9-977a-9668ea12ac5b.png)
  
  
### <a name="2.5"><a/>2.5 Cấu hình Graylog web interface
        
        vi /etc/graylog/server/server.conf
        
-  Tìm đến dòng 81 và 132 vào sửa lại ip cho hợp lý.        
            
   ![image](https://user-images.githubusercontent.com/19284401/55303068-56beee80-546e-11e9-9e2c-4f88dd60b1a3.png)
   
   ![image](https://user-images.githubusercontent.com/19284401/55303124-92f24f00-546e-11e9-8121-2039203481b9.png)
   
   
- Restart và enable graylog-server
 
        systemctl daemon-reload
        
        systemctl restart graylog-server
        
        systemctl enable graylog-server


- Truy cập 

    http://your_ip_server:9000
    
    ![image](https://user-images.githubusercontent.com/19284401/55304682-d486f800-5476-11e9-963c-1c54c967c277.png)
    
    - user mặc định là **admin** 
    
    - pass bạn đã tạo ở bước trước "echo -n **yourpassword** | sha256sum "
    
- Đăng nhập xong bạn vào tab **SEARCH**

    ![image](https://user-images.githubusercontent.com/19284401/55304827-94744500-5477-11e9-97a7-269180ffbbc3.png)
    
    - Đây là nơi hiển thị toàn bộ log mà server thư thập được từ client gửi lên.
    
    

    
    
  
  
 
 
      
                     
                             
        
        
               
     
     
   
 
 
   

     
     
     


