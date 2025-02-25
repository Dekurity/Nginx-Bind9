Berikut adalah langkah-langkah yang telah Anda sebutkan untuk mengkonfigurasi Nginx dan BIND9:

### Konfigurasi Nginx

1. **Instalasi Nginx**
    ```sh
    sudo apt update && sudo apt install nginx -y
    ```

2. **Konfigurasi Virtual Host**
    - Buat file konfigurasi baru:
      ```sh
      sudo nano /etc/nginx/sites-available/example.com
      ```
    - Tambahkan konfigurasi berikut:
      ```nginx
      server {
          listen 80;
          server_name example.com www.example.com;
          root /var/www/example.com;
          index index.html index.htm index.php;

          location / {
              try_files $uri $uri/ =404;
          }

          error_page 404 /404.html;
          location = /404.html {
              root /var/www/example.com;
          }

          location ~ \.php$ {
              include snippets/fastcgi-php.conf;
              fastcgi_pass unix:/run/php/php8.1-fpm.sock;
              fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
              include fastcgi_params;
          }
      }
      ```

3. **Membuat Direktori dan File Index**
    ```sh
    sudo mkdir -p /var/www/example.com
    echo "<h1>Website Aktif</h1>" | sudo tee /var/www/example.com/index.html
    sudo chown -R www-data:www-data /var/www/example.com
    ```

4. **Mengaktifkan Konfigurasi**
    ```sh
    sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
    sudo nginx -t
    sudo systemctl restart nginx
    ```

### Konfigurasi BIND9 (DNS Server)

1. **Instalasi BIND9**
    ```sh
    sudo apt update && sudo apt install bind9 -y
    ```

2. **Konfigurasi Zona DNS**
    - Edit file konfigurasi BIND:
      ```sh
      sudo nano /etc/bind/named.conf.local
      ```
    - Tambahkan konfigurasi zona berikut:
      ```sh
      zone "example.com" {
          type master;
          file "/etc/bind/db.example.com";
      };
      ```

3. **Membuat File Zona**
    - Buat file zona untuk example.com:
      ```sh
      sudo nano /etc/bind/db.example.com
      ```
    - Tambahkan isi berikut:
      ```sh
      $TTL 86400
      @   IN  SOA ns1.example.com. admin.example.com. (
              2025022501 ; Serial
              3600       ; Refresh
              1800       ; Retry
              604800     ; Expire
              86400      ; Minimum TTL
      )

      @       IN  NS  ns1.example.com.
      ns1     IN  A   192.168.1.1
      @       IN  A   192.168.1.2
      www     IN  A   192.168.1.2
      mail    IN  A   192.168.1.3
      ```

4. **Mengedit Konfigurasi BIND**
    - Edit file /etc/default/named (jika ada) atau /etc/default/bind9:
      ```sh
      sudo nano /etc/default/bind9
      ```
    - Pastikan opsi berikut ada:
      ```sh
      OPTIONS="-u bind -4"
      ```

5. **Restart BIND9**
    ```sh
    sudo systemctl restart bind9
    sudo systemctl enable bind9
    ```

6. **Pengujian Konfigurasi**
    - **Cek Syntax BIND9**
      ```sh
      sudo named-checkconf
      sudo named-checkzone example.com /etc/bind/db.example.com
      ```
    - **Cek Resolusi DNS**
      ```sh
      nslookup example.com 192.168.1.1
      dig example.com @192.168.1.1
      ```

Sekarang, Nginx dan BIND9 sudah dikonfigurasi dengan benar.
