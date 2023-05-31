# Ansible 
Ubuntu Server 22.04


## Ansible là gì?
- Ansible là một công cụ automation<Tự động> và orchestration <Phân bổ> phổ biến, giúp cho chúng ta đơn giản tự động hóa việc triển khai ứng dụng. Nó có thể cấu hình hệ thống, deploy phần mềm
- Ansible có thể tự động hóa việc cài đặt, cập nhật nhiều hệ thống hay triển khai một ứng dụng từ xa
- Ansible là công cụ cho phép mình thao tác hàng loạt trên nhiều máy client
### Install 

```
sudo apt install software-properties-common
apt-add-repository -y ppa:ansible/ansible
apt-get update
apt-get install -y ansible
ansible --version
```

## Mô hình hoạt động
![](https://hackmd.io/_uploads/SkVky5bI3.png)

- Ansible kết nối với các máy con bằng SSH
- Inventory là tệp lưu địa chỉ các máy con trên. Chia thành các nhóm A-B để quản lí: Nhóm web server, nhóm database server
- Playbook là file chứa các task của Ansible được ghi dưới định dạng YAML. Máy Ansible sẽ đọc các task trong Playbook và đẩy các lệnh thực thi tương ứng bằng Python xuống các máy con.

## Thuật ngữ cơ bản 
- Controller Machine: Là máy cài Ansible, chịu trách nhiệm quản lý, điều khiển và gởi task tới các máy con cần quản lý.
- Inventory: Là file chứa thông tin các server cần quản lý. File này thường nằm tại đường dẫn /etc/ansible/hosts.
- Playbook: Là file chứa các task của Ansible được ghi dưới định dạng YAML. Máy controller sẽ đọc các task trong Playbook và đẩy các lệnh thực thi tương ứng bằng Python xuống các máy con.
- Task: Một block ghi tác vụ cần thực hiện trong playbook và các thông số liên quan. Ví dụ 1 playbook có thể chứa 2 task là: yum update và yum install vim.
- Module: Ansible có rất nhiều module, ví dụ như moduel yum là module dùng để cài đặt các gói phần mềm qua yum. Ansible hiện có hơn ….2000 module để thực hiện nhiều tác vụ khác nhau, bạn cũng có thể tự viết thêm các module của mình nếu muốn.
- Role: Là một tập playbook được định nghĩa sẵn để thực thi 1 tác vụ nhất định (ví dụ cài đặt LAMP stack).
- Play: là quá trình thực thi của 1 playbook
- Facts: Thông tin của những máy được Ansible điều khiển, cụ thể là thông tin về OS, network, system…
- Handlers: Dùng để kích hoạt các thay đổi của dịch vụ như start, stop service.


# Thực nghiệm 
1 Ubuntu Server cài Ansible <10.0.0.128>
1 Ubuntu 18.04 làm máy con <10.0.0.131>
< có thời gian sẽ làm thêm windows: Lí do vì Windows ko có sẵn SSH nên kết nối sẽ phức tạp hơn 1 chút >
![](https://hackmd.io/_uploads/rk1oYi-In.png)

## Cách Ansible kết nối với Client
- Ansible hoạt động theo cơ chế agentless, có nghĩa là không cần cài agent vào các máy client để điều khiển, thay vào đó ansible sẽ sử dụng việc điều khiển các client thông qua SSH. Do vậy, tới bước này ta có thể dùng 2 cách để ansible có thể điều khiển được các máy client.

    + Cách 1: Sử dụng usename, port của ssh để khai báo trong inventory. Các này không được khuyến cáo khi dùng trong thực tế vì việc password dạng bản rõ sẽ hiện thị, hoặc nếu dùng cách này thì cần phải bảo vệ file hosts
    + Cách 2: Sử dụng ssh keypair. Có nghĩa là ta sẽ tạo ra private key và public key trên node AnisbleServer và copy chúng sang các node client (hay còn gọi là các host). 

=> Chúng ta dùng cách 2

[Ansible_Server]
```
ssh-keygen
ssh-copy-id root@10.0.0.131

```

![](https://hackmd.io/_uploads/r1zKrq-Ln.png)
![](https://hackmd.io/_uploads/Hk59d9W82.png)

## Thêm Inventory
```
nano /etc/asible/hosts
```
[Tên khối]
Địa chỉ IP Client

```
[Ubuntu]
10.0.0.131
```

Check 
```
ansible all --list-hosts
```
```
root@ah:/etc/ansible# ansible all --list-hosts
  hosts (1):
    10.0.0.131
```

Ping
```
ansible ubuntu -m ping
```
```
10.0.0.131 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
### Note 
- Nếu gặp lỗi permission deny khi check => Trên máy client :
- In /etc/ssh/sshd_config, if the following line exists, possibly commented out (with a # in front):
``` PermitRootLogin without-password ```
- Then change it to the following, uncommenting if needed (remove the # in front):
```
PermitRootLogin yes
``` 
- And restart SSH:
```
sudo service ssh restart
```
## [Ad-Hoc Ansible command  ](https://docs.ansible.com/ansible/latest/command_guide/intro_adhoc.html#why-use-ad-hoc-commands)
- là các lệnh có thể được chạy riêng lẻ để thực hiện các chức năng nhanh chóng. 
Lệnh list all các máy con
```
ansible all --list-hosts
```
Lệnh ping tới máy con thuộc group ubuntu
```
ansible ubuntu -m ping
```
Cấu trúc chung lệnh ansible
```
ansible [tên host cần gọi] -m [tên module] -a [tham số truyền vào module]
-i : inventory host. Trỏ thư viện group_host cần gọi, mặc định nếu không có -i thì sẽ gọi /etc/ansible/hosts
-m : gọi module của ansible
-a : command
```
- Dưới đây là test thử các module cơ bản

### Sử dụng module [command](https://docs.ansible.com/ansible/2.5/modules/command_module.html#command-module)
- Cấu trúc
```
ansible all -m command -a "ten_cau_lenh_module_command_ho_tro"
```
Ví dụ 
- Check uptime của host thuộc group ubuntu `ansible ubuntu -m command -a "uptime"`
```
root@ah:~#  ansible ubuntu -m command -a "uptime"
10.0.0.131 | CHANGED | rc=0 >>
 10:52:41 up 56 min,  2 users,  load average: 0,00, 0,02, 0,24
```
- Lệnh ls trên $Home của máy Client `ansible all -m command -a "ls -alh"``
```
root@ah:~# ansible all -m command -a "ls -alh"
10.0.0.131 | CHANGED | rc=0 >>
total 40K
drwx------  7 root root 4,0K Thg 5 29 10:17 .
drwxr-xr-x 25 root root 4,0K Thg 3 17  2021 ..
drwx------  3 root root 4,0K Thg 5 29 10:17 .ansible
-rw-------  1 root root   66 Thg 5 29 10:07 .bash_history
-rw-r--r--  1 root root 3,1K Thg 4  9  2018 .bashrc
drwx------  2 root root 4,0K Thg 5 29 10:14 .cache
drwx------  3 root root 4,0K Thg 5 29 10:14 .gnupg
drwxr-xr-x  3 root root 4,0K Thg 5 29 10:06 .local
-rw-r--r--  1 root root  148 Thg 8 17  2015 .profile
drwx------  2 root root 4,0K Thg 5 29 10:14 .ssh
```

- Tham khảo thêm tại [Command](https://docs.ansible.com/ansible/2.5/modules/command_module.html#command-module)

### Module [setup](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/setup_module.html) trong ansible


- Kiểm tra các thông tin tổng quát về hệ điều hành của các client 
```
ansible all -m setup
```
- Liệt kê ra các địa chỉ IPv4 trên các máy con
```
ansible all -m setup -a 'filter=ansible_default_ipv4'
```

## Ansible PLAYBOOK <QUAN TRỌNG>

- Playbook Ansible chính là cách để sử dụng module hoàn thành một tác vụ. Nó là một tệp cấu hình được viết bằng YAML cung cấp các hướng dẫn về việc cần thực hiện trên các nút quản lý vào trạng thái mong muốn. Nó rất đơn giản, bạn có thể đọc được và tự ghi lại. 
- File PLAYBOOK sẽ có đuôi là `.yaml`
=> Ansible là file ghi lại một list dòng lệnh thực hiện liên tục một dạng script 

### YAML Cơ bản 
- Biến (VARIABLES)
    + Khai báo 
      ```
      vars:
        tên biến: "giá trị"
      ```
    + Gọi biến 
     ```{{ tên biến }}```
    + Ví dụ 
```
-
  name: Print author of playbook 
  hosts: ubuntu
  vars:
    author: "VtaeDha"
    country: VN
    title: "Systems Engineer"
  tasks:
    -
      name: Print author
      command: echo "This is written by {{ title }}{{ author }}"

    -
      name: Print author 's country
      command: echo "{‌{ country }}"
``` 
- Register
    + register giúp nhận kết quả trả về từ một câu lệnh. Sau đó ta có thể dùng kết quá trả về đó cho những câu lệnh chạy sau đó.
    + Ví dụ ta có bài toán như sau: kiểm tra trạng thái của service httpd, nếu start thất bại thì gửi mail thông báo cho admin.
    ```    
    #Sample ansible playbook.yml
    -
      name: Check status of service and email if its down
      hosts: localhost
      tasks:
        - command: service httpd status
          register: command_output

        - mail:
            to: Admins 
            subject: Service Alert
            body: "Service is down"
          when: command_output.stdout.find("down") != -1
    ```
- Điều kiện IF CONDITIONS 
    + Ví dụ
    ```
    -
  name: Toi da tot nghiep chưa
  hosts: localhost
  vars:
    age: 25
  tasks:
    -
      command: echo "Toi chua tot nghiep"
      when: age < 22          
    -                     
      command: echo "Toi da tot nghiep"                     
      when: age >= 22
    ``` 
- [LOOPS](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_loops.html) Vòng lặp ví dụng lệnh apt install <tên gói> trong trường hợp cài nhiều gói lệnh này sẽ phát huy tác dụng

    + Ví dụ 
```
---
- hosts: allone
  become: yes

  tasks:
    - name: Install Apache.
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - apache2
        - mysql-server
        - php
        - php-mysql
    - name: Restart Apache and Mysql
      service:
        name: "{{item}}"
        state:  running
      loop:
          - apache2
          - mysql
```
- Handlers giúp chúng ta gọi lại hành động thực thi nhiều lần (notify) mà không cần phải viết lại.
    + Ví dụ
    ```---
    - hosts: allone
      become: yes

    tasks:
        - name: Install Apache.
          apt:
            name: "{{ item }}"
            state: present
          loop:
            - apache2
            - mysql-server
            - php
            - php-mysql

        - name: deploy html file
          template:
            src: /tmp/index.html
            dest: /var/www/html/index.html
          notify: restart web

      handlers:
        - name: restart web
          service:
            name: "{{ item }}"
            state:  running
          loop:
          - apache2
          - mysql

    ```
- Fact và when: Câu lệnh rẽ nhánh
    + Ví dụ playbook tự phát hiện ubuntu hay centos để dùng apt hay yum tương ứng 
```
---
- hosts: allone
  become: yes

  tasks:
    - name: Define Red Hat.
      set_fact:
         package_name: "httpd"
      when:
         ansible_os_family == "Red Hat"

    - name: Define Debian.
      set_fact:
         package_name: "apache2"
      when:
         ansible_os_family == "Debian"

    - name: Stop apache
      service:
        name: "{{ package_name }}"
        state: stopped
```
### Viết playbook
- Tạo thư mục lưu playbook `mkdir /etc/ansible/playbook`
- Playbook đơn giản ping tới group ubuntu 
```
vi /etc/ansible/playbook/ping_ubuntu.yaml
```
```
---
- hosts: ubuntu
  tasks: 
    - name: Ping check host
      ping: ~
```
![](https://hackmd.io/_uploads/BJMzojbLn.png)

- Chạy thử playbook
```
ansible-playbook -i /etc/ansible/hosts /etc/ansible/playbook/ping_ubuntu.yaml
```
![](https://hackmd.io/_uploads/rJMZiobL3.png)
- Playbook cài đặt phần mềm trên client
```
vi /etc/ansible/playbook/install_demo.yaml
```
```
- hosts: ubuntu 
  become: true
  tasks:
  ##########
  - name: Install Apache2
    apt: name=apache2 state=latest
########### Khởi động lại apache 
  - name: Restart Apache2
    service: name=apache2 state=restarted
```
```
ansible-playbook -i /etc/ansible/hosts /etc/ansible/playbook/install_demo.yaml
```
![](https://hackmd.io/_uploads/rJ-R2j-Ln.png)
- Client check
![](https://hackmd.io/_uploads/SyuRhsZU3.png)

## Nâng cao
### [Ansible-Roles](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)
- Các tác vụ liên quan đến nhau có thể được tập hợp lại thành role, sau đó áp dụng cho một nhóm các máy khi cần thiết
- Role Directory Structure
    + Task: Chứa các file yaml định nghĩa các nhiệm vụ chính khi triển khai.
    + Handles: Chứa các handler được sử dụng trong role
    + Files: chứa các file dc sử dụng bởi role, ví dụ như các file ảnh.
    + Templates: chứa các template file được sử dụng trong role, ví dụ như các file configuration... Các file này có đuôi *.j2, sử dụng jinja2 syntax
    + Vars: định nghĩa các variable được sử dụng ở trong roles
    + Defaults: Định nghĩa các giá trị default của các variable được sử dụng trong roles. Nếu variable không được định nghiã trong thư mục vars, các giá trị default này sẽ được gọi.
    + Meta: thư mục này chứa meta data của roles
    + Note: Không nhất thiết phải sử dụng tất cả các thư mục ở trên khi tạo một role.
- Demo RULE
```
ansible-galaxy init __template__
```
![](https://hackmd.io/_uploads/ryftHn-8h.png)

- Chú ý bên trong thư mục phải tuân thủ việc khai báo tên file , tên folder cho role
```
roles/x/tasks/main.yml
roles/x/handlers/main.yml
roles/x/vars/main.yml
roles/x/defaults/main.yml
roles/x/meta/main.yml
```
- Role search path
     + Bạn phải khai báo việc set role chính xác trong ansible.cfg để ansible có thể hiểu được bạn viết role và thực thi nó.
![](https://hackmd.io/_uploads/HJUYrn-Uh.png)
- Using Roles
     + Bạn có thể sử dụng role theo cách sau .
     ```
     ---
     - hosts: dev
       roles:
         - nginx

     ```
- Demo
```
root@ah:~# cd /etc/ansible
root@ah:/etc/ansible# mkdir demo_ansible
root@ah:/etc/ansible# cd demo_ansible/
root@ah:/etc/ansible/demo_ansible# mkdir inventory
root@ah:/etc/ansible/demo_ansible# mkdir roles
root@ah:/etc/ansible/demo_ansible# cd roles/
root@ah:/etc/ansible/demo_ansible/roles# mkdir nginx
root@ah:/etc/ansible/demo_ansible/roles# cd nginx
root@ah:/etc/ansible/demo_ansible/roles/nginx# mkdir tasks
root@ah:/etc/ansible/demo_ansible/roles/nginx# cd ..
root@ah:/etc/ansible/demo_ansible/roles# cd nginx
root@ah:/etc/ansible/demo_ansible/roles/nginx# mkdir templates
root@ah:/etc/ansible/demo_ansible/roles/nginx# cd ..
root@ah:/etc/ansible/demo_ansible/roles# mkdir vars
```
- Cấu trúc thư mục 
```
root@ah:/etc/ansible/demo_ansible# tree
.
├── ansible.cfg
├── inventory
├── main-playbook.yml
└── roles
    ├── nginx
    │   ├── tasks
    │   │   └── main.yml
    │   └── templates
    │       └── nginx.conf.j2
    └── vars
        └── front.yml

6 directories, 5 files
root@ah:/etc/ansible/demo_ansible#
```

-  /roles/nginx/tasks/main.yml
```
- name: NGINX --> Install the nginx packages for Ubuntu target
  become: yes
  apt: 
    name: "nginx"
    update-cache: yes

  
- name: Check nginx status
  shell: bash -lc "systemctl status nginx"
  register: nginx_status
- debug: 
    var: nginx_status.stdout_lines

- name: NGINX --> Copy extra/sites configuration files
  become: yes
  template:
    src: nginx.conf.j2
    dest: "{{ nginx_conf_dir }}/nginx.conf"

```
- /roles/nginx/templates/nginx.conf.j2
```
user www-data;
worker_processes auto ;

error_log  {{ nginx_log_dir }}/error.log {{ nginx_error_log_level }};
pid        {{ nginx_pid_file }};

worker_rlimit_nofile {{ nginx_worker_rlimit_nofile }};

events {
    worker_connections {{ nginx_worker_connections }};
}

http {
	default_type  application/octet-stream;
	access_log  {{ nginx_log_dir }}/access.log;
	keepalive_timeout {{ keepalive_timeout }};
	send_timeout {{ send_timeout }};
	client_body_timeout {{ client_body_timeout }};
	client_header_timeout {{ client_header_timeout }};
	proxy_send_timeout {{ proxy_send_timeout }};
	proxy_read_timeout {{ proxy_read_timeout }};

	gzip {{ nginx_gzip }};
	gzip_types  text/css text/javascript application/javascript;

	include /etc/nginx/mime.types;
	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}

```
- vars/front.yml
```
nginx_conf_dir: "/etc/nginx"
nginx_server_name: "localhost"
nginx_service_name: "nginx"
nginx_user: "nginx"
nginx_group: "nginx"
nginx_pid_file: "/var/run/nginx.pid"
nginx_worker_connections: 1024
nginx_worker_rlimit_nofile: 1024
nginx_log_dir: "/var/log/nginx"
nginx_error_log_level: "error"
nginx_gzip: "on"
nginx_start_service: true
nginx_start_at_boot: true
keepalive_timeout: 600
send_timeout: 600
client_body_timeout: 600
client_header_timeout: 600
proxy_send_timeout: 600
proxy_read_timeout: 600

```
- main-playbook.yml
```
- hosts: ubuntu
  become_method: sudo
  gather_facts: True
  vars_files:
  - /roles/vars/front.yml
  roles:
  - nginx

```
- ansible.cfg
```
[defaults]
roles_path = ../roles

```
- RUN
```
ansible-playbook -i /etc/ansible/hosts main-playbook.yml --extra-vars
```
- Result 
```
root@ah:/etc/ansible/demo_ansible# ansible-playbook -i /etc/ansible/hosts main-playbook.yml

PLAY [ubuntu] *******************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************************************************
ok: [10.0.0.131]

TASK [nginx : NGINX --> Install the nginx packages for Ubuntu target] ***********************************************************************************************************************************************************************
ok: [10.0.0.131]

TASK [nginx : Check nginx status] ***********************************************************************************************************************************************************************************************************
changed: [10.0.0.131]

TASK [nginx : debug] ************************************************************************************************************************************************************************************************************************
ok: [10.0.0.131] => {
    "nginx_status.stdout_lines": [
        "● nginx.service - A high performance web server and a reverse proxy server",
        "   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)",
        "   Active: active (running) since Mon 2023-05-29 12:35:40 +07; 49s ago",
        "     Docs: man:nginx(8)",
        "  Process: 7941 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)",
        "  Process: 7940 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)",
        " Main PID: 7942 (nginx)",
        "    Tasks: 2 (limit: 1010)",
        "   CGroup: /system.slice/nginx.service",
        "           ├─7942 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;",
        "           └─7943 nginx: worker process",
        "",
        "Thg 5 29 12:35:40 web systemd[1]: Starting A high performance web server and a reverse proxy server...",
        "Thg 5 29 12:35:40 web systemd[1]: nginx.service: Failed to parse PID from file /run/nginx.pid: Invalid argument",
        "Thg 5 29 12:35:40 web systemd[1]: Started A high performance web server and a reverse proxy server."
    ]
}

TASK [nginx : NGINX --> Copy extra/sites configuration files] *******************************************************************************************************************************************************************************
changed: [10.0.0.131]

PLAY RECAP **********************************************************************************************************************************************************************************************************************************
10.0.0.131                 : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

root@ah:/etc/ansible/demo_ansible#
```
### [Ansible galaxy](https://galaxy.ansible.com/)
- Ansible Galaxy là một trang web miễn phí để tìm kiếm, tải xuống, xếp hạng và xem xét tất cả các tính chất được cộng đồng Ansible phát triển Tôi sẽ giới thiệu một số command để bạn có thể tải xuống , tạo mới hay quản lý roles/
- Install Role 
```
ansible-galaxy install geerlingguy.nginx
```
```
root@ah:/etc/ansible/demo_ansible# ansible-galaxy install geerlingguy.nginx
Starting galaxy role install process
- downloading role 'nginx', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-nginx/archive/3.1.4.tar.gz
- extracting geerlingguy.nginx to /etc/ansible/roles/geerlingguy.nginx
- geerlingguy.nginx (3.1.4) was installed successfully
root@ah:/etc/ansible/demo_ansible#
```
- Để xem danh sách các role được install 
```
ansible-galaxy list
```
- Create Role
```
ansible-galaxy init __template__
```
- Search for Roles
```
ansible-galaxy search apache --author geerlingguy
```
- Remove Role
```
ansible-galaxy remove username.role_name
```
## Bonus YAML
- Các kiểu biểu diễn giá trị
![](https://hackmd.io/_uploads/S1mzOjWLn.png)
- Key Value Pair (Cặp khoá vs giá trị): 
    + Dữ liệu được thể hiện bởi kiểu khoá và giá trị (key và value). Trong YAML, khóa và giá trị được phân tách bằng dấu hai chấm (:). Luôn phải có khoảng trắng theo sau dấu hai chấm.
- Mảng trong YAML: 
    + Các phần tử trong mảng sẽ được thể hiện bởi dấu gạch ngang ( - ). Cần có khoảng trắng trước mỗi mục. Số lượng khoảng trắng cần bằng nhau trước các phần tử của một mảng. Chúng ta hãy xem xét kỹ hơn về các dấu khoảng trắng trong YAML. 
    + Ví dụ ở đây ta có một object là Banana. Trong đó có 3 thuộc tính là calories, fat và carbs.
    ```
    Banana:
      Calories: 105
      Fat: 0.4 g
      Carbs: 27 g
    ```
    + Lưu ý số lượng khoảng trắng trước mỗi thuộc tính sẽ chỉ ra mối quan hệ cha con như trên thì 3 thuộc tính nằm trong Banana. Nhưng sẽ khác nếu như thêm khoảng trống ở các thuộc tính (Cái này giống Python)
    ```
    Banana:
      Calories: 105
        Fat: 0.4 g
        Carbs: 27 g
     ```
    + Lúc này fat và crabs sẽ là con của thuộc tính calories và calories là thuộc tính con của Banana
# Tham khảo
[Tìm hiểu Ansible - Viblo P1](https://viblo.asia/p/phan-1-tim-hieu-ve-ansible-4dbZNxv85YM)
[Tìm hiểu Ansible - Viblo P2](https://viblo.asia/p/phan-2-tim-hieu-ve-ansible-YWOZry8rKQ0)
[Tìm hiểu Ansible - Viblo P3](https://viblo.asia/p/tim-hieu-ansible-phan-3-yMnKMN0aZ7P)
[[10 phút ] [Ansible] [Cơ bản]](https://news.cloud365.vn/10-phut-ansible-co-ban-phan-2-dung-lab-de-thuc-hanh-ansible/)
[Ansible - Khái niệm cơ bản](https://blog.vietnamlab.vn/phan-1-ansible-khai-niem-co-ban/)
