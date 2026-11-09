# Hướng dẫn viết agent Wrongway

Bạn lập trình chiến thuật cho một quân cờ: đi từ hàng xuất phát tới hàng đối diện trước đối thủ. Arena đã cài luật, kiểm tra hành động, quản lý lượt và tổ chức thi đấu. Bài nộp là **một file Python `.py`** có hàm `choose_action(observation)`.

## 1. Chạy thử

Cần Python 3.9 trở lên và chỉ dùng thư viện chuẩn. Mở terminal tại thư mục `wrongway`. Trên macOS/Linux dùng `python3`; trên Windows thay bằng `py -3`:

```bash
python3 arena.py --player1 submissions/greedy.py --player2 submissions/wall_builder.py --render --log logs/demo.json
```

Có thể dùng tên file không có `.py`, ví dụ `--player1 greedy --player2 wall_builder`. Sao chép `submissions/TEMPLATE.py` thành `submissions/MSSV.py`, sửa `choose_action`, rồi thử với `--player1` và `--player2` để kiểm tra cả hai bên xuất phát.

## 2. Luật chơi

Bàn cờ có 9 hàng, 9 cột; `[row, col]` tăng từ trên xuống dưới và trái sang phải. Player 0 ở `[8,4]` đi tới hàng 0, player 1 ở `[0,4]` đi tới hàng 8. Luôn đọc `you["goal_row"]`, không mặc định mình đi lên.

Mỗi lượt chọn di chuyển, đặt tường hoặc đứng yên. Quân đi một ô ngang/dọc qua cạnh mở. Khi đối thủ đứng kề, có thể nhảy thẳng qua nếu phía sau còn mở; nếu bị tường hoặc biên chặn mới đi chéo vòng qua theo cạnh mở. `legal_moves` đã tính sẵn tất cả nước hợp lệ.

Mỗi người có 10 tường, số còn lại là `you["walls_remaining"]`. Tường `H` tại `(r,c)` chặn hai cạnh dọc `[r,c]-[r+1,c]` và `[r,c+1]-[r+1,c+1]`; tường `V` chặn hai cạnh ngang `[r,c]-[r,c+1]` và `[r+1,c]-[r+1,c+1]`. Neo từ 0 đến 7. Tường không được chồng, cắt hoặc chặn hết đường tới đích; `legal_walls` chỉ gồm tường được phép đặt.

Người đầu tiên tới hàng đích thắng. Sau tối đa 500 lượt chưa có người thắng thì hòa, trừ khi giảng viên đổi `--max-turns`.

## 3. Giao thức bài nộp

```python
def choose_action(observation):
    # Đọc trạng thái, lựa chọn chiến thuật rồi trả về dict hành động.
    return {"type": "move", "to": observation["legal_moves"][0]}
```

`observation` là một `dict` tương thích JSON và là bản chụp trạng thái. Các khóa gồm:

| Khóa | Ý nghĩa |
| --- | --- |
| `board_size`, `player_id`, `current_player` | Kích thước bàn cờ, số hiệu bạn và người tới lượt |
| `turn`, `max_turns` | Số hành động đã diễn ra và giới hạn lượt |
| `you`, `opponent` | `position`, `goal_row`, `walls_remaining` của hai bên |
| `walls` | Danh sách neo `horizontal` và `vertical` đã đặt |
| `legal_moves`, `legal_walls` | Nước đi và tường được phép ngay |
| `distances` | Ma trận 9 × 9 khoảng cách tới đích của bạn, xét tường, bỏ qua quân; `-1` là không tới được |
| `done`, `winner` | Trạng thái kết thúc và người thắng |

```python
# Di chuyển tới một ô trong observation["legal_moves"].
return {"type": "move", "to": [row, col]}

# Đặt một cấu hình trong observation["legal_walls"].
return {"type": "wall", "orientation": "H", "row": row, "col": col}

# Nhường lượt.
return {"type": "pass"}
```

Không cần tự viết vòng lặp trận đấu, đọc bàn phím hay cài luật kiểm tra. Mỗi lần chọn nước tối đa 1 giây; nạp module tối đa 5 giây. Lỗi, quá giờ hoặc hành động sai làm mất lượt. Biến toàn cục có thể giữ trạng thái trong ván nhưng có thể mất sau restart. Arena tạo seed cho `random`, nên có thể dùng `random.choice(...)`.

## 4. Bắt đầu từ template

`TEMPLATE.py` đã chạy được: duyệt `legal_moves` và chọn ô có khoảng cách tới đích nhỏ nhất. Hãy thử từng thay đổi nhỏ rồi quan sát log:

1. Khi đồng khoảng cách, ưu tiên cột gần giữa bàn cờ.
2. Tách hàm chấm điểm nước đi để dễ đọc và chỉnh sửa.
3. Chọn ngẫu nhiên giữa các nước đồng điểm bằng `random.choice`.
4. Xem `wall_builder.py` để học cách mô hình hóa cạnh bị chặn và BFS đường ngắn nhất.
5. Kết hợp di chuyển với đặt tường, rồi thử nhiều đối thủ, seed và cả hai bên xuất phát.

`distances` và BFS ví dụ bỏ qua quân đối thủ nên chỉ là ước lượng. Dùng `legal_moves` để chọn nước thực tế, nhất là khi hai quân đứng cạnh nhau. `wall_builder.py` là ví dụ đơn giản, không phải lời giải tối ưu.

## 5. Nộp bài

Nộp một file `.py` độc lập, ví dụ `MSSV.py`, có `choose_action(observation)` và trả về đúng `dict` theo giao thức. Dùng thư viện chuẩn, không phụ thuộc Arena hay dữ liệu máy cá nhân. Chạy thử cả khi là người chơi 0 và 1 trước khi nộp.
