---
layout: post
title: Wordpress AWS Lightsail
---

Catatan cara install Wordpress pada AWS Ligthsail.
- Buat instance Ligthsail baru https://aws.amazon.com/id/lightsail/
- Login ke server dengan ssh, klik tombol **Connect using SSH**
- Dapatkan password dengan menjalankan perintah `cat $HOME/bitnami_application_password`
- Login ke WP-Admin dengan default user (user), gunakan password yang telah kita dapatkan dilangkah sebelumnya.
- Setting Static IP, caranya klik **Create static IP** pada networking .
- Setting domain, arahkan DNS domain kita ke IP tersebut (tambahkan A record di domain management kita).
- Setting custom domain di lightsail wp ini:
    `cd /opt/bitnami/apps/wordpress` 
    `sudo ./bnconfig --machine_hostname DOMAINKAMU.COM`
- Setting SSL Let's Encrypt, pastikan domain sudah mengarah ke IP public (static) aws kita, kemudian login ke terminal server lightsail
- ketikan perintah `sudo /opt/bitnami/bncert-tool` ikuti perintah selanjutnya.
- Setting URL Wordpress, caranya edit file `/opt/bitnami/apps/wordpress/htdocs/wp-config.php` (bisa dengan nano), sesuaikan URL pada 2 baris berikut:
    ```php
    define('WP_SITEURL', 'http://' . $_SERVER['HTTP_HOST'] . '/');
    define('WP_HOME', 'http://' . $_SERVER['HTTP_HOST'] . '/');
    // menjadi
    define('WP_SITEURL', 'https://DOMAINKAMU.COM/');
    define('WP_HOME', 'https://DOMAINKAMU.COM/');
    ```

Semoga bermanfaat.

Referensi:
- https://lightsail.aws.amazon.com/ls/docs/en_us/articles/amazon-lightsail-quick-start-guide-wordpress
- https://docs.bitnami.com/aws/faq/configuration/configure-custom-domain/
- https://docs.bitnami.com/aws/how-to/generate-install-lets-encrypt-ssl/

Salam.
