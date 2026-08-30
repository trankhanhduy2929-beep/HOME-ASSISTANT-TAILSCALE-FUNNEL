# HƯỚNG DẪN TRUY CẬP HOME ASSISTANT TỪ XA BẰNG TAILSCALE FUNNEL

Không cần mở port router
Không cần mua tên miền
Không cần DuckDNS
Không cần bật VPN/Tailscale trên điện thoại khi truy cập
Có HTTPS miễn phí với tên miền `*.ts.net`

---

## 1. Cài Tailscale trên Home Assistant

Vào:

Settings → Apps/Add-ons → Tailscale

Cài Tailscale Community App/Add-on.

Sau khi cài:

* Start on boot: ON
* Watchdog: ON
* Start Tailscale

Mở Web UI của Tailscale và đăng nhập tài khoản Tailscale.

Sau khi đăng nhập thành công, Home Assistant sẽ xuất hiện trong danh sách Machines trên Tailscale.

Ví dụ:

```text
homeassistant
homeassistant-me-loan
```

---

## 2. Bật MagicDNS và HTTPS trên Tailscale

Truy cập Tailscale Admin Console.

Vào:

```text
DNS
```

Bật:

```text
MagicDNS
HTTPS Certificates
```

Sau đó máy Home Assistant sẽ có tên miền dạng:

```text
homeassistant-me-loan.xxxxx.ts.net
```

Ví dụ:

```text
homeassistant-me-loan.tailfa12e.ts.net
```

Tailscale sẽ tự cấp chứng chỉ HTTPS.

Không cần Let's Encrypt.

---

## 3. Cho phép sử dụng Funnel

Vào:

```text
Tailscale Admin Console
→ Access controls
```

Trong ACL phải có:

```json
"nodeAttrs": [
  {
    "target": ["autogroup:member"],
    "attr": ["funnel"]
  }
]
```

Ví dụ ACL cơ bản:

```json
{
  "grants": [
    {
      "src": ["*"],
      "dst": ["*"],
      "ip": ["*"]
    }
  ],

  "nodeAttrs": [
    {
      "target": ["autogroup:member"],
      "attr": ["funnel"]
    }
  ]
}
```

Save lại.

Nếu máy Tailscale đang dùng tag riêng thì cần cấp quyền `funnel` cho tag đó thay vì chỉ dùng `autogroup:member`.

---

## 4. Cấu hình Home Assistant cho Reverse Proxy

Vào:

```text
Settings
→ System
→ Network
→ HTTP server
```

Bật:

```text
Trust X-Forwarded-For
```

Trong:

```text
Trusted proxies
```

thêm:

```text
127.0.0.1
```

Nếu trước đó đã dùng Nginx Proxy Manager hoặc reverse proxy khác thì giữ nguyên các Trusted Proxy cũ và chỉ thêm:

```text
127.0.0.1
```

Không xóa cấu hình cũ.

Sau đó Restart Home Assistant.

---

## 5. Bật Funnel trong Tailscale Add-on

Vào:

```text
Settings
→ Apps/Add-ons
→ Tailscale
→ Configuration
```

Quan trọng nhất là:

```yaml
share_homeassistant: funnel
share_on_port: '443'
```

Một cấu hình gọn để chỉ sử dụng remote Home Assistant:

```yaml
accept_dns: true
accept_routes: false

advertise_connector: false
advertise_exit_node: false
advertise_routes: []
advertise_tags: []

always_use_derp: false

log_level: info
log_upload: false

login_server: https://controlplane.tailscale.com

share_homeassistant: funnel
share_on_port: '443'

services: []

snat_subnet_routes: true
stateful_filtering: false

taildrive:
  local_apps: false
  app_configs: false
  backup: false
  config: false
  media: false
  share: false
  ssl: false

taildrop: false
userspace_networking: false
```

Save.

Sau đó Restart Tailscale.

---

## 6. Lấy tên miền truy cập Home Assistant

Vào:

```text
Tailscale Admin
→ Machines
→ chọn máy Home Assistant
```

Tìm:

```text
Full domain
```

Ví dụ:

```text
homeassistant-me-loan.tailfa12e.ts.net
```

URL truy cập sẽ là:

```text
https://homeassistant-me-loan.tailfa12e.ts.net
```

Không cần thêm:

```text
:8123
```

Không cần:

```text
:443
```

---

## 7. Test từ Internet

Trên điện thoại:

```text
Tắt Wi-Fi
↓
Dùng 4G/5G
↓
Tắt Tailscale trên điện thoại
↓
Mở trình duyệt
```

Truy cập:

```text
https://homeassistant-me-loan.xxxxx.ts.net
```

Nếu xuất hiện trang đăng nhập Home Assistant là thành công.

Điều này chứng minh:

```text
Internet
↓
Tailscale Funnel
↓
Home Assistant
```

mà không cần:

```text
Port Forwarding
VPN trên điện thoại
Public IPv4
DuckDNS
Tên miền riêng
Nginx Proxy Manager
```

---

# Mô hình hoạt động

```text
Điện thoại dùng 4G/5G
Không bật Tailscale
        │
        ▼
https://homeassistant.xxxxx.ts.net
        │
        ▼
Tailscale Funnel
        │
        ▼
Tailscale Add-on
        │
        ▼
Home Assistant
```

Router không cần mở:

```text
80
443
8123
```

CGNAT vẫn sử dụng được.

---

# Lưu ý bảo mật

Tailscale Funnel khác với Tailscale VPN.

Khi dùng Funnel, URL Home Assistant được public ra Internet.

Vì vậy nên:

* Bật MFA/2FA cho tài khoản Home Assistant
* Sử dụng mật khẩu mạnh
* Không dùng tài khoản Admin chung cho nhiều người
* Giữ Home Assistant và Tailscale luôn cập nhật

---

# Tóm tắt cấu hình

Home Assistant:

```text
Trust X-Forwarded-For: ON

Trusted proxies:
127.0.0.1
```

Tailscale ACL:

```json
"nodeAttrs": [
  {
    "target": ["autogroup:member"],
    "attr": ["funnel"]
  }
]
```

Tailscale Add-on:

```yaml
share_homeassistant: funnel
share_on_port: '443'
```

Kết quả:

```text
https://homeassistant.xxxxx.ts.net
```

✅ Miễn phí
✅ HTTPS
✅ Không mua domain
✅ Không DuckDNS
✅ Không mở port router
✅ Không cần IP public
✅ CGNAT vẫn chạy
✅ Điện thoại bên ngoài không cần bật Tailscale
