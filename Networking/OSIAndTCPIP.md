# OSI vs TCP/IP — Từ lịch sử đến encapsulation

## 1. Bối cảnh lịch sử: vì sao cần một chuẩn chung

Cuối thập niên 1970, mỗi hãng lớn tự phát triển một bộ giao thức mạng riêng, không nói chuyện được với nhau:

- **IBM** — SNA (Systems Network Architecture)
- **DEC** — DECnet
- **Xerox** — XNS (Xerox Network Systems)
- **Novell** — IPX/SPX (sau này)
- **DARPA/ARPANET** (Mỹ) — TCP/IP, riêng cho mạng của họ

Mỗi bộ giao thức có cách đóng gói, đánh địa chỉ, truyền tải hoàn toàn khác nhau. Máy chạy SNA của IBM không nói chuyện được với máy chạy DECnet của DEC — không phải vì khác phần cứng, mà vì toàn bộ logic tầng mạng, tầng vận chuyển đều khác nhau từ gốc. Đây là vấn đề vendor lock-in kinh điển: mua thiết bị hãng nào thì phải dùng cả hệ sinh thái mạng của hãng đó.

**ISO ra tay:** Năm 1977, ISO (International Organization for Standardization) bắt đầu phát triển OSI Reference Model, công bố chính thức khoảng 1984, với mục tiêu tạo ra một khung tham chiếu trung lập, không thuộc vendor nào — để bất kỳ hãng nào implement đúng chuẩn cũng tương thích với nhau. Đó là lý do OSI được thiết kế rất tách bạch: 7 lớp, mỗi lớp là một "hợp đồng" chuẩn.

**Plot twist:** Bộ giao thức thật của OSI (CLNP, X.400, X.500...) gần như thất bại ngoài thực tế. Trong khi đó TCP/IP — ra đời sớm hơn cho ARPANET — lại được Internet toàn cầu chọn làm nền tảng thật, vì đã chạy ổn định và có sẵn cộng đồng dùng (Unix, các trường đại học) trước khi OSI kịp hoàn thiện.

Kết quả cuối cùng:
- **OSI** thắng ở vai trò **mô hình tham chiếu / khung tư duy** để phân tích và troubleshoot.
- **TCP/IP** thắng ở vai trò **bộ giao thức thực chiến** chạy cả Internet.

CCNA dạy song song vì lý do đó: dùng "ngôn ngữ OSI" để mô tả/troubleshoot, nhưng "chạy" bằng giao thức TCP/IP thật.

---

## 2. Mô hình OSI — 7 lớp

| Lớp | Vai trò cốt lõi | PDU | Ví dụ |
|---|---|---|---|
| 7. Application | Giao diện cho ứng dụng người dùng | Data | HTTP, DNS, SMTP |
| 6. Presentation | Định dạng, mã hóa, nén dữ liệu | Data | TLS encoding, JPEG, ASCII/EBCDIC |
| 5. Session | Thiết lập/duy trì/kết thúc phiên giao tiếp | Data | Session ID, RPC, NetBIOS |
| 4. Transport | Giao tiếp end-to-end, đảm bảo tin cậy | Segment (TCP) / Datagram (UDP) | TCP, UDP |
| 3. Network | Định địa chỉ logic, định tuyến | Packet | IP, ICMP |
| 2. Data Link | Đóng khung, địa chỉ vật lý trong LAN | Frame | Ethernet, ARP, PPP |
| 1. Physical | Truyền bit thô qua môi trường vật lý | Bit | Cáp đồng, cáp quang, sóng radio |

Ghi nhớ thứ tự (từ dưới lên): **"Please Do Not Throw Sausage Pizza Away"** — Physical, Data link, Network, Transport, Session, Presentation, Application.

Vài điểm hay bị hiểu lầm:

- **Layer 6 (Presentation)** cụ thể gồm 3 việc: translation (chuyển định dạng ký tự/số giữa các hệ thống), compression, encryption. Về lý thuyết TLS "sống" ở ranh giới Session/Presentation, dù dân dev thực tế hay quy nó vào Transport.
- **Layer 5 (Session)** dễ bị bỏ qua vì TCP/IP không có khái niệm session riêng — TCP tự lo phần "duy trì kết nối" (3-way handshake, sequence number), nên chức năng Session gần như bị nuốt vào Transport trong thực tế.
- Đây chính là lý do 3 lớp trên cùng của OSI (Application/Presentation/Session) không có ranh giới rõ ràng trong code thực tế — application thường tự lo luôn cả phần presentation/session.

---

## 3. Mô hình TCP/IP — 4 lớp

