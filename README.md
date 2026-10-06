# Báo Cáo Thực Hành: Xử Lý Xung Đột Phức Tạp Trong Quá Trình Rebase

- **Bài tập**: Bài 3 - Xử lý xung đột phức tạp trong quá trình Rebase (Session 05 / Git Intermediate & Advanced Recovery)
- **Đường dẫn nộp bài**: `SS05/Bai03/README.md`
- **GitHub Remote Repository**: `https://github.com/DevOpsPTIT/SS05_Bai03.git`

---

## 1. Bối cảnh & Mục tiêu

### Bối cảnh
Khi làm việc nhóm trong mô hình Git Feature Branch Workflow, nhánh chính (`main`) liên tục tiếp nhận các commit mới từ các thành viên khác. Khi nhánh tính năng (`feature-api`) kéo các cập nhật mới nhất từ `main` về bằng kỹ thuật **Rebase** (`git rebase main`), Git sẽ tiến hành phát lại (replay) từng commit của nhánh tính năng lên trên đỉnh commit mới nhất của `main`. 

Nếu cả 2 nhánh cùng chỉnh sửa các dòng code trùng lặp trong cùng một tệp tin (ví dụ `config.json`), xung đột (conflict) sẽ xảy ra **từng chặng một tại từng commit** trong tiến trình Rebase. Việc hiểu và giải quyết xung đột từng bước bằng `git rebase --continue` là kỹ năng bắt buộc để giữ cho lịch sử commit của kho lưu trữ luôn thẳng hàng (Linear History) và sạch sẽ.

### Mục tiêu
- **Phân biệt cơ chế Rebase vs Merge**: Nắm vững lý do xung đột trong Rebase xảy ra theo từng commit nối tiếp thay vì 1 lần như Merge.
- **Thực hành quy trình giải quyết xung đột từng bước (Step-by-step resolution)**: 
  - Xử lý xung đột chặng 1 với commit `feat: change port`.
  - Xử lý xung đột chặng 2 với commit `feat: enable debug`.
- **Hoàn thiện cây lịch sử Git thẳng tắp**: Trích xuất `git log --graph --oneline` chứng minh không có Merge Commit thừa, các commit của `feature-api` được gắn nối tiếp ngay sau `main`.

---

## 2. Thông Số Cấu Hình & Ràng Buộc

| Thông số / Cấu hình | Giá trị lựa chọn / Thao tác | Ghi chú / Ràng buộc |
| :--- | :--- | :--- |
| **Thư mục bài tập** | `SS05/Bai03` | Repository quản lý cục bộ |
| **Tệp tin cấu hình** | `config.json` | Chứa tham số `port`, `debug`, `env` |
| **Nhánh chính** | `main` | Chứa 2 commit mới: `update port on main` & `add env config` |
| **Nhánh tính năng** | `feature-api` | Chứa 2 commit mới: `feat: change port` & `feat: enable debug` |
| **Lệnh thực thi rebase**| `git rebase main` | Rebase nhánh `feature-api` lên trên `main` |
| **Số lần phát sinh conflict** | 2 lần (2 chặng commit) | Xử lý lần lượt bằng `git rebase --continue` |
| **Kết quả mong đợi** | Cây Git thẳng tắp (Linear) | Không sử dụng `git merge` thông thường |
| **Remote Repository** | `https://github.com/DevOpsPTIT/SS05_Bai03.git` | Kho lưu trữ GitHub |

---

## 3. Bản Chất Xử Lý Xung Đột Trong Rebase vs Merge

### 1. Phân biệt Rebase Conflict vs Merge Conflict

| Tiêu chí | Git Merge (`git merge main`) | Git Rebase (`git rebase main`) |
| :--- | :--- | :--- |
| **Cách thức kết hợp** | Hợp nhất toàn bộ thay đổi của 2 nhánh trong **1 thao tác duy nhất** | Phát lại (replay) **từng commit một** của nhánh feature lên đỉnh `main` |
| **Tần suất Conflict** | Xảy ra tối đa **1 lần** duy nhất | Có thể xảy ra **nhiều lần** (tương ứng từng commit bị xung đột) |
| **Lệnh tiếp tục** | `git commit` (tạo Merge Commit) | `git add <file>` && `git rebase --continue` |
| **Hình dạng cây Git** | Phân nhánh & hội tụ (Tạo Merge Commit nút thắt) | **Thẳng tắp (Linear)**, không có Merge Commit thừa |

---

### 2. Sơ đồ minh họa quá trình Rebase tuyến tính

```
[Trạng thái rẽ nhánh ban đầu]
          C---D  (feature-api: feat: change port -> feat: enable debug)
         /
A---B---E---F    (main: update port on main -> add env config)

[Tiến trình Rebase step-by-step]
Step 1: Replay commit C ('feat: change port') lên F  --> CONFLICT 1! (Sửa conflict -> git rebase --continue)
Step 2: Replay commit D ('feat: enable debug') lên C' --> CONFLICT 2! (Sửa conflict -> git rebase --continue)

[Trạng thái hoàn thành Rebase]
A---B---E---F---C'---D'  (HEAD -> feature-api, main)
(Lịch sử hoàn toàn thẳng hàng, các commit C' và D' nối tiếp ngay sau F)
```

---

## 4. Các Bước Thực Hiện Chi Tiết

### Bước 1: Khởi tạo repository & Commit gốc trên `main`

```bash
git init
git branch -M main

# Tạo tệp config.json ban đầu
echo '{ "port": 8080, "debug": false }' > config.json
git add config.json
git commit -m "init config"
```

---

### Bước 2: Tạo nhánh `feature-api` và thực hiện 2 commit

