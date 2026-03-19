# Lab: Building a Centralized Log Monitoring System with ELK Stack

This folder contains a complete hands-on lab for building a **centralized log monitoring system (mini SOC)** using **ELK Stack**, **pfSense**, **Suricata**, and **Winlogbeat**.

The full step‑by‑step guide is documented in:

- ELK-Stack-Lab-Guide.md

Below is a quick table of contents so bạn có thể nắm được bố cục trước khi đọc chi tiết.

## Table of Contents (Lab Guide)

1. **Introduction & Scope**  
   Mục tiêu của lab, phạm vi, những gì bạn sẽ học được.

2. **Architecture Overview**  
   Mô hình mạng doanh nghiệp giả lập, các thành phần: pfSense, ELK Server, Suricata IDS/IPS, Domain Controller, WinClient, Attacker.

3. **Network & VM Planning**  
   Quy hoạch IP, thiết lập Virtual Networks, danh sách máy ảo và vai trò từng máy.

4. **ELK Server Deployment (192.168.20.100)**  
   Cài đặt và cấu hình **Elasticsearch**, **Kibana**, **Logstash**, bao gồm pipeline xử lý log từ pfSense, Suricata (Filebeat) và Winlogbeat.

5. **pfSense Log Forwarding**  
   Cấu hình Syslog trên pfSense để gửi log firewall về Logstash/Elasticsearch.

6. **Suricata IDS/IPS + Filebeat (192.168.20.5)**  
   Cài đặt Suricata, rule tự viết (Ping, SQL Injection), cấu hình Filebeat đọc `eve.json` và đẩy log về ELK.

7. **Winlogbeat on Domain Controller (192.168.20.10)**  
   Cấu hình Winlogbeat thu thập Windows Event Logs (đặc biệt sự kiện đăng nhập) và gửi về ELK.

8. **Kibana Data Views & Dashboards**  
   Tạo Data View, xây dựng Dashboard giám sát brute-force, log Windows, log pfSense/Suricata.

9. **Attack Scenarios & Detection**  
   - Demo 1: Brute-force RDP vào Domain Controller bằng Hydra và theo dõi `event.code: 4625`.
   - Demo 2: SQL Injection được phát hiện bởi Suricata và hiển thị trên Kibana.
   - Demo 3: Nmap scan vào pfSense WAN và phân tích log firewall.

10. **Conclusion & Next Steps**  
    Gợi ý mở rộng lab thành môi trường SOC mini: thêm nguồn log, cải thiện parsing Logstash, alerting, v.v.

> Để triển khai lab, hãy mở **ELK-Stack-Lab-Guide.md** và làm theo tuần tự từ đầu đến cuối. README này chỉ đóng vai trò mục lục và mô tả nhanh.
