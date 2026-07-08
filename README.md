# HỆ THỐNG MẠNG DOANH NGHIỆP NGÂN HÀNG THƯƠNG MẠI ABC
## Đề tài Bài tập lớn môn Mạng máy tính & Quản trị hệ thống — Nhóm 3

---

## 1. Tổng quan dự án

Trong hoạt động của một ngân hàng thương mại hiện đại, hạ tầng công nghệ thông tin (CNTT) đóng vai trò cốt lõi quyết định sự vận hành trơn tru, bảo mật và an toàn của toàn hệ thống. Dự án này tập trung thiết kế và mô phỏng hệ thống mạng doanh nghiệp hoàn chỉnh cho **Ngân hàng Thương mại ABC** trên phần mềm **Cisco Packet Tracer**. 

Hệ thống được thiết kế đáp ứng đầy đủ các tiêu chuẩn bảo mật khắt khe, phân vùng mạng nghiệp vụ chi tiết, triển khai các dịch vụ nội bộ (DHCP, DNS, Web, Database) và kết nối bảo mật từ xa (VPN). Sơ đồ logic này là nền tảng thiết kế phục vụ triển khai thực tế trên hệ điều hành **Windows Server 2025** chạy trên nền tảng ảo hóa doanh nghiệp (**VMware vSphere / Proxmox VE**).

---

## 2. Danh sách yêu cầu kỹ thuật & Đóng góp hệ thống

### Yêu cầu chung:
*   **Quy mô hạ tầng:** Hỗ trợ tối thiểu 100 nhân viên làm việc đồng thời tại Hội sở chính và kết nối an toàn đến chi nhánh.
*   **Phân vùng mạng (VLAN):** Thiết lập tối thiểu 15 VLANs để cô lập lưu lượng, tối ưu hóa băng thông và bảo mật cho từng phòng ban (Giám đốc, Kế toán, Tín dụng, Giao dịch, Kiểm toán, CNTT, ATM, Camera, WiFi nội bộ, WiFi khách, Server...).
*   **Thiết bị phần cứng:** Mô phỏng tối thiểu 5 Switches, 3 Routers và 2 Firewalls Cisco.
*   **Hạ tầng dịch vụ:** Triển khai cấp phát IP tự động (DHCP), phân giải tên miền (DNS) nội bộ, cổng thông tin Internet Banking (Web Server) và cơ sở dữ liệu giao dịch bảo mật (Database Server).
*   **Bảo mật đa lớp:** Sử dụng tường lửa cứng Cisco ASA 5506-X làm lá chắn bảo vệ vùng DMZ chứa Web Server; thiết lập chính sách ACL ngăn chặn kết nối trái phép từ các vùng nhạy cảm (ATM, Camera, WiFi Khách) vào mạng LAN chính.
*   **Kênh truyền VPN:** Thiết lập kênh truyền VPN Site-to-Site giữa Hội sở và Chi nhánh cùng giải pháp VPN Remote Access cho nhân viên làm việc từ xa.
*   **Ảo hóa doanh nghiệp:** Thiết kế hạ tầng sẵn sàng cho ảo hóa máy chủ trên Proxmox/VMware.

---

## 3. Thiết kế kiến trúc hệ thống

### 3.1 Bảng phân hoạch VLAN (VLAN Allocation Table)

