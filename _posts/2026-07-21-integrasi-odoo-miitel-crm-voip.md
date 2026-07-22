---
layout: post
title: "Integrasi Odoo dengan MiiTel: CRM, VoIP, dan Voice Analytics dalam Satu Platform"
date: 2026-07-21
description: Panduan integrasi Odoo dengan MiiTel Phone System. Addon jsi_miitel untuk Odoo 18 — click-to-call, riwayat panggilan otomatis, notifikasi real-time. Dikembangkan Jidoka System Indonesia.
image: "https://img.youtube.com/vi/M02Meu6sdn8/maxresdefault.jpg"
og_image: "https://img.youtube.com/vi/M02Meu6sdn8/maxresdefault.jpg"
canonical_url: "https://arisnew.odoo.com/blog/blog-odoo-erp-software-development-1/integrasi-odoo-miitel-crm-voip-6"
robots: noindex, follow
odoo_path: /blog/blog-odoo-erp-software-development-1/integrasi-odoo-miitel-crm-voip-6
---

Tim sales dan customer service sering terjebak di antara dua dunia: **CRM** tempat data pelanggan disimpan, dan **aplikasi telepon** tempat percakapan sebenarnya terjadi. Setiap kali berpindah aplikasi, konteks hilang, follow-up terlambat, dan riwayat interaksi tidak lengkap.

