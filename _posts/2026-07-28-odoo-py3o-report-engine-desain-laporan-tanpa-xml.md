---
layout: post
title: "Py3o Report Engine di Odoo: Desain Laporan & Print Out dengan LibreOffice — Tanpa QWeb XML"
date: 2026-07-28
description: Panduan implementasi Py3o Report Engine di Odoo. Desain laporan invoice, SO, payslip dengan LibreOffice Writer — tanpa QWeb XML. Cocok untuk Odoo Community & Enterprise.
canonical_url: "https://arisnew.odoo.com/blog/odoo-py3o-report-engine-desain-laporan-tanpa-xml"
robots: noindex, follow
---

Salah satu pain point terbesar saat kustomisasi Odoo adalah **desain laporan PDF**: invoice, quotation, delivery order, payslip, atau laporan operasional. Engine bawaan Odoo memakai **QWeb** (HTML + CSS + XML view) — powerful untuk developer, tetapi **sulit diakses pengguna bisnis** yang hanya ingin mengubah layout, logo, atau format tabel tanpa menyentuh kode.

**Py3o Report Engine** — modul open source dari [OCA/reporting-engine](https://github.com/OCA/reporting-engine) — menawarkan alternatif yang lebih ramah pengguna: desain laporan langsung di **LibreOffice Writer** atau **Calc**, dengan output PDF, DOCX, ODT, XLS, dan format lain yang didukung LibreOffice.

Artikel ini membahas cara kerja py3o, langkah implementasinya, dan workflow agar **desain report bisa dikerjakan pengguna fungsional** tanpa harus menulis custom QWeb XML.

## Masalah dengan Report QWeb Bawaan Odoo

Report standar Odoo (mis. `account.report_invoice`, `sale.report_saleorder`) dibangun dengan:

- **QWeb template** — sintaks XML/HTML khusus Odoo
- **CSS report** — styling terpisah, sering tricky untuk print layout A4
- **Python report parser** — untuk logika data kompleks

Konsekuensinya:

| Aspek | QWeb Report | Dampak |
| --- | --- | --- |
| Desain layout | Edit XML + CSS | Butuh developer |
| Preview | Tidak WYSIWYG penuh | Iterasi lambat |
| Format kompleks | Sulit (margin, header/footer, tabel dinamis) | Waktu development tinggi |
| Perubahan kecil | Redeploy module | Biaya maintenance |

Odoo Enterprise punya **Report Designer di Odoo Studio**, tetapi fitur itu **tidak tersedia di Community**. Py3o mengisi celah ini — terutama untuk tim Odoo Community yang butuh fleksibilitas desain laporan tanpa investasi Enterprise.

## Apa Itu Py3o Report Engine?

**Py3o** adalah reporting engine berbasis **LibreOffice/OpenDocument** yang diintegrasikan ke Odoo melalui modul **`report_py3o`**.

Alur kerjanya:

1. **Template** dibuat di LibreOffice Writer (`.odt`) atau Calc (`.ods`)
2. **Data Odoo** (record, relasi, fungsi helper) di-inject ke template via sintaks py3o
3. **Output** dihasilkan sebagai ODT/ODS native, atau dikonversi ke PDF/DOCX/XLS via LibreOffice di server

Keunggulan utama menurut [dokumentasi resmi OCA](https://github.com/OCA/reporting-engine/tree/18.0/report_py3o):

- **Full WYSIWYG** — desain report persis seperti dokumen Word
- **Tidak perlu jadi developer** untuk mengubah layout
- **Format A4/Letter** lebih natural di LibreOffice dibanding HTML/CSS
- **User bisa edit ulang** output ODT/DOCX setelah di-generate Odoo
- **Spreadsheet report** (ODS → XLS) untuk laporan tabular kompleks

Modul ini tersedia untuk Odoo Community **dan** Enterprise, dengan lisensi AGPL-3.

## Prasyarat Instalasi

### 1. Modul Odoo

Clone atau tambahkan repo OCA ke addons path:

```bash
git clone https://github.com/OCA/reporting-engine.git
# Install modul: report_py3o
```

Untuk Odoo 18, gunakan branch `18.0`. Modul juga tersedia di [OCA Apps Store](https://apps.odoo-community.org/modules/report_py3o).

### 2. Dependensi Python

```bash
pip install py3o.template py3o.formats
```

### 3. LibreOffice (untuk konversi PDF/DOCX)

Di server Odoo (Linux):

```bash
apt-get --no-install-recommends install libreoffice
```

Tanpa LibreOffice, report tetap bisa di-generate sebagai **ODT/ODS native** — tetapi tidak bisa dikonversi ke PDF.

### 4. (Opsional) Fusion Server untuk Performa

Modul `report_py3o` standalone akan **spawn proses LibreOffice per konversi** — lambat jika volume report tinggi. Untuk production dengan banyak PDF, pertimbangkan modul tambahan **`report_py3o_fusion_server`** yang menjalankan LibreOffice sebagai daemon.

## Workflow: Desain Report Tanpa QWeb XML

Inti value py3o: **pemisahan desain dan integrasi**.

```
┌─────────────────────┐     ┌──────────────────────┐     ┌─────────────────┐
│  Pengguna Bisnis    │     │  Developer (1x)      │     │  Odoo Server    │
│  LibreOffice Writer │────▶│  Daftarkan report    │────▶│  py3o engine    │
│  Desain layout ODT  │     │  + upload template   │     │  → PDF/ODT      │
└─────────────────────┘     └──────────────────────┘     └─────────────────┘
```

### Langkah 1: Developer — Setup Awal (Sekali)

Developer perlu:

1. Install modul `report_py3o`
2. Mendaftarkan report action (`ir.actions.report`) dengan `report_type = py3o`
3. Menentukan model Odoo yang di-report (mis. `account.move`, `sale.order`, `hr.payslip`)

Contoh XML minimal untuk mengganti report invoice bawaan:

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
  <record id="account.account_invoices" model="ir.actions.report">
    <field name="report_type">py3o</field>
    <field name="py3o_filetype">pdf</field>
    <field name="module">my_report_module</field>
    <field name="py3o_template_fallback">report/invoice_custom.odt</field>
  </record>
</odoo>
```

Atau menambah report baru (bukan replace):

```xml
<record id="report_partner_summary" model="ir.actions.report">
  <field name="name">Partner Summary</field>
  <field name="model">res.partner</field>
  <field name="report_name">res.partner.summary</field>
  <field name="report_type">py3o</field>
  <field name="py3o_filetype">pdf</field>
  <field name="module">my_report_module</field>
  <field name="py3o_template_fallback">report/partner_summary.odt</field>
  <field name="binding_model_id" ref="base.model_res_partner"/>
  <field name="binding_type">report</field>
</record>
```

**Catatan:** Setup XML ini **hanya sekali** — setelah itu, pengguna bisa mengubah desain tanpa redeploy module (lihat Langkah 3).

### Langkah 2: Pengguna — Desain Template di LibreOffice

Buka LibreOffice Writer dan buat dokumen A4. Sisipkan placeholder dinamis menggunakan sintaks py3o:

**Field sederhana:**

```
Invoice No: ${object.name}
Customer: ${object.partner_id.name}
Date: ${object.invoice_date}
Total: ${o_format_lang(object.amount_total, currency_obj=object.currency_id)}
```

**Loop baris invoice:**

```
for="line in object.invoice_line_ids"
┌──────────────┬─────────┬──────────┐
│ ${line.name} │ ${line.quantity} │ ${o_format_lang(line.price_subtotal)} │
└──────────────┴─────────┴──────────┘
/endfor
```

**Kondisi:**

```
if="object.state == 'posted'"
  ✓ CONFIRMED
/endif
```

**Logo perusahaan (gambar statis):**

```
staticimage.company_logo
```

**Alamat terformat:**

```
${display_address(object.partner_id)}
```

Fungsi helper bawaan py3o di Odoo antara lain:

| Fungsi | Kegunaan |
| --- | --- |
| `object` | Record utama yang di-print |
| `objects` | Semua record terpilih (multi-print) |
| `user` | User yang login |
| `lang` | Kode bahasa perusahaan |
| `o_format_lang()` | Format angka/mata uang sesuai locale |
| `o_format_date()` | Format tanggal sesuai locale |
| `display_address()` | Alamat partner terformat |
| `html_sanitize()` | Bersihkan HTML dari field rich text |
| `format_multiline_value()` | Field multiline dengan line break |

Dokumentasi lengkap sintaks templating: [py3o.template — Templating Guide](http://py3otemplate.readthedocs.io/en/latest/templating.html)

Simpan file sebagai `.odt` (Writer) atau `.ods` (Calc untuk laporan spreadsheet).

### Langkah 3: Upload Template — Tanpa Redeploy Module

Setelah setup awal, ada **tiga cara** update template tanpa menulis QWeb XML lagi:

#### Opsi A: Upload via Odoo UI (Paling Ramah Pengguna)

Modul `report_py3o` menyediakan model **`py3o.template`**:

1. Buka **Settings → Technical → Py3o Templates** (atau menu terkait)
2. Buat record baru, upload file `.odt`
3. Di **Settings → Technical → Reports**, pilih report action terkait
4. Set field **Template** (`py3o_template_id`) ke template yang baru di-upload

Template di database **memprioritaskan** file fallback di module — jadi pengguna bisa iterasi desain langsung dari UI Odoo.

#### Opsi B: Folder Template di Server (`root_tmpl_path`)

Untuk environment di mana pengguna mengedit file ODT langsung di server/shared folder:

```ini
# odoo.conf
[report_py3o]
root_tmpl_path=/odoo/templates/py3o
```

Letakkan file `.odt` di folder tersebut, lalu set `py3o_template_fallback` ke path absolut. Pengguna cukup **replace file ODT** — tanpa restart Odoo, tanpa redeploy.

#### Opsi C: Update File di Custom Module

Ganti file `.odt` di folder `report/` module, lalu upgrade module. Cocok untuk workflow Git-based, tetapi kurang fleksibel untuk pengguna non-teknis.

## Contoh Praktis: Custom Invoice Report

### Template ODT (cuplikan)

```
                    INVOICE
────────────────────────────────────────
No. Invoice  : ${object.name}
Tanggal      : ${o_format_date(object.invoice_date)}
Jatuh Tempo  : ${o_format_date(object.invoice_date_due)}
Customer     : ${object.partner_id.name}
${display_address(object.partner_id)}
────────────────────────────────────────

for="line in object.invoice_line_ids"
${line.name}    ${line.quantity}    ${o_format_lang(line.price_unit)}    ${o_format_lang(line.price_subtotal)}
/endfor

────────────────────────────────────────
Subtotal : ${o_format_lang(object.amount_untaxed, currency_obj=object.currency_id)}
PPN      : ${o_format_lang(object.amount_tax, currency_obj=object.currency_id)}
TOTAL    : ${o_format_lang(object.amount_total, currency_obj=object.currency_id)}
```

### Sample Template Siap Pakai

Proyek [odoo-py3o-report-templates](https://github.com/akretion/odoo-py3o-report-templates) menyediakan template ODT untuk report Odoo standar:

- Invoice (`account.move`)
- Sales Order (`sale.order`)
- Purchase Order (`purchase.order`)
- Delivery Order (`stock.picking`)
- Dan lainnya

Fork atau download sebagai starting point — edit layout di LibreOffice, sesuaikan field.

## Perbandingan: QWeb vs Py3o vs Odoo Studio

| Kriteria | QWeb (Bawaan) | Py3o (OCA) | Odoo Studio (Enterprise) |
| --- | --- | --- | --- |
| Target user desain | Developer | Pengguna bisnis + LibreOffice | Pengguna bisnis (UI) |
| Perlu XML? | Ya (template) | Tidak untuk desain; setup awal saja | Tidak |
| WYSIWYG | Terbatas | ✅ Penuh (LibreOffice) | ✅ Visual editor |
| Odoo Community | ✅ | ✅ | ❌ |
| Output format | PDF (wkhtmltopdf) | PDF, ODT, DOCX, XLS, dll. | PDF |
| Spreadsheet report | Sulit | ✅ Native (ODS/Calc) | Terbatas |
| Performa volume tinggi | Baik | Perlu fusion server | Baik |
| Lisensi | LGPL (Odoo) | AGPL-3 (OCA) | Enterprise |

**Rekomendasi:**

- **Odoo Community + butuh custom report fleksibel** → Py3o
- **Odoo Enterprise + perubahan layout sederhana** → Odoo Studio Report Designer
- **Logika report sangat kompleks / embedded chart** → QWeb atau kombinasi py3o + Python extender

## Tips Implementasi Production

1. **Pisahkan role** — Developer setup report action & akses; tim finance/HR desain template ODT
2. **Standarisasi naming** — `invoice_v2.odt`, `payslip_standard.odt` di folder terpusat
3. **Test multi-record** — aktifkan `py3o_multi_in_one` jika perlu satu PDF untuk banyak record
4. **Locale Indonesia** — gunakan `o_format_lang()` dan `o_format_date()` agar format Rupiah & tanggal benar
5. **Backup template** — simpan versi ODT di Git atau document management
6. **Monitor LibreOffice** — pastikan `libreoffice --headless` berjalan stabil di server; log error konversi
7. **Production scale** — deploy `report_py3o_fusion_server` jika >100 PDF/hari

## Kapan Tidak Cocok Pakai Py3o?

Py3o bukan silver bullet. Pertimbangkan alternatif (QWeb/custom module) jika:

- Report butuh **interaktif web preview** real-time di browser
- Layout sangat dinamis dengan **chart/grafik kompleks** embedded
- Infrastruktur server **tidak bisa install LibreOffice** (beberapa PaaS restricted)
- Kebutuhan **sub-second report generation** tanpa fusion server

## Py3o vs Aeroo

Py3o sering dibandingkan dengan [Aeroo Reports](https://github.com/aeroo-community/aeroo_reports) — keduanya LibreOffice-based, tetapi **template tidak interchangeable**. Py3o lebih aktif di ekosistem OCA modern (Odoo 16–19), sedangkan Aeroo lebih legacy di beberapa implementasi Odoo lama.

## Referensi & Dokumentasi

### Modul & Source Code

- [**OCA report_py3o**](https://github.com/OCA/reporting-engine/tree/18.0/report_py3o) — modul utama Py3o Report Engine
- [**OCA py3o.template**](https://github.com/OCA/py3o.template) — engine templating Python
- [**Sample templates**](https://github.com/akretion/odoo-py3o-report-templates) — invoice, SO, PO, picking
- [**Py3o templating docs**](http://py3otemplate.readthedocs.io/en/latest/templating.html) — sintaks lengkap for/if/image

### Artikel Terkait di Blog Ini

- [Odoo Studio: Kustomisasi ERP Tanpa Coding](https://arisnew.odoo.com/blog/blog-odoo-erp-software-development-1/odoo-studio-kustomisasi-erp-tanpa-coding-2) — alternatif Enterprise untuk kustomisasi UI & report
- [Odoo Project Management](https://arisnew.odoo.com/blog/blog-odoo-erp-software-development-1/odoo-project-management-panduan-lengkap-7) — contoh modul dengan kebutuhan reporting progress

### Layanan Implementasi

- [Halaman layanan Odoo](https://arisnew.odoo.com/our-services) — custom module & report development
- [Hubungi kami](https://arisnew.odoo.com/contactus) — konsultasi implementasi Py3o untuk Odoo Community

## Mulai dengan Py3o

Langkah praktis untuk memulai:

1. Install `report_py3o` + dependensi Python + LibreOffice di server staging
2. Download [sample template invoice](https://github.com/akretion/odoo-py3o-report-templates) sebagai starting point
3. Edit layout di LibreOffice Writer — ganti logo, header, tabel
4. Upload template via menu **Py3o Templates** di Odoo, atau deploy ke `root_tmpl_path`
5. Test print invoice dari Odoo — iterasi desain tanpa touch XML/QWeb
6. Rollout ke report lain (quotation, DO, payslip) dengan pola yang sama

Dengan Py3o, **desain report menjadi skill yang bisa dikuasai tim operasional** — bukan bottleneck developer. Setup teknis awal tetap perlu bantuan konsultan Odoo, tetapi setelah itu perusahaan bisa iterasi layout laporan secepat edit dokumen Word.

Butuh bantuan implementasi Py3o Report Engine, migrasi dari QWeb, atau setup fusion server untuk production? [Hubungi saya](https://arisnew.odoo.com/contactus) untuk konsultasi gratis.