| VLAN ID | Tên VLAN | Phân khúc Subnet | Địa chỉ Default Gateway | Ghi chú |
|---|---|---|---|---|
| **10** | GiamDoc | 192.168.10.0/24 | 192.168.10.1 | Dành riêng cho Ban Giám đốc |
| **20** | KeToan | 192.168.20.0/24 | 192.168.20.1 | Bộ phận tài chính - kế toán |
| **30** | TinDung | 192.168.30.0/24 | 192.168.30.1 | Bộ phận hỗ trợ tín dụng |
| **40** | GiaoDich | 192.168.40.0/24 | 192.168.40.1 | Bộ phận giao dịch viên |
| **50** | KiemToan | 192.168.50.0/24 | 192.168.50.1 | Bộ phận kiểm soát nội bộ |
| **60** | CNTT | 192.168.60.0/24 | 192.168.60.1 | Đội ngũ quản trị hệ thống |
| **70** | ATM | 192.168.70.0/24 | 192.168.70.1 | Kết nối các cây rút tiền ATM |
| **80** | Camera | 192.168.80.0/24 | 192.168.80.1 | Hệ thống camera an ninh giám sát |
| **90** | WiFiNoiBo | 192.168.90.0/24 | 192.168.90.1 | Sóng WiFi dành cho nhân viên |
| **91** | WiFiKhach | 192.168.91.0/24 | 192.168.91.1 | Sóng WiFi công cộng cho khách hàng |
| **100** | Server | 192.168.100.0/24 | 192.168.100.1 | Phân vùng Server Farm nội bộ |
| **110** | Quanly | 192.168.110.0/24 | 192.168.110.1 | Quản trị thiết bị mạng (OOBM) |
| **120** | NhanSu | 192.168.120.0/24 | 192.168.120.1 | Phòng Tổ chức nhân sự |
| **130** | PhapChe | 192.168.130.0/24 | 192.168.130.1 | Phòng Pháp chế & Pháp lý |
| **140** | PhongHop | 192.168.140.0/24 | 192.168.140.1 | Phòng họp và phòng hội nghị |
| **DMZ** | DMZ (Vùng biên) | 172.16.1.0/24 | 172.16.1.1 | Chứa Web Server phục vụ Internet Banking |

---

### 3.2 Danh sách thiết bị phần cứng & Vai trò

| Tên thiết bị | Model | Số lượng | Vai trò trong hệ thống |
|---|---|---|---|
| **R1** | Cisco 2911 | 1 | **Router Core:** Định tuyến liên VLAN (Router-on-a-Stick), cấu hình DHCP Relay Agent. |
| **R2** | Cisco 2911 | 1 | **Router biên:** Đấu nối mạng Internet/ISP, thiết lập đầu cuối VPN GRE Tunnel cho Hội sở. |
| **R-Branch** | Cisco 2911 | 1 | **Router chi nhánh:** Thiết lập đầu cuối VPN kết nối mạng chi nhánh về Hội sở. |
| **FW1** | Cisco ASA 5506-X | 1 | **Firewall biên:** Phân chia vùng an ninh (`inside`, `outside`, `dmz`), thực hiện Static NAT và kiểm soát ACL an ninh. |
| **SW1** | Cisco 2960 | 1 | **Core Switch:** Switch lõi trung chuyển trunking toàn bộ các VLAN lên Router Core và Access Switch. |
| **SW2** | Cisco 2960 | 1 | **Access Switch Nghiệp vụ:** Gán cổng access cho các máy trạm VLAN 10 đến VLAN 50. |
| **SW3** | Cisco 2960 | 1 | **Access Switch Hạ tầng:** Gán cổng access cho VLAN CNTT, ATM và Camera (VLAN 60, 70, 80). |
| **SW4** | Cisco 2960 | 1 | **Access Switch Server Farm:** Cung cấp kết nối cho các máy chủ nội bộ (DNS, DHCP, DB). |
| **SW5** | Cisco 2960 | 1 | **Access Switch WiFi & Phụ trợ:** Kết nối các Access Point và các phòng ban hành chính còn lại. |
| **DNS-Server** | Máy chủ vật lý | 1 | Phân giải tên miền nội bộ (`abcbank.com.vn`). |
| **DHCP-Server** | Máy chủ vật lý | 1 | Quản lý 12 pool cấp IP động cho toàn bộ máy trạm. |
| **Web-Server** | Máy chủ vật lý | 1 | Chạy cổng Web Internet Banking vùng DMZ. |
| **DB-Server** | Máy chủ vật lý | 1 | Cơ sở dữ liệu tài khoản, khách hàng và giao dịch nội bộ. |