**Integrasi Odoo dengan MiiTel** menutup celah itu. Addon [**jsi_miitel**](https://apps.odoo.com/apps/modules/18.0/jsi_miitel) — dikembangkan oleh [Jidoka System Indonesia](https://jidokasystem.co.id) — menghubungkan Odoo 18 CRM dengan [MiiTel Phone System](https://miitel.id), platform cloud telephony dan voice analytics berbasis AI. Hasilnya: panggilan masuk dan keluar langsung dari Odoo, riwayat tercatat otomatis, dan tim bisa fokus melayani pelanggan tanpa switching aplikasi.

## Apa Itu MiiTel?

**MiiTel** (dikembangkan oleh RevComm) adalah solusi *AI-powered business phone system* yang memungkinkan perusahaan:

- Melakukan panggilan telepon melalui **browser** (softphone) tanpa perangkat fisik tambahan
- Menganalisis **performa percakapan** — intonasi, tempo bicara, durasi, skor kualitas
- Meningkatkan produktivitas tim **sales** dan **customer service** dengan insight berbasis suara
- Mengintegrasikan data panggilan ke CRM, helpdesk, atau sistem bisnis lainnya

MiiTel sudah terintegrasi native dengan Salesforce, HubSpot, kintone, Slack, Zapier, dan layanan lain. Dengan addon **jsi_miitel**, integrasi serupa kini tersedia untuk ekosistem **Odoo ERP**.

## Addon jsi_miitel: Odoo MiiTel Integration

Modul [**jsi_miitel**](https://apps.odoo.com/apps/modules/18.0/jsi_miitel) (*Odoo MiiTel Integration*) adalah addon resmi di [Odoo Apps Store](https://apps.odoo.com/apps/modules/18.0/jsi_miitel) yang dikembangkan oleh **PT Jidoka System Indonesia**. Modul ini dirancang khusus untuk **Odoo 18** dan memanfaatkan [MiiTel JavaScript Widget](https://en.developers.miitel.com/docs/javascript-widget) serta webhook MiiTel untuk sinkronisasi data.

| Detail | Informasi |
| --- | --- |
| Technical name | `jsi_miitel` |
| Versi Odoo | 18.0 |
| Lisensi | OPL-1 |
| Developer | [Jidoka System Indonesia](https://jidokasystem.co.id) |
| Dependencies | CRM, Calendar, Contacts, Discuss |

## Fitur Utama Integrasi Odoo × MiiTel

### 1. Click-to-Call Langsung dari Odoo

Tim sales dan customer support dapat **melakukan panggilan keluar** langsung dari form kontak, lead, atau opportunity di Odoo CRM — tanpa membuka aplikasi telepon terpisah. Nomor telepon pelanggan yang sudah tersimpan di Odoo bisa langsung dipanggil dengan satu klik.

### 2. Terima Panggilan Masuk di Antarmuka Odoo

Widget MiiTel Phone ter-embed di Odoo sehingga agent dapat **menerima panggilan masuk** tanpa keluar dari CRM. Saat panggilan masuk, sistem dapat menampilkan informasi kontak yang cocok berdasarkan nomor telepon.

### 3. Riwayat Panggilan & Meeting Tercatat Otomatis

Setiap interaksi — panggilan masuk (*inbound*), panggilan keluar (*outbound*), dan **meeting** — tersimpan otomatis di profil pelanggan di Odoo CRM, lengkap dengan:

- Durasi panggilan
- Status (answered, missed, dll.)
- Timestamp
- Link ke analitik MiiTel (sequence ID)

Dokumentasi interaksi yang lengkap memudahkan follow-up, pelaporan kinerja tim, dan evaluasi pipeline sales.

### 4. Notifikasi Real-Time

Pengguna mendapat **pemberitahuan instan** saat ada panggilan masuk, sehingga respons ke pelanggan bisa lebih cepat — terutama penting untuk tim customer service dengan SLA ketat.

### 5. Voice Analytics Terintegrasi

MiiTel menganalisis percakapan secara otomatis. Hasil analisis — skor kualitas, tag, durasi — dapat diakses melalui MiiTel Analytics dan dikaitkan dengan aktivitas CRM di Odoo untuk insight yang lebih mendalam tentang performa agent dan kebutuhan pelanggan.

## Bagaimana Integrasi Bekerja?

Integrasi **jsi_miitel** memanfaatkan arsitektur yang didokumentasikan di [MiiTel Developers](https://en.developers.miitel.com/docs/getting-started):

```
┌─────────────────┐     JavaScript Widget      ┌──────────────────┐
│   Odoo CRM      │ ◄────────────────────────► │  MiiTel Phone    │
│  (jsi_miitel)   │   click-to-call, events    │  (Softphone)     │
└────────┬────────┘                            └────────┬─────────┘
         │                                                │
         │         Outgoing Webhook                       │
         │ ◄──────────────────────────────────────────────┘
         │         (call/meeting history)
         ▼
┌─────────────────┐
│  Odoo Chatter   │
│  & CRM Activity │
└─────────────────┘
```

**JavaScript Widget** — MiiTel Phone di-embed ke antarmuka Odoo. Event seperti `onReceiveCall`, `onCallEnd`, dan `onReceiveSequenceId` memicu aksi di CRM (popup kontak, log aktivitas, dll.). Detail lengkap: [JavaScript Widget](https://en.developers.miitel.com/docs/javascript-widget).

**Outgoing Webhook** — Setelah panggilan atau meeting selesai, MiiTel mengirim data ke Odoo via webhook sehingga riwayat tersimpan otomatis. Referensi: [Tutorial — Receive a FastAPI Outgoing Webhook](https://en.developers.miitel.com/docs/receive-a-fastapi-outgoing-webhook).

**MiiTel API** — Untuk kebutuhan lanjutan (export CSV riwayat panggilan, integrasi custom), MiiTel menyediakan REST API. Lihat [API Reference](https://en.developers.miitel.com/reference) di portal developer.

## Panduan Konfigurasi

Instalasi dan konfigurasi addon meliputi:

1. **Install modul `jsi_miitel`** di Odoo 18 (via Apps Store atau manual)
2. **Dapatkan embed code** dari MiiTel Admin → Third Party Integration → JavaScript Widget ([panduan resmi](https://en.developers.miitel.com/docs/javascript-widget))
3. **Konfigurasi Company ID dan Access Key** di pengaturan modul Odoo
4. **Setup Outgoing Webhook** di MiiTel Admin agar riwayat panggilan tersinkron ke Odoo
5. **Uji panggilan** — inbound dan outbound — dari form CRM

Video panduan konfigurasi step-by-step:

<div class="video-embed"><iframe src="https://www.youtube.com/embed/M02Meu6sdn8" title="YouTube video player" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen="" loading="lazy"></iframe></div>

**Video:** [Konfigurasi Integrasi Odoo MiiTel — jsi_miitel](https://www.youtube.com/watch?v=M02Meu6sdn8)

## Manfaat untuk Bisnis Indonesia

| Manfaat | Dampak |
| --- | --- |
| **Satu platform** | CRM + telepon + analytics tanpa switching aplikasi |
| **Produktivitas tim** | Click-to-call mengurangi waktu admin sebelum/sesudah panggilan |
| **Data terpusat** | Semua interaksi pelanggan terekam di Odoo pipeline |
| **Insight berbasis suara** | Voice analytics MiiTel untuk coaching dan QA tim |
| **Skalabilitas cloud** | Infrastruktur MiiTel cloud — tanpa PBX on-premise |
| **Odoo native** | Tidak perlu middleware pihak ketiga; data tetap di Odoo |

Integrasi ini sangat cocok untuk perusahaan di bidang **telemarketing**, **customer service**, **B2B sales**, **real estate**, **asuransi**, dan industri dengan volume panggilan tinggi.

## Kapan Butuh Integrasi Custom vs jsi_miitel?

| Kebutuhan | jsi_miitel (Ready-made) | Custom Development |
| --- | --- | --- |
| Click-to-call dari CRM Odoo | ✅ | Opsional |
| Log riwayat panggilan otomatis | ✅ | Perlu develop |
| Notifikasi panggilan masuk | ✅ | Perlu develop |
| Integrasi modul Odoo lain (Helpdesk, Project) | ⚠️ Perlu extend | ✅ Fleksibel |
| Logika bisnis khusus per industri | ⚠️ Terbatas | ✅ Diperlukan |

**Best practice:** Mulai dengan **jsi_miitel** untuk kebutuhan standar CRM + telepon. Jika kebutuhan berkembang (integrasi helpdesk, workflow approval, reporting khusus), [Jidoka System Indonesia](https://jidokasystem.co.id) dapat membantu extend modul sesuai proses bisnis Anda.

## Referensi & Dokumentasi

### Addon Odoo

- [**jsi_miitel — Odoo Apps Store**](https://apps.odoo.com/apps/modules/18.0/jsi_miitel) — halaman resmi modul di Odoo 18
- [**Integrasi CRM Odoo dengan MiiTel**](https://jidokasystem.co.id/blog/dokumentasi-odoo-3/post/integrasi-crm-odoo-dengan-miitel-497) — artikel Jidoka System Indonesia

### MiiTel Developers (Official)

- [**Getting Started**](https://en.developers.miitel.com/docs/getting-started) — portal developer MiiTel
- [**JavaScript Widget**](https://en.developers.miitel.com/docs/javascript-widget) — embed MiiTel Phone ke aplikasi web
- [**Tutorial — Open MiiTel Phone on React**](https://en.developers.miitel.com/docs/softphone-javascript-widget) — contoh implementasi widget
- [**Outgoing Webhook**](https://en.developers.miitel.com/docs/receive-a-fastapi-outgoing-webhook) — terima data riwayat panggilan/meeting
- [**Incoming Webhook**](https://en.developers.miitel.com/docs/use-python-to-import-audio-data-to-miitel) — import audio dari layanan telepon lain
- [**API Reference**](https://en.developers.miitel.com/reference) — endpoint REST API MiiTel

### MiiTel Product

- [**MiiTel Indonesia**](https://miitel.id) — informasi produk dan demo
- [**MiiTel Analytics**](https://account.miitel.jp/v1/signin) — login admin & analytics

## Mulai Integrasi Odoo × MiiTel

Langkah praktis untuk memulai:

1. Pastikan Odoo 18 Enterprise/Community dengan modul **CRM** aktif
2. Install [**jsi_miitel**](https://apps.odoo.com/apps/modules/18.0/jsi_miitel) dari Odoo Apps Store
3. Tonton [video konfigurasi](https://www.youtube.com/watch?v=M02Meu6sdn8) di atas
4. Ikuti [dokumentasi MiiTel JavaScript Widget](https://en.developers.miitel.com/docs/javascript-widget) untuk setup embed code
5. Uji alur panggilan inbound/outbound dari CRM Odoo

Butuh bantuan implementasi, konfigurasi webhook, atau kustomisasi integrasi MiiTel dengan modul Odoo lain? [Hubungi saya](https://arisnew.odoo.com/contactus) untuk konsultasi, atau kunjungi [halaman layanan](https://arisnew.odoo.com/our-services) dan [Jidoka System Indonesia](https://jidokasystem.co.id) untuk solusi Odoo ERP lengkap.
