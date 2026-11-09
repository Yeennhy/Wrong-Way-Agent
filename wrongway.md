---
title: "Wrongway Arena"
short_title: "Wrongway"
label: arena-wrongway
description: "Đồ án xây dựng agent cho Wrongway Arena: điều khiển quân cờ trên bàn 9 × 9, di chuyển hoặc đặt tường, thi đấu vòng tròn qua hai giai đoạn nộp bài."
---

# Wrongway Arena

Trước khi bắt đầu, mời bạn trải nghiệm trò chơi tại [wrongway.app](https://wrongway.app), có thể chọn chế độ chơi với máy.

## 1. Giới thiệu

Wrongway Arena là sân chơi xây dựng chương trình điều khiển quân cờ, gọi là **agent**. Mỗi agent cần đưa quân cờ của mình tới bờ đối diện trước đối thủ và có thể đặt tường để làm chậm đường đi của đối thủ.

Sinh viên phát triển agent trong **một file Python duy nhất**, sau đó tham gia mùa giải giữa các bài nộp trong lớp. Arena đã cung cấp luật chơi, bàn cờ, danh sách hành động hợp lệ và cơ chế tổ chức trận đấu. Việc cần làm là đọc trạng thái, chọn hành động và cải thiện chiến thuật.

## 2. Arena

Arena tự động điều hành trận đấu giữa hai agent. Mỗi agent cung cấp `choose_action(observation)`: nhận trạng thái bàn cờ và trả về hành động muốn thực hiện.

### 2.1. Vòng lặp tương tác

Hai người chơi **thay phiên nhau**, mỗi lượt thực hiện một hành động:

```text
Khởi tạo bàn cờ và hai agent.
Trong khi trận đấu chưa kết thúc:
    1. Cung cấp trạng thái hiện tại cho agent đang tới lượt.
    2. Gọi choose_action(observation), giới hạn 1 giây.
    3. Nếu hành động hợp lệ, cập nhật bàn cờ.
       Nếu quá giờ, lỗi hoặc hành động sai, giữ nguyên bàn cờ.
    4. Tăng số lượt đã diễn ra.
    5. Kiểm tra người thắng và giới hạn tổng số lượt.
    6. Nếu trận đấu tiếp tục, chuyển lượt cho đối thủ.
```

Trạng thái cung cấp cho agent là bản sao; sửa `observation` không làm thay đổi bàn cờ thật.

### 2.2. Bàn cờ và điều kiện kết thúc

Bàn cờ có kích thước **9 × 9**. Một ô là `[row, col]`, trong đó hàng tăng từ trên xuống dưới, cột tăng từ trái sang phải; cả hai chỉ số thuộc `0` đến `8`.

Người chơi 0 bắt đầu tại `[8, 4]`, cần tới một ô bất kỳ ở hàng 0. Người chơi 1 bắt đầu tại `[0, 4]`, cần tới một ô bất kỳ ở hàng 8. Người chơi 0 đi trước. Người đầu tiên tới hàng đích thắng ngay. Nếu sau **500 lượt đơn** chưa có người thắng thì hòa; lượt đứng yên, quá giờ hoặc hành động sai vẫn được tính. Tọa độ không đổi khi đổi bên; đọc `you["goal_row"]` để biết đích của mình.

### 2.3. Các hành động

**Di chuyển.** Quân cờ đi một ô lên, xuống, trái hoặc phải qua cạnh không có tường. Không được ra ngoài bàn cờ hoặc đứng cùng ô với đối thủ. Khi đối thủ ở ô kề và không có tường ngăn giữa, có thể nhảy thẳng qua nếu ô phía sau còn trong bàn và cạnh sau không bị chặn. Nếu phía sau bị biên hoặc tường chặn, có thể đi chéo vòng qua đối thủ theo cạnh bên còn mở. Arena tính sẵn các đích đến trong `legal_moves`.

**Đặt tường.** Mỗi người bắt đầu với **10 tường**. Tường chặn hai cạnh ô liền nhau, có hướng ngang `H` hoặc dọc `V`, tiêu tốn một lượt và không thể thu hồi.

| Hướng | Vị trí neo `(row, col)` | Hai cạnh bị chặn |
| --- | --- | --- |
| `H` | `(r, c)` | `[r,c] - [r+1,c]` và `[r,c+1] - [r+1,c+1]` |
| `V` | `(r, c)` | `[r,c] - [r,c+1]` và `[r+1,c] - [r+1,c+1]` |

Neo có hàng và cột từ `0` đến `7`. Tường không được chồng lấn, cắt nhau hoặc chặn hết đường tới đích của bất kỳ người chơi nào. `legal_walls` chứa các tường hợp lệ ở lượt hiện tại.

**Đứng yên.** Trả về `{"type": "pass"}` để giữ nguyên vị trí và nhường lượt cho đối thủ.

### 2.4. Dữ liệu đầu vào và hành động trả về

Hàm `choose_action(observation)` nhận một `dict` gồm thông tin `you`, `opponent`, tường đã đặt và số lượt. Arena cũng cung cấp `legal_moves`, `legal_walls`, và `distances`: ma trận khoảng cách ngắn nhất tới đích **của mình**, xét tường và bỏ qua hai quân; `-1` nghĩa là không có đường.

```python
{"type": "move", "to": [row, col]}
{"type": "wall", "orientation": "H", "row": row, "col": col}
{"type": "pass"}
```

Với tường, `orientation` là `"H"` hoặc `"V"`; tọa độ là số nguyên. Phần tử trong `legal_walls` chưa có khóa `"type"`, nên cần thêm `"type": "wall"` khi trả về.

### 2.5. Bộ mã và chạy thử

Gói `wrongway-student.zip` gồm mã Arena, replay, mẫu nộp bài, các bot `greedy`, `random_agent`, `wall_builder` và tài liệu hướng dẫn. Cần Python 3.9 trở lên, chỉ sử dụng thư viện chuẩn. Sao chép `submissions/TEMPLATE.py` thành `submissions/MSSV.py`, viết chiến thuật trong `choose_action`, rồi chạy thử cả hai bên xuất phát. Sửa mã Arena trên máy cá nhân không thay đổi luật chấm.

## 3. Mùa giải và đánh giá

Mỗi giai đoạn nộp có một mùa giải vòng tròn. Mặc định mỗi cặp gặp nhau **hai ván đổi bên**, nên mỗi agent có một ván đi trước và một ván đi sau. Với `n` bài tham gia, mỗi agent chơi `2 × (n − 1)` ván và toàn mùa giải có `n × (n − 1)` ván.

| Kết quả | Điểm |
| --- | ---: |
| Thắng | 3 |
| Hòa | 1 |
| Thua | 0 |

Tổng điểm là `3 × số ván thắng + số ván hòa`. Xếp hạng theo tổng điểm giảm dần, sau đó xét số ván thắng; bằng cả hai chỉ số thì đồng hạng.

Đồ án được chấm trên thang **10 điểm**: 8 điểm cho file Python hoàn thiện đúng yêu cầu, 1 điểm cho xếp hạng mùa giải đầu và 1 điểm cho xếp hạng mùa giải sau. Điểm mỗi mùa giải là `(3 × số ván thắng + số ván hòa) / (3 × số ván đã đấu)`.

## 4. Nộp bài

Mỗi giai đoạn nộp trực tiếp **đúng một file `MSSV.py` trên Moodle**; không nộp báo cáo, ZIP, thư mục Arena, log hoặc file phụ. Ví dụ mã số `26127000` nộp `26127000.py`. File phải có `choose_action(observation)`, tự chạy độc lập, có thể chứa hàm hỗ trợ nhưng không phụ thuộc file riêng hoặc đường dẫn máy cá nhân.

Đồ án có hai giai đoạn liên tiếp, mỗi giai đoạn kéo dài một tuần. Chỉ sinh viên đã nộp ở giai đoạn 1 mới được nộp giai đoạn 2. Ngày giờ do giảng viên công bố trên Moodle; không nhận bài trễ.

## 5. Môi trường chấm và giới hạn tài nguyên

Môi trường chấm là Google Colab CPU cố định. Mỗi agent có tối đa **128 MB RAM** và **1 giây** cho mỗi lần chọn hành động; khởi động và nạp file có giới hạn riêng **5 giây**. Không áp dụng GPU. Arena không dành riêng số lõi CPU cho bot. Các trận hiện chạy tuần tự; chạy song song có thể làm tăng thời gian quyết định. Chưa quy định số luồng tối đa trong bot, nhưng bot tự chịu trách nhiệm nếu gặp lỗi.

## 6. Hạn chế của Arena

Bot vẫn chạy với quyền của tài khoản khởi động Arena. Môi trường chấm hiện không có cách ly OS/container, sandbox filesystem/network hoặc cưỡng chế giới hạn bộ nhớ cho mã bên ngoài. Trên POSIX, dừng bot sẽ dừng cả nhóm tiến trình; trên Windows, Arena dừng tiến trình bot trực tiếp.

Không khai thác các hạn chế này để tạo kết quả không công bằng. Bài vi phạm có thể bị loại khỏi mùa giải. Hãy báo lỗi cho giảng viên; có thể được cộng tối đa 10% điểm đồ án.

## 7. Ghi chú

- Giới hạn 1 giây gồm xử lý và giao tiếp với agent; thời gian Arena tính hành động hợp lệ và ma trận khoảng cách không tính vào giới hạn này.
- Quá giờ, lỗi chương trình hoặc hành động sai làm agent mất lượt; bàn cờ giữ nguyên và trận vẫn tiếp tục.
- Chỉ dùng thư viện chuẩn Python; không cần `requirements.txt`, không chờ nhập bàn phím hoặc thao tác thủ công.
- Có thể dùng biến toàn cục để ghi nhớ trong một ván, nhưng timeout hoặc restart sẽ làm mất bộ nhớ đó; agent phải vẫn hoạt động từ observation hiện tại.
- Công cụ AI được phép sử dụng. Sinh viên chịu trách nhiệm hiểu và giải thích bài nộp; không sao chép bài người khác hoặc gian lận kết quả.