---

### 3.3 Sơ đồ kết nối logic (Topology)

```
                       Cộng đồng Internet (Public WAN)
                                    |
            R2 (Router Biên Hội sở) ─────── GRE Tunnel ─────── R-Branch (Router Chi nhánh)
                               |                                      |
                     FW1 (Firewall ASA 5506-X)                   PC-Chi Nhánh
                   /     (Security Level)      \
           (inside: 100)                    (dmz: 50)
                 /                                \
    R1 (Router Core Lớp 3)                     Web-Server (Internet Banking)
                 |
      SW1 (Core Switch Lớp 2)
     /     /      \       \
   SW2    SW3     SW4     SW5
  (LAN) (Hạ tầng) (Server) (WiFi)
```

---

### 3.4 Bảng đấu dây chi tiết (Cabling Matrix)

| STT | Thiết bị đầu A | Cổng đấu nối A | Loại cáp sử dụng | Thiết bị đầu B | Cổng đấu nối B |
|---|---|---|---|---|---|
| 1 | R2 | GigabitEthernet0/0 | Cáp thẳng (Copper Straight-Through) | FW1 | GigabitEthernet1/1 |
| 2 | FW1 | GigabitEthernet1/3 | Cáp thẳng (Copper Straight-Through) | Web-Server | FastEthernet0 |
| 3 | FW1 | GigabitEthernet1/2 | Cáp thẳng (Copper Straight-Through) | R1 | GigabitEthernet0/0 |
| 4 | R2 | GigabitEthernet0/2 | Cáp thẳng (Copper Straight-Through) | R-Branch | GigabitEthernet0/1 |
| 5 | R1 | GigabitEthernet0/1 | Cáp thẳng (Copper Straight-Through) | SW1 | GigabitEthernet0/1 |
| 6 | SW1 | FastEthernet0/1 | Cáp chéo (Copper Cross-Over) | SW2 | FastEthernet0/1 |
| 7 | SW1 | FastEthernet0/2 | Cáp chéo (Copper Cross-Over) | SW3 | FastEthernet0/1 |
| 8 | SW1 | FastEthernet0/4 | Cáp chéo (Copper Cross-Over) | SW5 | FastEthernet0/1 |
| 9 | SW1 | FastEthernet0/5 | Cáp thẳng (Copper Straight-Through) | SW4 | FastEthernet0/1 |
| 10 | SW2 | FastEthernet0/2 | Cáp thẳng (Copper Straight-Through) | PC-GiamDoc | FastEthernet0 |
| 11 | SW2 | FastEthernet0/3 | Cáp thẳng (Copper Straight-Through) | PC-KeToan | FastEthernet0 |
| 12 | SW3 | FastEthernet0/2 | Cáp thẳng (Copper Straight-Through) | PC-CNTT | FastEthernet0 |
| 13 | SW3 | FastEthernet0/3 | Cáp thẳng (Copper Straight-Through) | PC-ATM | FastEthernet0 |
| 14 | SW3 | FastEthernet0/4 | Cáp thẳng (Copper Straight-Through) | PC-Camera | FastEthernet0 |
| 15 | SW4 | FastEthernet0/2 | Cáp thẳng (Copper Straight-Through) | DNS-Server | FastEthernet0 |
| 16 | SW4 | FastEthernet0/3 | Cáp thẳng (Copper Straight-Through) | DHCP-Server | FastEthernet0 |
| 17 | SW4 | FastEthernet0/4 | Cáp thẳng (Copper Straight-Through) | DB-Server | FastEthernet0 |
| 18 | SW5 | FastEthernet0/6 | Cáp thẳng (Copper Straight-Through) | AP1-NoiBo | FastEthernet0 |
| 19 | SW5 | FastEthernet0/7 | Cáp thẳng (Copper Straight-Through) | AP2-Khach | FastEthernet0 |

---

