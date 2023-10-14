Cấu trúc cơ bản 1 thư mục ansible playbook như này.
```
├── group_vars
├── hosts
├── roles
└── site.yml
```
- File hosts: Chứa thông tin các group và các ip của máy client 
    + Ví dụ: Group Ubuntu với Client 1 thuộc group trên
    ```
    [ubuntu]
    ubuntu_1 ansible_ssh_host=192.168.226.135
    ```


- File site.yml: Chứa cấu hình cho các host sẽ chạy role nào 
    + Ví dụ: hosts:all là all các group được list trong file `hosts` sẽ được execute role common
	  Group ubuntu sẽ dc execute roles config_mysql
	```
    ---
    - hosts: all
      roles:
      - role: common
    - hosts: ubuntu  # Define the hosts from your hosts file
      become: true
      roles:
        #- install_packages 
        - config_mysql
    ```
- Folder `group_vars` chứa file `all` file này chứa all các biến sử dụng trong quá trình execute cho all các group
- Folder `roles` chứa các role thực hiện: Hiểu các `roles` các nhiệm vụ cần thực hiện ví dụ cài apache, cài php ...
	+ Mỗi role có 1 folder tương ứng: Cấu trúc như sau với role `demo`
```
│   ├── demo_roles (folder role)
│   │   ├── defaults
│   │   │   └── main.yml (File chứa các biến mặc định)
│   │   ├── handlers
│   │   │   └── main.yml (File handler)
│   │   ├── tasks
│   │   │   └── main.yml (File chứa các task chính hiểu là các bước để thực hiện nhiệm vụ)
│   │   ├── templates
│   │   │   ├── 10-auth.conf.j2 (File template dùng trong task)
│   │   └── vars 
│   │   │   └── main.yml (File chứa các biến sử dụng)
```

