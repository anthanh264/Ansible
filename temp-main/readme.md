## Ansible-Roles
- Các tác vụ liên quan đến nhau có thể được tập hợp lại thành role, sau đó áp dụng cho một nhóm các máy khi cần thiết
- Role Directory Structure
    + Task: Chứa các file yaml định nghĩa các nhiệm vụ chính khi triển khai.
    + Handles: Chứa các handler được sử dụng trong role
    + Files: chứa các file dc sử dụng bởi role, ví dụ như các file ảnh.
    + Templates: chứa các template file được sử dụng trong role, ví dụ như các file configuration... Các file này có đuôi *.j2, sử dụng jinja2 syntax
    + Vars: định nghĩa các variable được sử dụng ở trong roles
    + Defaults: Định nghĩa các giá trị default của các variable được sử dụng trong roles. Nếu variable không được định nghiã trong thư mục vars, các giá trị default này sẽ được gọi.
- Không nhất thiết phải sử dụng tất cả các thư mục ở trên khi tạo một role.
## Structure 
```
├── group_vars
│   └── all (File chứa các biến cho all các host khi play script)
├── roles
│   ├── demo_roles (folder role)
│   │   ├── defaults
│   │   │   └── main.yml (File chứa các biến mặc định)
│   │   ├── handlers
│   │   │   └── main.yml (File handler)
│   │   ├── tasks
│   │   │   └── main.yml (File chứa các task chính)
│   │   ├── templates
│   │   │   ├── 10-auth.conf.j2 (File template dùng trong task)
│   │   │   ├── 10-mail.conf.j2
│   │   └── vars 
│   │   │   └── main.yml (File chứa các biến sử dụng)
├── hosts (File chứa thông tin về các host)
└── site.yml (File cấu hình xem host nào chạy roles nào)
```
## Demo
- Ansible sử dụng rule `common` cho all host và chạy các roles config mailserver cho host `ubuntu`
- Cấu trúc thư mục như sau
```
├── group_vars
│   └── all
├── hosts
├── roles
│   ├── common
│   │   ├── defaults
│   │   └── tasks
│   │       └── main.yml
│   ├── config_dovecot
│   │   ├── defaults
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   ├── templates
│   │   │   ├── 10-auth.conf.j2
│   │   │   ├── 10-mail.conf.j2
│   │   │   ├── 10-master.conf.j2
│   │   │   ├── 10-master.j2
│   │   │   ├── 10-ssl.conf.j2
│   │   │   ├── auth-sql.conf.ext.j2
│   │   │   ├── dovecot.cf.j2
│   │   │   ├── dovecot.conf.j2
│   │   │   └── dovecot-sql.conf.ext.j2
│   │   └── vars
│   ├── config_mysql
│   │   ├── defaults
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   ├── templates
│   │   │   ├── data.sql.j2
│   │   │   └── schema.sql.j2
│   │   └── vars
│   ├── config_postfix
│   │   ├── defaults
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   ├── templates
│   │   │   ├── main.cf.j2
│   │   │   ├── master.cf.j2
│   │   │   ├── virtual-aliases.cf.j2
│   │   │   ├── virtual-domains.cf.j2
│   │   │   ├── virtual-email2email.cf.j2
│   │   │   └── virtual-users.cf.j2
│   │   └── vars
│   └── install_packages
│       ├── defaults
│       │   └── main.yml
│       └── tasks
│           └── main.yml
└── site.yml

```