## 4. Tập lệnh cấu hình chi tiết (CLI Commands Sheets)

### 4.1 Cấu hình VLAN & Trunking trên Core Switch SW1

Thực hiện khai báo danh sách cơ sở dữ liệu các VLAN nghiệp vụ và thiết lập cấu hình đường Trunk trên các cổng kết nối lên Router và xuống các Access Switch:

```ios
! Truy cập chế độ cấu hình toàn cục
configure terminal

! Khai báo cơ sở dữ liệu các VLAN
vlan 10
 name GiamDoc
vlan 20
 name KeToan
vlan 30
 name TinDung
vlan 40
 name GiaoDich
vlan 50
 name KiemToan
vlan 60
 name CNTT
vlan 70
 name ATM
vlan 80
 name Camera
vlan 90
 name WiFiNoiBo
vlan 91
 name WiFiKhach
vlan 100
 name Server
vlan 110
 name Quanly
vlan 120
 name NhanSu
vlan 130
 name PhapChe
vlan 140
 name PhongHop
exit

! Thiết lập cổng Trunk nối lên Router Core R1
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan all
 no shutdown
 exit

! Cấu hình các cổng Trunk nối sang Switch nhánh
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan all
 exit
interface FastEthernet0/2
 switchport mode trunk
 switchport trunk allowed vlan all
 exit
interface FastEthernet0/4
 switchport mode trunk
 switchport trunk allowed vlan all
 exit
interface FastEthernet0/5
 switchport mode trunk
 switchport trunk allowed vlan all
 exit
```

---

### 4.2 Cấu hình Router-on-a-Stick & DHCP Relay trên Router Core R1

Router R1 đóng vai trò định tuyến liên VLAN (Inter-VLAN routing) bằng cách chia cổng vật lý thành nhiều subinterface logic, đồng thời chuyển tiếp yêu cầu DHCP (DHCP Relay Agent) tới DHCP-Server (`192.168.100.10`):

```ios
configure terminal

! Bật cổng vật lý hướng kết nối xuống SW1
interface GigabitEthernet0/1
 no shutdown
 exit

! --- CẤU HÌNH SUBINTERFACE CHO CÁC VLAN ---
! VLAN 10 (Giám đốc)
interface GigabitEthernet0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 192.168.100.10
 no shutdown
 exit

! VLAN 20 (Kế toán)
interface GigabitEthernet0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 192.168.100.10
 no shutdown
 exit

! VLAN 60 (CNTT)
interface GigabitEthernet0/1.60
 encapsulation dot1Q 60
 ip address 192.168.60.1 255.255.255.0
 ip helper-address 192.168.100.10
 no shutdown
 exit

! VLAN 70 (ATM)
interface GigabitEthernet0/1.70
 encapsulation dot1Q 70
 ip address 192.168.70.1 255.255.255.0
 ip helper-address 192.168.100.10
 no shutdown
 exit

! VLAN 80 (Camera)
interface GigabitEthernet0/1.80
 encapsulation dot1Q 80
 ip address 192.168.80.1 255.255.255.0
 ip helper-address 192.168.100.10
 no shutdown
 exit

! VLAN 100 (Server Farm - Không cần DHCP Helper)
interface GigabitEthernet0/1.100
 encapsulation dot1Q 100
 ip address 192.168.100.1 255.255.255.0
 no shutdown
 exit
```

---

### 4.3 Cấu hình an ninh các phân vùng vùng DMZ và Firewall ASA 5506-X

Cấu hình Firewall làm cổng chặn an ninh, phân chia 3 vùng (inside, outside, dmz) tương ứng với 3 mức độ bảo mật. Thực hiện Static NAT trỏ từ IP Public ra IP Private của Web Server trong vùng DMZ và đặt Access Control List (ACL) cho phép truy cập:

