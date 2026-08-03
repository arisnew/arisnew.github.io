---
layout: post
title: "Filestore Odoo di AWS S3: Multi-Server dengan OCA fs_attachment & fs_attachment_s3"
date: 2026-08-03
description: Panduan Odoo filestore di AWS S3 dengan OCA fs_attachment & fs_attachment_s3. Arsitektur multi-server, konfigurasi Nginx, custom path modul/tanggal, dan best practice production.
image: "https://opengraph.githubassets.com/1/OCA/storage"
og_image: "https://opengraph.githubassets.com/1/OCA/storage"
canonical_url: "https://arisnew.odoo.com/blog/blog-odoo-erp-software-development-1/odoo-fs-attachment-s3-filestore-multi-server-11"
robots: noindex, follow
odoo_path: /blog/blog-odoo-erp-software-development-1/odoo-fs-attachment-s3-filestore-multi-server-11
---

Saat Odoo di-deploy dengan **arsitektur multi-server** — load balancer di depan, beberapa worker/app server di belakang — attachment yang disimpan di **filestore lokal** (`~/.local/share/Odoo/filestore/`) menjadi masalah: file upload di server A tidak bisa diakses dari server B.

Solusi klasik: NFS shared mount. Solusi modern: **object storage** seperti **AWS S3** — scalable, durable, dan native untuk cloud deployment.