```bash
git checkout -b feature-api

# Commit 1 trên feature-api: Đổi port 9000
echo '{ "port": 9000, "debug": false }' > config.json
git add config.json
git commit -m "feat: change port"

# Commit 2 trên feature-api: Bật debug true
echo '{ "port": 9000, "debug": true }' > config.json
git add config.json
git commit -m "feat: enable debug"
```

---

### Bước 3: Tạo 2 commit mới trên nhánh `main` (Gây rẽ nhánh)

```bash
git checkout main

# Commit 1 trên main: Đổi port 8081
echo '{ "port": 8081, "debug": false }' > config.json
git add config.json
git commit -m "update port on main"

# Commit 2 trên main: Thêm tham số env
echo '{ "port": 8081, "debug": false, "env": "production" }' > config.json
git add config.json
git commit -m "add env config"
```

---

### Bước 4: Bắt đầu Rebase & Giải quyết Conflict Chặng 1

Quay lại nhánh `feature-api` và tiến hành rebase lên `main`:

```bash
git checkout feature-api
git rebase main
```

*Kết quả Terminal*:
```console
Auto-merging config.json
CONFLICT (content): Merge conflict in config.json
Rebasing (1/2)error: could not apply 101ebff... feat: change port
```

Mở tệp `config.json` để kiểm tra các thẻ xung đột Git:

```json
{
<<<<<<< HEAD
  "port": 8081,
  "debug": false,
  "env": "production"
=======
  "port": 9000,
  "debug": false
>>>>>>> 101ebff (feat: change port)
}
```

**Thao tác giải quyết Conflict 1**: Giữ tham số `port: 9000` của nhánh feature và giữ `env: production` từ nhánh main:

```json
{
  "port": 9000,
  "debug": false,
  "env": "production"
}
```

Lưu tệp, đưa vào Staging Area và tiếp tục Rebase:

```bash
git add config.json
git rebase --continue
```

---

### Bước 5: Giải quyết Conflict Chặng 2 & Hoàn tất Rebase

Sau khi tiếp tục, Git áp dụng commit thứ hai `feat: enable debug` và phát sinh xung đột chặng 2:

```console
Auto-merging config.json
CONFLICT (content): Merge conflict in config.json
Rebasing (2/2)error: could not apply f5e531b... feat: enable debug
```

Nội dung thẻ xung đột trong `config.json`:

```json
{
  "port": 9000,
<<<<<<< HEAD
  "debug": false,
  "env": "production"
=======
  "debug": true
>>>>>>> f5e531b (feat: enable debug)
}
```

**Thao tác giải quyết Conflict 2**: Tích hợp cả `debug: true` và `env: production`:

```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```

Lưu tệp và hoàn tất Rebase:

```bash
git add config.json
git rebase --continue
```

*Kết quả Terminal*: `Successfully rebased and updated refs/heads/feature-api.`

---

### Bước 6: Đồng bộ nhánh `main` (Fast-forward merge)

Cập nhật con trỏ của nhánh `main` tiến lên vị trí mới nhất của `feature-api`:

```bash
git checkout main
git merge feature-api
```

---

## 5. Trích Xuất Log Terminal Chứng Minh (Verification Logs)

### 1. Kiểm tra cây lịch sử commit thẳng tắp (`git log --graph --oneline --all`)

```console
$ git log --graph --oneline --all
* 708c267 (HEAD -> main, feature-api) feat: enable debug
* 3f5adc0 feat: change port
* b3deeac add env config
* d4e2acc update port on main
* 4df98e5 init config
```

> **Đánh giá**: 
> - Lịch sử commit hoàn toàn thẳng tắp (Linear History).
> - Không hề có Merge Commit xuất hiện.
> - Hai commit của `feature-api` (`3f5adc0` và `708c267`) nằm nối tiếp ngay trên đỉnh các commit của `main` (`b3deeac`).

---

### 2. Kiểm tra nội dung tệp `config.json` cuối cùng

```console
$ Get-Content config.json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```

> **Đánh giá**: Tệp cấu hình chứa đầy đủ cả 3 thông số: `port: 9000` (từ feature), `debug: true` (từ feature) và `env: production` (từ main).

---

### 3. Kiểm tra trạng thái kho lưu trữ (`git status`)

```console
$ git status
On branch main
nothing to commit, working tree clean
```

---

## 6. Bộ Lệnh Điều Khiển Tiến Trình Rebase

Trong quá trình thực thi `git rebase`, nếu gặp sự cố hoặc muốn thay đổi phương án, lập trình viên có thể sử dụng các lệnh điều hướng sau:

| Lệnh | Công dụng |
| :--- | :--- |
| **`git rebase --continue`** | Tiếp tục tiến trình Rebase sau khi đã giải quyết xong conflict và `git add` |
| **`git rebase --skip`** | Bỏ qua commit hiện tại đang bị xung đột (không áp dụng thay đổi của commit đó nữa) |
| **`git rebase --abort`** | Hủy bỏ hoàn toàn tiến trình Rebase và đưa repository về trạng thái ban đầu trước khi gõ lệnh rebase |

---

## 7. Kết Luận & Bài Học Kinh Nghiệm

1. **Hiểu bản chất theo từng bước**: Rebase giải quyết xung đột theo từng commit phát lại. Cần xử lý kiên trì qua từng chặng (`git rebase --continue`) cho đến khi hoàn tất.
2. **Bảo toàn dữ liệu 2 bên**: Khi giải quyết conflict trong file cấu hình như `config.json`, cần cẩn trọng kết hợp hợp lý các giá trị mới từ cả nhánh `main` và nhánh `feature`.
3. **Lợi ích của cây Git thẳng (Linear History)**: Giúp cây lịch sử dự án cực kỳ rõ ràng, dễ dàng tra cứu bug bằng `git bisect` hoặc revert khi cần thiết.
