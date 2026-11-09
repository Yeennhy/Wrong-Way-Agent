# Wrongway Arena

**Đồ án môn Tư duy tính toán**

## 1. Giới thiệu

Wrongway Arena giúp sinh viên làm quen Python bằng cách xây dựng chương trình điều khiển quân cờ, gọi là **agent**. Agent cần tới bờ đối diện trước đối thủ và có thể đặt tường để làm chậm đối thủ.

Đây là đồ án **cá nhân**. Mỗi sinh viên phát triển một agent trong **một file Python duy nhất**, sau đó tham gia mùa giải giữa các bài nộp của lớp. Arena đã có luật chơi, bàn cờ, danh sách hành động hợp lệ và cơ chế tổ chức trận đấu. Sinh viên tập trung đọc trạng thái, chọn hành động và cải thiện chiến thuật.

## 2. Arena

Mỗi agent cung cấp `choose_action(observation)`: nhận trạng thái bàn cờ và trả về hành động. Hai người chơi thay phiên nhau; mỗi lượt Arena đưa trạng thái cho agent, gọi hàm với giới hạn 1 giây, áp dụng nước hợp lệ hoặc giữ nguyên bàn cờ nếu quá giờ/lỗi/nước sai, tăng lượt, kiểm tra thắng hoặc giới hạn lượt, rồi đổi bên. `observation` là bản sao nên sửa nó không thay đổi bàn cờ thật.

### 2.1. Bàn cờ và hành động

Bàn cờ 9 × 9; tọa độ là `[row, col]`, từ 0 đến 8. Người chơi 0 xuất phát `[8, 4]` và đi tới hàng 0; người chơi 1 xuất phát `[0, 4]` và đi tới hàng 8. Người chơi 0 đi trước. Không lật tọa độ khi đổi bên; đọc `you["goal_row"]`.

Quân cờ đi một ô ngang/dọc qua cạnh không bị tường chặn. Khi đối thủ đứng kề, có thể nhảy qua nếu ô phía sau còn mở; nếu bị biên hoặc tường chặn, có thể đi chéo vòng qua đối thủ theo cạnh mở. Các đích đi hợp lệ có trong `legal_moves`.

Mỗi người bắt đầu với 10 tường. Tường ngang `H` tại `(r,c)` chặn `[r,c]-[r+1,c]` và `[r,c+1]-[r+1,c+1]`; tường dọc `V` chặn `[r,c]-[r,c+1]` và `[r+1,c]-[r+1,c+1]`. Neo tường từ 0 đến 7. Không được chồng, cắt tường hay chặn mọi đường tới đích của một người. `legal_walls` chỉ chứa tường hợp lệ ở lượt hiện tại. Có thể đứng yên bằng `{"type": "pass"}`.

Người đầu tiên tới hàng đích thắng ngay. Mặc định sau 500 lượt mà chưa thắng thì hòa; lượt đứng yên, quá giờ hoặc sai vẫn được tính.

### 2.2. Giao thức

`observation` là `dict` chứa `board_size`, `player_id`, `current_player`, `turn`, `max_turns`, `you`, `opponent`, `walls`, `legal_moves`, `legal_walls`, `distances`, `done`, và `winner`. `distances` là ma trận khoảng cách ngắn nhất tới đích của mình, xét tường và bỏ qua quân cờ; `-1` là không tới được.

```python
{"type": "move", "to": [row, col]}
{"type": "wall", "orientation": "H", "row": row, "col": col}
{"type": "pass"}
```

Với tường, tự thêm `"type": "wall"` vì phần tử trong `legal_walls` không có khóa này.

## 3. Chạy thử, mùa giải và nộp bài

Gói `wrongway-student.zip` gồm Arena, replay, template và ba bot ví dụ. Dùng Python 3.9 trở lên và thư viện chuẩn; sao chép `submissions/TEMPLATE.py` thành `submissions/MSSV.py`, rồi thử ở cả hai bên xuất phát. Sửa Arena cục bộ không thay đổi luật chấm.

Mỗi giai đoạn có mùa giải vòng tròn. Mỗi cặp mặc định đấu hai ván đổi bên. Thắng được 3 điểm, hòa 1 điểm, thua 0 điểm. Xếp hạng theo tổng điểm, sau đó số ván thắng. Đồ án 10 điểm: 8 điểm file hoàn thiện, 1 điểm xếp hạng giai đoạn 1, 1 điểm xếp hạng giai đoạn 2.

Nộp đúng một file `MSSV.py` trên Moodle, không nộp báo cáo, ZIP, log hay file phụ. File phải định nghĩa `choose_action(observation)` và tự chạy trong một file. Có hai giai đoạn một tuần liên tiếp; chỉ người đã nộp giai đoạn 1 được nộp giai đoạn 2, và không nhận bài trễ.

## 4. Lưu ý

- Mỗi lần chọn nước tối đa 1 giây; nạp module có giới hạn riêng 5 giây. Quá giờ, lỗi hay nước sai làm mất lượt.
- Chỉ dùng thư viện chuẩn; không dùng nhập bàn phím hoặc thao tác thủ công.
- Có thể dùng biến toàn cục trong một ván, nhưng timeout hoặc restart có thể làm mất bộ nhớ.
- Công cụ AI được phép. Sinh viên phải hiểu bài, có thể giải thích chiến thuật, và không sao chép/gian lận.
