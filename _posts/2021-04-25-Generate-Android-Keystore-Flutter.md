---
layout: post
title: Generate Android Keystore Flutter
---

Catatan cara generate keystore Android (Flutter). Sebelumnya kami mengalami masalah `Invalid keystore format`.
- Masuk ke direktori keystore android, jika belum tahu bisa cek dengan menggunakan perintah `flutter doctor -v`, lihat pada bagian `Java binary at: `.
- Misal di Mac : `cd /Applications/Android\ Studio.app/Contents/jre/jdk/Contents/Home/bin`
- Kemudian jalankan perintah `keytool -genkey -v -keystore ~/LOKASI/FOLDER/TUJUAN/NAMA_KEY.jks -keyalg RSA -storetype JKS -keysize 2048 -validity 10000 -alias key`
- Kita menggunakan format lama (JKS) jika `-storetype JKS` disertakan.


Semoga bermanfaat.

Referensi:
- https://flutter.dev/docs/deployment/android#signing-the-app

Salam.
