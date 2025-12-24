# AutoPostBloggerPro Gemini V3.5 (PySide6)

Ứng dụng desktop (Windows) mô phỏng UI + workflow của `autopost.php` để **tự động tạo nội dung bằng Gemini**, quản lý bài viết Blogger, và (tuỳ chọn) **tự tạo ảnh bằng Pollinations** rồi **upload ảnh** lên **Google Drive** *hoặc* **GitHub (jsDelivr CDN)** để nhúng trực tiếp vào nội dung bài viết trước khi đăng.

---

## Tính năng chính

### Blogger (Blogger API v3)
- OAuth Desktop (mở trình duyệt để đăng nhập)
- List / Search / Load / Update / Create / Publish bài viết
- Progress + Log + Pause/Resume + Stop
- Auto schedule đăng bài theo giờ cấu hình

### Gemini (Prompt / Title / Description / Labels)
- Tạo ngẫu nhiên nội dung + prompt theo ngôn ngữ VN / EN / CN / KR
- Output JSON fields: `title`, `description`, `prompt`, `labels`

### Ảnh (Pollinations)
- Tạo ảnh từ prompt qua Pollinations (không cần API key)
- Throttle an toàn (mặc định chờ ~16s giữa các request)

### Upload ảnh (chọn 1 trong 2)
1) **Google Drive** (Drive API)  
   Upload ảnh lên folder Drive → lấy link `uc?export=view&id=...` để nhúng vào Blogger
2) **GitHub + jsDelivr CDN** (V3.5) ✅  
   Upload ảnh lên GitHub repo public bằng GitHub API → lấy link CDN jsDelivr để nhúng vào Blogger

---

## Yêu cầu hệ thống
- Windows 10/11
- Python 3.10+ (khuyến nghị)
- Internet (đăng nhập OAuth + gọi API)

---

## Cấu trúc thư mục (tóm tắt)
- `main.py` : entrypoint
- `ui_main.py` : UI + workflow chính
- `blogger_client.py` : Blogger API
- `gemini_client.py` : gọi Gemini API
- `pollinations_client.py` : tạo ảnh từ prompt
- `drive_client.py` : upload Drive (tuỳ chọn)
- `github_client.py` : upload GitHub + tạo link jsDelivr (V3.5)
- `utils.py` : tiện ích load/save config, resource path
- `data/config.default.json` : cấu hình mặc định
- `credentials/client_secret.json` : Google OAuth client secret (bắt buộc cho Blogger)

---

## 1) Chuẩn bị Google OAuth (BẮT BUỘC cho Blogger)

1. Vào **Google Cloud Console** → tạo project (nếu chưa có)
2. Enable API:
   - **Blogger API** (bắt buộc)
   - **Google Drive API** (chỉ cần nếu bạn muốn upload ảnh lên Drive)
3. Create Credentials → **OAuth client ID** → chọn **Desktop app**
4. Tải file JSON (client secret) và đặt vào:
   - `credentials/client_secret.json`

> ⚠️ Lưu ý: Không commit / không publish file `client_secret.json`.

---

## 2) (Tuỳ chọn) Gemini API Key

- Bạn cần **Gemini API Key** để app tạo nội dung tự động bằng Gemini.
- Dán key trong **Settings → Gemini API Key**.

---

## 3) (Tuỳ chọn) Upload ảnh lên Google Drive

Nếu bạn chọn **Drive upload**:
- Bật trong Settings: **Tự tạo ảnh (Pollinations)** + **Upload Google Drive**
- Nhập **Google Drive Folder ID** (folder chứa ảnh)
- Sau đó bấm **Auth/Login** lại (vì cần Drive scope)

Link nhúng ảnh dạng:
`https://drive.google.com/uc?export=view&id=FILE_ID`

---

## 4) (Tuỳ chọn) Upload ảnh lên GitHub + jsDelivr CDN (V3.5)

### 4.1 Repo GitHub
- Repo nên là **Public** để jsDelivr truy cập CDN ổn định.
- Ví dụ repo của bạn: `https://github.com/chusuntv/images_ai`

### 4.2 Tạo GitHub Token (PAT)
PAT là “mật khẩu API” để app upload ảnh lên repo.

Cách tạo (khuyên dùng Fine-grained token):
1. GitHub → **Settings**
2. **Developer settings**
3. **Personal access tokens** → **Fine-grained tokens**
4. **Generate new token**
5. Chọn:
   - Resource owner: (tài khoản của bạn)
   - Repository access: chọn đúng repo (vd: `images_ai`)
   - Permissions:
     - **Contents: Read and write** (quan trọng)
6. Generate token → copy token (chỉ hiện 1 lần)

> ⚠️ Không chia sẻ token. Nếu lộ → Revoke và tạo token mới.

### 4.3 Cấu hình trong App
Vào **Settings → Image (Pollinations + Upload)**:
- ✅ Bật **Tự tạo ảnh (Pollinations)** (nếu muốn tạo ảnh)
- ✅ Tick **Upload ảnh bằng GitHub (jsDelivr)**
- Điền:
  - GitHub Token (PAT)
  - Owner: `chusuntv`
  - Repo: `images_ai`
  - Branch: `main`
  - Path prefix: `images` (hoặc bạn đặt tuỳ ý)

### 4.4 Link ảnh jsDelivr app dùng
App ưu tiên dùng URL bất biến theo **commit SHA** để tránh cache:
`https://cdn.jsdelivr.net/gh/<owner>/<repo>@<commit_sha>/<path>`

Ví dụ:
`https://cdn.jsdelivr.net/gh/chusuntv/images_ai@a1b2c3d4/images/2025/12/24/abc.jpg`

> Gợi ý: Dùng `@main` sẽ ngắn hơn nhưng có thể bị cache (ảnh mới không lên ngay).

---

## 5) Cài đặt môi trường (dev)

```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
