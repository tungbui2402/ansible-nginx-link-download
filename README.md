# ansible-nginx-link-download
## File ini host
```
[webservers]
10.0.9.203
```
## Nginx Create Link Download 1 file
```
---
- name: Install and configure Nginx to serve a file
  hosts: webservers
  become: yes
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Create directory to store file
      file:
        path: /var/www/html/files
        state: directory
        mode: '0755'

    - name: Copy test.zip to Nginx directory
      copy:
        src: test.txt
        dest: /var/www/html/files/test.txt
        mode: '0644'

    - name: Configure Nginx to serve the file with Content-Disposition header
      copy:
        dest: /etc/nginx/sites-available/default
        content: |
          server {
              listen 80;
              server_name 10.0.9.203;

              location / {
                  alias /var/www/html/files;
                  add_header Content-Disposition 'attachment; filename="test.txt"';
                  try_files /test.txt =404;
              }
          }

    - name: Restart Nginx to apply changes
      service:
        name: nginx
        state: restarted
```

## Nginx Create Link Download more than 1 files
nano cat nginx.yml
```
---
- name: Cài đặt và cấu hình Nginx
  hosts: webservers
  become: yes

  tasks:
    - name: Cài đặt Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Tạo thư mục chứa các file cần tải về
      file:
        path: /var/www/html/files
        state: directory
        mode: '0755'

    - name: Sao chép các file vào thư mục
      copy:
        src: "{{ item.src }}"
        dest: "/var/www/html/files/{{ item.dest }}"
        mode: '0644'
      with_items:
        - { src: 'test1.rar', dest: 'test1.rar' }
        - { src: 'test2.rar', dest: 'test2.rar' }
        - { src: 'test.rar', dest: 'test.rar' }

    - name: Cấu hình Nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/default

    - name: Khởi động lại Nginx
      service:
        name: nginx
        state: restarted
```

nano nginx.conf.j2
```
server {
    listen 80;
    server_name 10.0.9.203;

    location / {
        root /var/www/html;
        index index.html index.htm;
    }

    location /test1.rar {
        alias /var/www/html/files/test1.rar;
    }

    location /test2.rar {
        alias /var/www/html/files/test2.rar;
    }

    location /test.rar {
        alias /var/www/html/files/test.rar;
    }
}
```
