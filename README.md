Konfigurasi Nginx Dan Bind9


---

1. Konfigurasi Nginx

Instalasi Nginx

sudo apt update && sudo apt install nginx -y

Konfigurasi Virtual Host

Misalkan domain yang akan digunakan adalah example.com, buat file konfigurasi baru:

sudo nano /etc/nginx/sites-available/example.com

Tambahkan konfigurasi berikut:

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

Membuat Direktori dan File Index

sudo mkdir -p /var/www/example.com
echo "<h1>Website Aktif</h1>" | sudo tee /var/www/example.com/index.html
sudo chown -R www-data:www-data /var/www/example.com

Mengaktifkan Konfigurasi

sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx


---

2. Konfigurasi BIND9 (DNS Server)

Instalasi BIND9

sudo apt update && sudo apt install bind9 -y

Konfigurasi Zona DNS

Edit file konfigurasi BIND:

sudo nano /etc/bind/named.conf.local

Tambahkan konfigurasi zona berikut:

zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
};

Membuat File Zona

Buat file zona untuk example.com:

sudo nano /etc/bind/db.example.com

Tambahkan isi berikut:

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

Mengedit Konfigurasi BIND

Edit file /etc/default/named (jika ada) atau /etc/default/bind9:

sudo nano /etc/default/bind9

Pastikan opsi berikut ada:

OPTIONS="-u bind -4"

Restart BIND9

sudo systemctl restart bind9
sudo systemctl enable bind9

Pengujian Konfigurasi

1. Cek Syntax BIND9

sudo named-checkconf
sudo named-checkzone example.com /etc/bind/db.example.com


2. Cek Resolusi DNS

nslookup example.com 192.168.1.1
dig example.com @192.168.1.1




---

Sekarang, Nginx dan BIND9 sudah dikonfigurasi dengan benar.