```ios
! Truy cập chế độ cấu hình thiết bị ASA
configure terminal

! Cấu hình cổng kết nối mạng bên ngoài Internet (outside)
interface GigabitEthernet1/1
 nameif outside
 security-level 0
 ip address 10.1.1.2 255.255.255.0
 no shutdown
 exit

! Cấu hình cổng kết nối mạng LAN nội bộ (inside)
interface GigabitEthernet1/2
 nameif inside
 security-level 100
 ip address 192.168.1.2 255.255.255.0
 no shutdown
 exit

! Cấu hình cổng kết nối vùng DMZ bảo vệ Web Server
interface GigabitEthernet1/3
 nameif dmz
 security-level 50
 ip address 172.16.1.1 255.255.255.0
 no shutdown
 exit

! --- CẤU HÌNH STATIC NAT CHO WEB SERVER (Internet Banking) ---
object network Web_Server_Private
 host 172.16.1.10
 nat (dmz,outside) static 10.1.1.10
 exit

! --- CẤU HÌNH ACCESS CONTROL LIST (ACL) ---
! Cho phép truy cập HTTP (80), HTTPS (443) và ICMP Ping từ Internet vào Web Server vùng DMZ
access-list OUTSIDE_TO_DMZ extended permit tcp any host 172.16.1.10 eq 80
access-list OUTSIDE_TO_DMZ extended permit tcp any host 172.16.1.10 eq 443
access-list OUTSIDE_TO_DMZ extended permit icmp any host 172.16.1.10
! Áp dụng ACL vào luồng chiều IN trên cổng outside
access-group OUTSIDE_TO_DMZ in interface outside
```

---

### 4.4 Thiết lập ACL Bảo mật chặn kết nối mạng nội bộ trên Router Core R1

Nhằm cô lập các vùng nhạy cảm, nhóm đã triển khai các bộ lọc ACL (Access Control List) trên Router Core R1:

```ios
configure terminal

! --- ACL 101: BLOCK-ATM ---
! Cho phép ATM kết nối tới DNS (192.168.100.20), DHCP (10), DB-Server (192.168.100.40)
access-list 101 permit ip 192.168.70.0 0.0.0.255 host 192.168.100.10
access-list 101 permit ip 192.168.70.0 0.0.0.255 host 192.168.100.20
access-list 101 permit ip 192.168.70.0 0.0.0.255 host 192.168.100.40
! Chặn toàn bộ kết nối từ ATM sang các dải mạng LAN nội bộ khác
access-list 101 deny ip 192.168.70.0 0.0.0.255 192.168.0.0 0.0.255.255
access-list 101 permit ip any any

! --- ACL 102: BLOCK-CAMERA ---
! Cho phép Camera gửi dữ liệu ghi hình về Database/Storage Server
access-list 102 permit ip 192.168.80.0 0.0.0.255 host 192.168.100.40
! Cô lập hoàn toàn Camera khỏi các mạng LAN nội bộ và mạng quản trị khác
access-list 102 deny ip 192.168.80.0 0.0.0.255 192.168.0.0 0.0.255.255
access-list 102 permit ip any any

! --- ÁP DỤNG ACL VÀO CÁC SUBINTERFACE TƯƠNG ỨNG ---
interface GigabitEthernet0/1.70
 ip access-group 101 in
 exit
interface GigabitEthernet0/1.80
 ip access-group 102 in
 exit
```

---

### 4.5 Cấu hình VPN Site-to-Site bằng GRE Tunnel

Cấu hình VPN thiết lập kênh truyền ảo mã hóa kết nối giữa mạng nội bộ Hội sở và mạng LAN Chi nhánh:

#### Cấu hình tại Router Biên Hội sở R2:
```ios
configure terminal
interface Tunnel0
 ip address 172.16.100.1 255.255.255.252
 tunnel source GigabitEthernet0/2
 tunnel destination 10.2.2.2
 no shutdown
 exit
! Định tuyến sang mạng LAN Chi nhánh thông qua IP đầu Tunnel của Chi nhánh
ip route 192.168.200.0 255.255.255.0 172.16.100.2
```

