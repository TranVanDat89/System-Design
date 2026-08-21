# Internet
* Internet là một hệ thống thông tin toàn cầu, kết nối hàng tỷ thiết bị điện tử. Internet không phải một mạng duy nhất, mà là mạng của các mạng (network of networks).

# Kiển trúc mạng Internet (theo tầng)
* **Tier 1 ISP** (nhà mạng cấp 1): Các công ty viễn thông khổng lồ (AT&T, NTT, Lumen, Telia...) sở hữu hạ tầng backbone toàn cầu, bao gồm chính các tuyến cáp quang biển. Họ kết nối trực tiếp với nhau (peering) mà không phải trả phí cho ai — đây là "xương sống" của Internet.
* **Tier 2 ISP**: Nhà mạng khu vực/quốc gia (VD: Viettel, VNPT, FPT ở Việt Nam) — họ mua băng thông từ Tier 1, đồng thời cũng peering với nhau qua các IXP (Internet Exchange Point) — điểm trung chuyển nơi nhiều ISP kết nối chéo để trao đổi traffic nội địa nhanh hơn, đỡ phải đi vòng ra nước ngoài.
* **Tier 3 ISP**: Nhà mạng cung cấp trực tiếp tới người dùng cuối — đây là phần "lắp đặt internet, wifi".

# Cách Internet transfer data
Ví dụ truy cập web Mỹ từ Việt Nam:
* Request tới router nhà → router NAT địa chỉ private thành public.
* Request tới ISP Việt Nam (Viettel/VNPT/FPT).
* ISP đưa traffic ra trạm cập bờ cáp quang biển (Việt Nam có các tuyến như AAG, APG, SJC, AAE-1...).
* Tín hiệu đi dưới dạng ánh sáng trong sợi cáp quang xuyên đại dương — không phải sóng vô tuyến. Ánh sáng truyền với tốc độ ~2/3 tốc độ ánh sáng trong chân không (do chiết suất của lõi thủy tinh), và có các bộ khuếch đại quang đặt dưới đáy biển mỗi ~80km để duy trì tín hiệu trên quãng đường hàng ngàn km.
* Cập bờ ở Mỹ, đi qua ISP Mỹ, được định tuyến bằng giao thức BGP (Border Gateway Protocol) — đây là "GPS" của Internet, các router trao đổi bảng định tuyến để biết đường ngắn nhất/tốt nhất tới một dải IP nào đó.
* Tới server đích, xử lý, gửi response ngược lại theo đường tương tự (có thể khác đường đi do BGP chọn route khác nhau 2 chiều).