Artikel ini membahas implementasi nyata menggunakan modul OCA [**fs_attachment**](https://github.com/OCA/storage/tree/18.0/fs_attachment) dan [**fs_attachment_s3**](https://github.com/OCA/storage/tree/18.0/fs_attachment_s3) — berdasarkan pengalaman production dengan S3, plus **custom naming path** berdasarkan modul dan tanggal agar bucket tetap terorganisir saat dibrowse langsung dari AWS Console.

## Masalah Filestore Standar Odoo

Odoo default menyimpan binary attachment di:

| Mode | Lokasi | Multi-server? |
| --- | --- | --- |
| `file` (default) | Disk lokal per server | ❌ Tidak shared |
| `db` | Kolom `db_datas` di PostgreSQL | ✅ Shared, tapi DB membengkak |

Filestore standar juga menggunakan **checksum SHA** sebagai nama file — misalnya `a1/b2/a1b2c3d4...` — sehingga identifikasi file dari filesystem/S3 console hampir mustahil tanpa query database.

```
                    ┌─────────────┐
  User upload ─────►│  Server A   │──► /filestore/abc123...  ✅ ada di A
                    └─────────────┘
                           ✗
                    ┌─────────────┐
  User download ───►│  Server B   │──► file not found ❌
                    └─────────────┘
```

**OCA fs_attachment** menyelesaikan dua masalah sekaligus: storage eksternal yang shared + filename yang meaningful.

---

## Arsitektur: Odoo + S3 + Nginx

```
┌──────────┐     ┌─────────────┐     ┌──────────────────────┐
│  Browser │────►│   Nginx     │────►│  Odoo (multi worker) │
└──────────┘     │  LB + proxy │     │  fs_attachment       │
                 └──────┬──────┘     └──────────┬───────────┘
                        │                         │
                        │ X-Accel-Redirect        │ read/write
                        ▼                         ▼
                 ┌──────────────────────────────────────────┐
                 │              AWS S3 Bucket               │
                 │  my-bucket/prod/account/2026/08/inv.pdf  │
                 └──────────────────────────────────────────┘
```

Alur serve file:

1. Browser request `/web/content/<id>`
2. Odoo generate response dengan header `X-Accel-Redirect` (via fs_attachment_s3)
3. Nginx proxy langsung ke S3 (signed URL jika bucket private)
4. Odoo worker **tidak** stream file — hemat RAM & CPU

---

## Modul OCA yang Dibutuhkan

| Modul | Fungsi | Link |
| --- | --- | --- |
| **fs_storage** | Abstraksi filesystem via [fsspec](https://filesystem-spec.readthedocs.io/) | [OCA/storage](https://github.com/OCA/storage/tree/18.0) |
| **fs_attachment** | Redirect `ir.attachment` ke external storage | [fs_attachment](https://github.com/OCA/storage/tree/18.0/fs_attachment) |
| **fs_attachment_s3** | S3-specific: signed URL, X-Accel-Redirect, mimetype | [fs_attachment_s3](https://github.com/OCA/storage/tree/18.0/fs_attachment_s3) |

Dependency Python: `fsspec`, `s3fs`, `boto3` (via s3fs).

Semua modul **AGPL-3**, maintained oleh [OCA](https://odoo-community.org) (Camptocamp, ACSONE).

---

## Perbedaan Filestore Odoo vs fs_attachment

| Aspek | Filestore standar | fs_attachment |
| --- | --- | --- |
| Nama file | Checksum SHA (opaque) | `<name>-<attachment_id>-<version>.<ext>` |
| Lokasi | Disk lokal server | S3, NFS, Azure, GCS, dll. via fsspec |
| Multi-server | Perlu NFS/shared disk | Native shared via object storage |
| Serve file | Odoo stream langsung | Streaming atau X-Sendfile/X-Accel-Redirect |
| URL fields | Tidak ada | `Internal URL` + `Filesystem URL` |
| API file | Standard | Method `attachment.open()` — IOBase, low memory |

Pattern filename dari [dokumentasi fs_attachment](https://github.com/OCA/storage/tree/18.0/fs_attachment):

```
invoice-12345-0.pdf
         ↑      ↑  ↑
       name    id version
```

---

## Konfigurasi AWS S3 — Step by Step

### 1. Install modul

```bash
# Clone OCA storage branch 18.0
git clone -b 18.0 https://github.com/OCA/storage.git
# Install: fs_storage, fs_attachment, fs_attachment_s3
```

Tambahkan ke `addons_path`, update apps list, install ketiga modul.

### 2. Buat FS Storage di Odoo

**Settings → Technical → Storage → File Storage → Create**

| Field | Contoh value |
| --- | --- |
| **Code** | `s3_prod` |
| **Protocol** | `s3` |
| **Directory Path** | `my-odoo-bucket/{db_name}` |
| **Options (JSON)** | `{"key": "AKIA...", "secret": "...", "client_kwargs": {"region_name": "ap-southeast-1"}}` |
| **Use As Default For Attachment** | ✅ |
| **Autovacuum GC** | ✅ (hapus orphan file di S3) |
| **Use X-Sendfile To Serve Internal Url** | ✅ |

Placeholder `{db_name}` didukung sejak **fs_attachment 18.0.2.2.0** — berguna jika satu bucket dipakai multi database.

### 3. Force DB untuk file kecil (performance)

Attachment kecil sebaiknya tetap di **database** agar list/kanban view cepat — terutama thumbnail image & assets JS/CSS.

Default rules (dari docs):

```json
{"image/": 51200, "application/javascript": 0, "text/css": 0}
```

Artinya: image < 50KB → DB; JS/CSS → selalu DB. Konfigurasi di field **Force DB For Default Attachment Rules** pada FS Storage.

### 4. Konfigurasi fs_attachment_s3

Modul [**fs_attachment_s3**](https://github.com/OCA/storage/tree/18.0/fs_attachment_s3) menambah opsi khusus S3 saat **Use X-Sendfile** aktif:

| Field | Rekomendasi |
| --- | --- |
| **S3 Uses Signed URL For X-Accel-Redirect** | ✅ untuk private bucket |
| **S3 Signed URL Expiration** | 30 detik (default) |

Format header X-Accel-Redirect untuk S3:

```
X-Accel-Redirect: /fs_x_sendfile/{scheme}/{host}/{path}?{signed_query}
```

Env var alternatif (server environment file):

```ini
[fs_storage.s3_prod]
s3_uses_signed_url_for_x_sendfile=True
s3_signed_url_expiration=30
```

---

## Konfigurasi Nginx untuk S3 X-Accel-Redirect

Tambahkan location block di Nginx ([contoh dari fs_attachment_s3 README](https://github.com/OCA/storage/tree/18.0/fs_attachment_s3)):

```nginx
location ~ ^/fs_x_sendfile/(.*?)/(.*?)/(.*) {
    internal;
    set $url_scheme $1;
    set $url_host $2;
    set $url_path $3;
    set $url $url_scheme://$url_host/$url_path;

    proxy_pass $url$is_args$args;
    proxy_set_header Host $url_host;
    proxy_ssl_server_name on;
}
```

Untuk storage non-S3 dengan code `my_storage`, rule alternatif:

```nginx
location /my_storage/ {
    internal;
    proxy_pass http://myserver.com;
}
```

**Penting:** block harus `internal` — hanya bisa diakses via X-Accel-Redirect dari Odoo, bukan direct public access.

---

## Custom Path: Organisasi Berdasarkan Modul & Tanggal

### Motivasi

Default fs_attachment sudah meaningful (`invoice-12345-0.pdf`), tapi saat browse **AWS S3 Console** atau audit backup, ribuan file flat dalam satu prefix tetap sulit dinavigasi.

Dari pengalaman implementasi production, kami menambahkan **custom logic pada path penyimpanan** agar struktur S3 menjadi:

```
s3://my-odoo-bucket/
└── prod/                          ← {db_name}
    └── account/                   ← modul (res_model)
        └── 2026/
            └── 08/
                └── invoice-12345-0.pdf
```

Contoh variasi per modul:

```
prod/account/2026/08/Bill-98765-0.pdf       ← vendor bill
prod/sale/2026/08/Quotation-54321-0.pdf     ← sale order PDF
prod/hr/2026/08/Contract-11111-0.pdf        ← HR document
prod/documents/2026/08/NDA-partner-0.pdf    ← Documents app
```

### Manfaat

| Manfaat | Detail |
| --- | --- |
| **Audit & compliance** | Tim finance bisa filter `account/2026/` langsung di S3 |
| **Lifecycle policy** | S3 Intelligent-Tiering / Glacier per prefix tahun |
| **Debugging** | Identifikasi file tanpa query SQL ke `ir_attachment` |
| **Backup seletif** | Sync prefix modul tertentu saja |

### Implementasi (custom module)

OCA fs_attachment mendukung `directory_path` dengan placeholder `{db_name}`. Untuk path dinamis per modul/tanggal, buat **thin custom module** yang override path computation — contoh konsep:

```python
# custom_fs_attachment_path/models/ir_attachment.py
from odoo import models, fields
from datetime import datetime

class IrAttachment(models.Model):
    _inherit = "ir.attachment"

    def _fs_attachment_path_parts(self):
        """Extend path: {model}/{YYYY}/{MM}/"""
        parts = super()._fs_attachment_path_parts()
        model = (self.res_model or "unknown").replace(".", "_")
        now = self.create_date or fields.Datetime.now()
        if isinstance(now, str):
            now = fields.Datetime.from_string(now)
        parts.extend([
            model,
            now.strftime("%Y"),
            now.strftime("%m"),
        ])
        return parts
```

> **Catatan:** method name di atas ilustratif — sesuaikan dengan hook yang tersedia di versi fs_attachment Anda. Intinya: inject `res_model` + `create_date` ke path sebelum filename.

Alternatif tanpa custom code: gunakan **Optimizes Directory Path** (checksum-based 2-level depth) bawaan OCA — cukup untuk distribusi file, tapi tidak human-readable per modul.

---

## Server Environment File (Production Best Practice)

Jangan hardcode credential S3 di database UI production. Gunakan **server environment** (`server_environment` OCA):

```ini
[fs_storage.s3_prod]
protocol=s3
directory_path=my-odoo-bucket/{db_name}
options={"key": "AKIA...", "secret": "...", "client_kwargs": {"region_name": "ap-southeast-1"}}
use_as_default_for_attachments=True
use_x_sendfile_to_serve_internal_url=True
s3_uses_signed_url_for_x_sendfile=True
s3_signed_url_expiration=30
autovacuum_gc=True
force_db_for_default_attachment_rules={"image/": 51200, "application/javascript": 0, "text/css": 0}
optimizes_directory_path=False
use_filename_obfuscation=False
```

Set juga system parameter:

```
ir_attachment.location = db
```

Ini mencegah attachment module icon dll. tersimpan di local filestore saat boot (modul load sebelum fs_attachment).

---

## Multi-Staging: Prod vs Staging

Tips dari [dokumentasi OCA](https://github.com/OCA/storage/tree/18.0/fs_attachment):

- Restore DB production ke staging → attachment lama tetap dibaca dari **S3 production** (read-only)
- Staging pakai **storage terpisah** sebagai default → attachment baru tidak menimpa production
- Konfigurasi staging storage: prefix berbeda (`staging/{db_name}/`) atau bucket terpisah

```
Production DB  ──read──►  s3://bucket/prod/
Staging DB     ──write──►  s3://bucket/staging/   (new files only)
              ──read───►  s3://bucket/prod/       (old files from backup)
```

---

## Checklist Go-Live

1. ✅ Bucket S3 private + IAM policy least-privilege (PutObject, GetObject, DeleteObject)
2. ✅ Install `fs_storage`, `fs_attachment`, `fs_attachment_s3`
3. ✅ Set `ir_attachment.location = db`
4. ✅ Konfigurasi FS Storage + Force DB rules
5. ✅ Nginx X-Accel-Redirect block tested
6. ✅ Signed URL untuk private bucket (`fs_attachment_s3`)
7. ✅ Test upload/download dari **setiap** app server di belakang LB
8. ✅ Autovacuum GC aktif (hemat biaya S3 storage)
9. ✅ Lifecycle policy S3 (opsional: archive `*/2024/` ke Glacier)
10. ✅ Monitoring: CloudWatch S3 request metrics + Odoo log untuk fs access errors

---

## Troubleshooting Umum

| Gejala | Penyebab | Solusi |
| --- | --- | --- |
| File not found di server B | Masih pakai local filestore | Aktifkan default FS storage |
| 502 saat download attachment | Nginx X-Accel config salah | Cek location block & signed URL |
| `Invalid bucket name` presigned URL | `directory_path` = `/` saja | Gunakan `{db_name}` prefix ([bugfix 18.0.1.2.1](https://github.com/OCA/storage/tree/18.0/fs_attachment_s3)) |
| Thumbnail lambat | Semua image ke S3 | Aktifkan Force DB rules untuk image kecil |
| Icon module hilang setelah deploy | Boot order attachment | Set `ir_attachment.location=db` |
| Staging hapus file production | Shared storage + GC | Storage terpisah, prod read-only di staging |

---

## Kapan Pakai S3 vs Alternatif?

| Storage | Cocok untuk |
| --- | --- |
| **AWS S3** | AWS-native deployment, Odoo on EC2/ECS, multi-AZ |
| **MinIO** | Self-hosted S3-compatible, on-premise |
| **NFS/GlusterFS** | Legacy infra, latency ultra-low |
| **Azure Blob / GCS** | fsspec support via fs_storage protocol |
| **DB only** | Instance kecil, < 10 user, attachment minimal |

Untuk Odoo **Odoo.sh**: filestore sudah managed — modul ini relevan untuk **self-hosted** dan **on-premise**.

---

## Referensi

### OCA Storage (Official)

- [**fs_attachment**](https://github.com/OCA/storage/tree/18.0/fs_attachment) — Base Attachment Object Store
- [**fs_attachment_s3**](https://github.com/OCA/storage/tree/18.0/fs_attachment_s3) — S3 extensions & X-Accel-Redirect
- [**OCA/storage repository**](https://github.com/OCA/storage/tree/18.0) — semua modul storage
- [**Try on Runboat**](https://runboat.odoo-community.org/builds?repo=OCA/storage&target_branch=18.0) — test modul tanpa install lokal

### Dokumentasi Teknis

- [fsspec — filesystem spec](https://filesystem-spec.readthedocs.io/)
- [Nginx X-Accel-Redirect](https://www.nginx.com/resources/wiki/start/topics/examples/x-accel/)
- [AWS S3 IAM best practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

### Artikel terkait

- [REST API di Odoo dengan FastAPI](https://arisnew.odoo.com/blog/blog-odoo-erp-software-development-1/odoo-fastapi-rest-api-integrasi-10) — integrasi multi-system lainnya
- [Odoo Project Management](https://arisnew.odoo.com/blog/blog-odoo-erp-software-development-1/odoo-project-management-panduan-lengkap-7) — deployment tim & infrastruktur

### Layanan implementasi

- [Jidoka System Indonesia](https://jidokasystem.co.id) — Odoo deployment, DevOps, custom storage module
- [Hubungi saya](https://arisnew.odoo.com/contactus) — konsultasi arsitektur multi-server Odoo + AWS

---

## Kesimpulan

**OCA fs_attachment + fs_attachment_s3** adalah solusi mature untuk masalah filestore multi-server Odoo. Kombinasi **S3 sebagai shared storage**, **X-Accel-Redirect via Nginx**, dan **custom path modul/tanggal** memberikan infrastruktur yang scalable sekaligus mudah diaudit — tanpa NFS single point of failure.

Mulai dari modul OCA standar, lalu extend path naming sesuai kebutuhan organisasi Anda. Jangan migrate production database ke S3 storage sebelum checklist go-live above ✅ semua teruji di staging.

Butuh bantuan setup Odoo multi-server di AWS, custom fs_attachment path module, atau migrasi filestore existing ke S3? [Hubungi saya](https://arisnew.odoo.com/contactus) untuk konsultasi.