#### Cấu hình tại Router Chi nhánh R-Branch:
```ios
configure terminal
interface Tunnel0
 ip address 172.16.100.2 255.255.255.252
 tunnel source GigabitEthernet0/1
 tunnel destination 10.2.2.1
 no shutdown
 exit
! Định tuyến về toàn bộ dải mạng LAN Hội sở thông qua IP đầu Tunnel của Hội sở
ip route 192.168.0.0 255.255.0.0 172.16.100.1
```

---

## 5. Tự động hóa quản trị mạng (Network Automation Python Script)

Dưới đây là mã nguồn Python mẫu sử dụng thư viện **Netmiko** để tự động kết nối qua SSH vào Core Switch SW1, truy vấn trạng thái cơ sở dữ liệu các VLAN và tự động khởi tạo VLAN mới phục vụ nhu cầu mở rộng phòng ban:

```python
from netmiko import ConnectHandler
from netmiko.exceptions import NetmikoTimeoutException, NetmikoAuthenticationException

# Định nghĩa thông số kết nối SSH tới Cisco Switch
cisco_switch = {
    "device_type": "cisco_ios",
    "host": "192.168.110.10",    # Địa chỉ IP quản trị Out-of-Band
    "username": "admin_network",
    "password": "SecurePassword123",
    "secret": "EnableSecret123",  # Mật khẩu đặc quyền (Privilege)
}

def automate_switch_configuration():
    try:
        print("[INFO] Đang thiết lập kết nối SSH tới Switch SW1...")
        connection = ConnectHandler(**cisco_switch)
        connection.enable()
        
        # Đọc danh sách VLAN hiện tại trên thiết bị
        print("[INFO] Đang truy xuất danh sách VLAN...")
        output_vlan = connection.send_command("show vlan brief")
        print("\n=== CƠ SỞ DỮ LIỆU VLAN HIỆN TẠI ===")
        print(output_vlan)
        
        # Tạo thêm một VLAN nghiệp vụ mới (ví dụ: VLAN 150 cho Phòng Nghiên Cứu)
        new_vlan_commands = [
            "vlan 150",
            "name PhongNghienCuu",
            "exit"
        ]
        print("\n[INFO] Đang tiến hành tạo thêm VLAN 150...")
        config_output = connection.send_config_set(new_vlan_commands)
        print(config_output)
        
        # Lưu cấu hình từ RAM vào bộ nhớ Flash (startup-config)
        print("[INFO] Đang lưu cấu hình vào startup-config...")
        save_output = connection.send_command("write memory")
        print(f"[SUCCESS] Kết quả lưu: {save_output.strip()}")
        
        # Ngắt kết nối SSH
        connection.disconnect()
        print("[INFO] Đã ngắt kết nối SSH an toàn.")
        
    except NetmikoTimeoutException:
        print("[ERROR] Kết nối thất bại: Hết thời gian chờ (Timeout)!")
    except NetmikoAuthenticationException:
        print("[ERROR] Xác thực thất bại: Sai Username hoặc Password!")
    except Exception as e:
        print(f"[ERROR] Đã xảy ra lỗi hệ thống: {str(e)}")

if __name__ == "__main__":
    automate_switch_configuration()
```

---

## 6. Các lỗi thường gặp và cách xử lý (Troubleshooting)