**Network Access (Link layer)**
- Gộp cả Physical (đẩy bit ra dây) lẫn Data Link (đóng frame + MAC address) của OSI.
- Đây là nơi **switching** diễn ra: switch học MAC address qua CAM table, forward frame trong cùng broadcast domain.
- Khái niệm quan trọng: **MTU** — Ethernet mặc định 1500 bytes. Packet lớn hơn MTU sẽ phải fragment (hoặc bị drop + gửi ICMP "packet too big" nếu có cờ Don't Fragment).

**Internet layer**
- IP là "hợp đồng chung" để các loại network khác nhau (Ethernet, Wi-Fi, 4G...) hiểu nhau ở tầng logic.
- ICMP là giao thức báo lỗi/chẩn đoán (ping, traceroute, destination unreachable...), chạy trên IP nhưng không có port như Transport thật sự.
- Routing table + longest prefix match là cơ chế router dùng để quyết định hướng đi của gói tin.

**Transport layer**
- **TCP**: connection-oriented — 3-way handshake (SYN → SYN/ACK → ACK), sequence number đảm bảo thứ tự, ACK đảm bảo tin cậy, sliding window để flow control.
- **UDP**: connectionless — gửi xong là xong, không đảm bảo gì. Dùng cho DNS query, video streaming, VoIP.
- **Port number**: Well-known (0–1023: 80=HTTP, 443=HTTPS, 22=SSH), Registered (1024–49151), Dynamic/Ephemeral (49152–65535, client tự chọn khi kết nối ra ngoài).

**Application layer**
- Lớp duy nhất user "thấy được" trực tiếp. Mọi giao thức (HTTP, DNS, SMTP, FTP) chỉ là "ngôn ngữ" 2 ứng dụng thỏa thuận để trao đổi dữ liệu — việc vận chuyển thật sự do 3 lớp dưới lo.

---

## 4. Ánh xạ OSI ↔ TCP/IP

```
OSI                           TCP/IP
Application    ─┐
Presentation    ├──────────►  Application     (PDU: Data)
Session        ─┘
Transport      ────────────►  Transport        (PDU: Segment)
Network        ────────────►  Internet         (PDU: Packet)
Data Link      ─┐
Physical       ─┴──────────►  Network Access   (PDU: Frame)
```

TCP/IP không "bỏ" 3 lớp trên của OSI — nó gom các chức năng gần nhau thành nhóm lớn hơn. 4 lớp TCP/IP không có nghĩa network stack chỉ có 4 nhóm chức năng, chỉ là một cách tổ chức khác.

---

## 5. Encapsulation / Decapsulation

Nguyên lý cốt lõi: **mỗi lớp chỉ đọc được header của chính nó**, coi mọi thứ bên trong là hộp đen.

Ví dụ với 1 request HTTPS:

```
Application data (HTTP request bytes)
  → TCP thêm header 20 bytes (src port, dst port, seq#, ack#, flags, window size)
    → IP thêm header 20 bytes (src IP, dst IP, TTL, protocol number)
      → Ethernet thêm header 14 bytes + trailer FCS 4 bytes (src MAC, dst MAC, EtherType)
```

Chuỗi khi gửi: `Data → Segment → Packet → Frame`
Chuỗi khi nhận (decapsulation): `Frame → Packet → Segment → Data`

Overhead cộng dồn ~58 bytes/frame — lý do MTU 1500 chỉ chở được ~1460 bytes data thật (gọi là **MSS – Maximum Segment Size**, TCP tự tính để tránh phải fragment).

Liên hệ 3 khái niệm quan trọng: **Layer → Protocol → PDU**
- Transport → TCP → Segment
- Internet → IP → Packet
- Network Access → Ethernet → Frame

---

## 6. Router đi qua bao nhiêu lớp?

- **Switch** chỉ hoạt động ở Layer 2 → không tháo IP header, chỉ nhìn MAC address để forward.
- **Router** hoạt động ở Layer 3 → phải tháo hẳn Ethernet frame ra (decapsulate tới Layer 3), đọc destination IP, quyết định routing, rồi tạo Ethernet frame mới hoàn toàn (encapsulate lại) với src/dst MAC khác để gửi tới next hop.

Nguyên tắc quan trọng: **IP giảm TTL mỗi hop nhưng giữ nguyên src/dst IP xuyên suốt hành trình**, còn **MAC address thay đổi ở mỗi hop** vì MAC chỉ có ý nghĩa cục bộ trong 1 network segment. TTL về 0 thì packet bị drop — đây là cơ chế `traceroute` lợi dụng để dò từng hop.

```
PC → Router → Router → Server
```
Mỗi đoạn Ethernet dùng một cặp src/dst MAC khác nhau, nhưng IP packet vẫn mang nguyên src IP → dst IP.

---

## 7. Vì sao vẫn cần OSI để troubleshoot

OSI giúp chia một vấn đề lớn thành từng lớp để tìm root cause, thường đi theo hướng bottom-up (rẻ tiền nhất để check trước):

1. **Physical** — đèn link có sáng không, cáp cắm đúng chưa
2. **Data Link** — interface up/up chưa, VLAN đúng chưa, MAC có được học không
3. **Network** — `ping` gateway được không, routing/default gateway đúng chưa
4. **Transport** — `telnet <ip> <port>` xem port có mở không (loại trừ firewall chặn)
5. **Application** — DNS resolve được chưa, service có đang chạy không

---

## 8. Layer violation trong thực tế

Nhiều hệ thống thực tế không tôn trọng ranh giới layer tuyệt đối:

- **Load balancer Layer 4 vs Layer 7**: L4 LB chỉ nhìn IP+port để route; L7 LB (HTTP load balancer) phải decapsulate tới tận Application layer để đọc URL path/header HTTP mới quyết định route — vì vậy L7 LB tốn CPU hơn nhiều.
- **TLS** về lý thuyết OSI nằm ở ranh giới Session/Presentation, nhưng dân dev thực tế hay gọi nó là "transport security" dù nó chạy trên TCP chứ không sửa đổi TCP.

---

## 9. Tổng kết

- Đừng chỉ nhớ "OSI = 7, TCP/IP = 4" — quan trọng hơn là hiểu **mối quan hệ nhóm lớp** giữa hai mô hình.
- **TCP/IP** cho biết thực tế nó chạy thế nào (giao thức thật, code thật).
- **OSI** cho biết khi có sự cố, cắt lát vấn đề ở đâu để tìm root cause nhanh nhất.
- Hiểu được dòng dữ liệu biến thành `Data → Segment → Packet → Frame` và ngược lại, các khái niệm tưởng rời rạc (TCP, UDP, Port, IP, Routing, MAC, VLAN, Ethernet) sẽ kết nối thành một hệ thống hoàn chỉnh.