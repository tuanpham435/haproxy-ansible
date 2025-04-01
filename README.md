# Ansible Role: HAProxy cho OpenShift

Role Ansible này triển khai và cấu hình HAProxy cho môi trường OpenShift, bao gồm cân bằng tải cho:
- API Server (cổng 6443)
- Machine Config Server (cổng 22623)
- Ingress Router (cổng 80 và 443)

## Yêu cầu

- Ansible 2.9+
- Hệ điều hành được hỗ trợ:
  - RHEL/CentOS 7, 8
  - Ubuntu 18.04, 20.04
  - Debian 10, 11

## Cấu trúc thư mục

```
haproxy-ansible/
├── ansible.cfg                 # Cấu hình Ansible
├── inventory/
│   └── hosts.ini               # File inventory
├── group_vars/
│   └── haproxy_servers.yml     # Biến cho nhóm haproxy_servers
├── roles/
│   └── haproxy/                # Role HAProxy
│       ├── defaults/           # Biến mặc định
│       ├── handlers/           # Handlers
│       ├── tasks/              # Tasks
│       ├── templates/          # Templates
│       ├── vars/               # Biến nội bộ
│       └── meta/               # Thông tin meta
└── site.yml                    # Playbook chính
```

## Biến cấu hình

Bạn có thể tùy chỉnh các biến sau trong `group_vars/haproxy_servers.yml` hoặc khi chạy playbook:

### Biến cơ bản

| Biến | Mặc định | Mô tả |
|------|----------|-------|
| `haproxy_bind_api_address` | "10.0.91.28" | Địa chỉ IP cho API và Machine Config Server |
| `haproxy_bind_ingress_address` | "10.0.91.29" | Địa chỉ IP cho Ingress Router |
| `control_plane_nodes` | [xem defaults/main.yml] | Danh sách các Control Plane nodes |
| `compute_nodes` | [xem defaults/main.yml] | Danh sách các Compute nodes |

### Biến nâng cao

| Biến | Mặc định | Mô tả |
|------|----------|-------|
| `enable_stats` | false | Bật/tắt trang thống kê HAProxy |
| `stats_port` | 9000 | Cổng cho trang thống kê |
| `stats_username` | "admin" | Tên đăng nhập cho trang thống kê |
| `stats_password` | "StrongPassword123" | Mật khẩu cho trang thống kê |
| `haproxy_max_connections` | 4000 | Số kết nối tối đa |
| `haproxy_timeout_connect` | "10s" | Timeout cho kết nối |
| `haproxy_timeout_client` | "1m" | Timeout cho client |
| `haproxy_timeout_server` | "1m" | Timeout cho server |
| `haproxy_manage_firewall` | true | Tự động cấu hình firewall |

Xem thêm các biến khả dụng trong `roles/haproxy/defaults/main.yml`.

## Cách sử dụng

### 1. Chuẩn bị

Clone repository và điều chỉnh cấu hình:

```bash
# Clone repository (nếu sử dụng git)
git clone <repository-url> haproxy-ansible
cd haproxy-ansible

# Hoặc tạo thủ công các thư mục và file
mkdir -p haproxy-ansible/{inventory,group_vars,roles/haproxy/{defaults,handlers,tasks,templates,vars,meta}}
# Sau đó tạo các file như trong hướng dẫn
```

### 2. Tùy chỉnh inventory

Chỉnh sửa file `inventory` để cấu hình máy chủ HAProxy:

```ini
[haproxy_servers]
haproxy.ocp-ssc2.amigo.vn ansible_host=10.0.91.27

[all:vars]
ansible_user=admin
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### 3. Tùy chỉnh biến

Chỉnh sửa file `group_vars/haproxy_servers.yml` để phù hợp với môi trường của bạn, đặc biệt là:
- `haproxy_bind_api_address`
- `haproxy_bind_ingress_address`
- `control_plane_nodes`
- `compute_nodes`

### 4. Chạy playbook

```bash
ansible-playbook site.yml
```

Hoặc với các tùy chọn bổ sung:

```bash
# Nhập mật khẩu sudo
ansible-playbook site.yml --ask-become-pass

# Chạy chỉ trên một máy chủ
ansible-playbook site.yml --limit haproxy.ocp-ssc2.amigo.vn

# Kiểm tra cấu hình mà không áp dụng (dry-run)
ansible-playbook site.yml --check

# Ghi đè biến khi chạy
ansible-playbook site.yml -e "enable_stats=true stats_password=SecurePassword123"
```

## Kiểm tra sau khi triển khai

Sau khi chạy playbook thành công, bạn có thể kiểm tra:

1. Dịch vụ HAProxy:
   ```bash
   ssh admin@10.0.91.27 'sudo systemctl status haproxy'
   ```

2. API Server:
   ```bash
   curl -k https://10.0.91.28:6443/version
   ```

3. Trang thống kê (nếu đã bật):
   ```
   http://10.0.91.27:9000/ (đăng nhập với thông tin trong cấu hình)
   ```