| Lỗi gặp phải | Nguyên nhân gốc rễ | Hướng giải quyết & Khắc phục |
|---|---|---|
| Cổng kết nối Router/Firewall báo màu đỏ (Down) | Theo mặc định của Cisco, toàn bộ các cổng vật lý trên Router và Firewall ASA đều ở trạng thái tắt (`shutdown`). | Truy cập vào interface tương ứng qua CLI và gõ lệnh `no shutdown` để mở cổng. |
| Ping thử nghiệm liên VLAN thất bại mặc dù cấu hình định tuyến đúng | Switch lõi SW1 chưa được khai báo đầy đủ cơ sở dữ liệu (VLAN database) nên không nhận diện tag VLAN. | Tiến hành khai báo lần lượt các VLAN từ 10 đến 140 trực tiếp trong chế độ config trên SW1. |
| Firewall ASA 5506-X báo lỗi cú pháp khi chia cổng phụ (subinterface) | Phiên bản tường lửa ASA 5506-X được giả lập trên Packet Tracer không hỗ trợ Trunking/Subinterface. | Đấu nối trực tiếp Access Switch Server (SW4) vào Switch Core SW1. Cổng DMZ trên Firewall nối trực tiếp Web Server để quản lý an ninh độc lập. |
| Thiết bị đầu cuối không nhận được địa chỉ IP DHCP (lỗi APIPA `169.254.x.x`) | Yêu cầu DHCP Request (Broadcast) bị chặn bởi Router định tuyến hoặc bị rớt tại Firewall lớp 2. | Cấu hình lệnh `ip helper-address 192.168.100.10` trên từng subinterface của R1 và bypass kết nối trực tiếp SW4 sang SW1 để loại bỏ lọc DHCP trên Firewall nội bộ. |
| Lệnh `crypto isakmp` thiết lập IPSec báo lỗi `Invalid input` | Thiết bị Router trong bản Cisco Packet Tracer thông dụng bị giới hạn, không hỗ trợ tập lệnh mã hóa IPSec VPN hoàn chỉnh. | Sử dụng công nghệ đường hầm **GRE Tunnel** thay thế cho mô phỏng kết nối VPN Site-to-Site. |

---

## 7. Các tệp tin đính kèm trong dự án

*   **`De1_NganHang_Duc_v2.pkt`**: Tệp tin mô phỏng sơ đồ mạng hoàn chỉnh trên công cụ Cisco Packet Tracer (Phiên bản khuyến nghị: **8.2.x** trở lên).
*   **`nhom3.pdf`**: Báo cáo báo cáo chi tiết bằng văn bản (.PDF) của dự án Nhóm 3 trình bày cơ sở lý thuyết, phân tích hệ thống và kết quả kiểm thử.

---

## 8. Hướng dẫn chạy & Sử dụng file giả lập `.pkt`

1.  Tải và cài đặt phần mềm **Cisco Packet Tracer** (phiên bản tối thiểu **8.2.0**).
2.  Tải tệp tin `De1_NganHang_Duc_v2.pkt` về máy tính cục bộ.
3.  Mở tệp tin bằng Cisco Packet Tracer. Đợi khoảng 30 - 60 giây để toàn bộ giao thức Spanning Tree Protocol (STP) hội tụ (các chấm chuyển sang màu xanh lá cây).
4.  **Kiểm tra DHCP:** Click vào một máy trạm bất kỳ (ví dụ: `PC-GiamDoc` hoặc `PC-KeToan`), truy cập mục **Desktop** > **IP Configuration**, chuyển chế độ sang **DHCP** để kiểm tra máy nhận thành công địa chỉ IP động.
5.  **Kiểm tra Web:** Click vào một PC trong mạng LAN, mở mục **Web Browser**, nhập địa chỉ URL `abcbank.com.vn` và nhấn Enter để kiểm chứng kết nối tới cổng dịch vụ Internet Banking của Ngân hàng ABC.
6.  **Kiểm tra ACL:** Mở **Command Prompt** trên máy trạm `PC-ATM` và gõ lệnh `ping 192.168.10.10` (Ping sang phòng Giám đốc). Xác nhận kết quả trả về là `Request timed out` (Chặn thành công).

---

## 9. Thành viên thực hiện dự án (Nhóm 3)
*   **Sinh viên thực hiện:** Nguyễn Đức - Mã SV: ...
*   **Giảng viên hướng dẫn:** Thầy Nguyễn Văn An
*   **Đơn vị:** Khoa Công nghệ thông tin - Trường Đại học Đại Nam (DNU)
