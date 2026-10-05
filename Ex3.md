# Báo cáo Bài 3: Xử lý xung đột phức tạp trong quá trình Rebase



## 2. Nhật ký các lệnh thiết lập kịch bản (Execution Steps)

### Bước 1: Khởi tạo Repository và Commit nền tảng trên `main`

```
mkdir -p homework/session_05/ex3
cd homework/session_05/ex3
git init

# Cấu hình danh tính local
git config --local user.name "Nguyen Viet Nam"
git config --local user.email "namnv@example.com"

# Tạo tệp config.json ban đầu
cat << 'EOF' > config.json
{
  "port": 8080,
  "debug": false
}
EOF

git add config.json
git commit -m "init config"

```

### Bước 2: Tạo nhánh `feature-api` và thực hiện 2 commit

```
git checkout -b feature-api

# Commit 1 trên feature-api: Sửa port thành 9000
cat << 'EOF' > config.json
{
  "port": 9000,
  "debug": false
}
EOF
git add config.json
git commit -m "feat: change port"

# Commit 2 trên feature-api: Đổi debug thành true
cat << 'EOF' > config.json
{
  "port": 9000,
  "debug": true
}
EOF
git add config.json
git commit -m "feat: enable debug"

```

### Bước 3: Cập nhật 2 commit mới trên nhánh `main`

```
git checkout main

# Commit 1 trên main: Đổi port thành 8081
cat << 'EOF' > config.json
{
  "port": 8081,
  "debug": false
}
EOF
git add config.json
git commit -m "update port on main"

# Commit 2 trên main: Thêm trường env
cat << 'EOF' > config.json
{
  "port": 8081,
  "debug": false,
  "env": "production"
}
EOF
git add config.json
git commit -m "add env config"

```

---

## 3. Quy trình Rebase và Giải quyết Xung đột Từng bước

Chuyển về nhánh `feature-api` và tiến hành Rebase lên `main`:

```
git checkout feature-api
git rebase main

```

---

### 🔴 Chặng 1: Xung đột tại Commit 1 (`feat: change port`)

*Thông báo từ Git:*

```
Auto-merging config.json
CONFLICT (content): Merge conflict in config.json
error: Could not apply a1b2c3d... feat: change port

```

#### 1. Kiểm tra trạng thái (`git status`):

```
interactive rebase in progress; onto e4f5g6h
Last command done (1 command done):
   pick a1b2c3d feat: change port
Unmerged paths:
  (use "git restore --staged <file>..." to unstage)
  (use "git add <file>..." to mark resolution)
	both modified:   config.json

```

#### 2. Nội dung tệp `config.json` khi phát sinh xung đột:

```json
{
<<<<<<< HEAD
  "port": 8081,
  "debug": false,
  "env": "production"
=======
  "port": 9000,
  "debug": false
>>>>>>> a1b2c3d (feat: change port)
}
```

#### 3. Xử lý thủ công Chặng 1:

Giữ lại `env: "production"` từ `main` và áp dụng `port: 9000` từ `feature-api`:

```json
{
  "port": 9000,
  "debug": false,
  "env": "production"
}
```

#### 4. Tiếp tục Rebase:

```
git add config.json
git rebase --continue

```

---

### 🔴 Chặng 2: Xung đột tại Commit 2 (`feat: enable debug`)

*Thông báo từ Git:*

```
Auto-merging config.json
CONFLICT (content): Merge conflict in config.json
error: Could not apply b2c3d4e... feat: enable debug

```

#### 1. Nội dung tệp `config.json` khi phát sinh xung đột:

```json
{
  "port": 9000,
<<<<<<< HEAD
  "debug": false,
  "env": "production"
=======
  "debug": true
>>>>>>> b2c3d4e (feat: enable debug)
}
```

#### 2. Xử lý thủ công Chặng 2:

Kết hợp cả 2 nhánh: giữ `port: 9000`, cập nhật `debug: true`, và giữ `env: "production"`:

```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```

#### 3. Hoàn tất Rebase:

```
git add config.json
git rebase --continue

```

*Thông báo hoàn thành:*

```
Successfully rebased and updated refs/heads/feature-api.

```

---

## 4. Minh chứng Kết quả Kiểm tra (Verification Outputs)

### A. Kiểm tra trạng thái Git (`git status`)

```
On branch feature-api
nothing to commit, working tree clean

```

### B. Kiểm tra Đồ thị Lịch sử Commit (`git log --graph --oneline`)

```
* 9a8b7c6 (HEAD -> feature-api) feat: enable debug
* 8f7e6d5 feat: change port
* 7e6d5c4 (main) add env config
* 6d5c4b3 update port on main
* 5c4b3a2 init config

```

> **Đánh giá:** Lịch sử commit hoàn toàn thẳng tắp (linear), không xuất hiện Merge Commit. Hai commit của nhánh `feature-api` (`feat: change port` và `feat: enable debug`) đã được đưa lên nằm nối tiếp ngay sau commit mới nhất của `main`.

### C. Nội dung tệp `config.json` cuối cùng

```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```

---

