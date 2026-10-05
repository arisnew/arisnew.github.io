---
layout: post
title: "Odoo 20 Resmi Dirilis: Fitur Baru AI, Manufacturing, Offline Mode & Panduan Upgrade"
date: 2026-10-05
description: Panduan Odoo 20 untuk bisnis Indonesia. AI otomasi proses, MCP, CRM lead directory, manufacturing continuous production, eCommerce, offline mode, Accounting Assistant — plus checklist upgrade.
image: "https://img.youtube.com/vi/jM6w0YIYvWs/maxresdefault.jpg"
og_image: "https://img.youtube.com/vi/jM6w0YIYvWs/maxresdefault.jpg"
canonical_url: "https://arisnew.odoo.com/blog/odoo-20-fitur-baru-rilis-odoo-experience-2026"
robots: noindex, follow
---

**Odoo 20** resmi diperkenalkan pada **24 September 2026** di opening keynote [Odoo Experience 2026](https://www.odoo.com/event/odoo-experience-2026-9099) Brussels. Versi ini melanjutkan arah Odoo 19 — AI di mana-mana — tetapi dengan pergeseran penting: agent tidak lagi sekadar **menjawab dan menyarankan**, melainkan bisa **membuat dan mengubah record**, menjalankan **automated actions**, serta terhubung ke tool AI eksternal lewat **MCP (Model Context Protocol)**.

Artikel ini merangkum highlight produk untuk tim bisnis dan implementator di Indonesia, berdasarkan [Meet Odoo 20](https://www.odoo.com/blog/odoo-news-5/meet-odoo-20-2439) dan [release notes resmi](https://www.odoo.com/page/release-notes), plus catatan praktis untuk perencanaan upgrade.

## 🎥 Video: Opening Keynote — Unveiling Odoo 20

Fabien Pinckaers memperkenalkan Odoo 20 termasuk demo AI, website builder, manufacturing, dan modul industri:

<div class="video-embed"><iframe src="https://www.youtube.com/embed/jM6w0YIYvWs" title="YouTube video player" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen="" loading="lazy"></iframe></div>

**Video:** [Opening Keynote — Unveiling Odoo 20](https://www.youtube.com/watch?v=jM6w0YIYvWs) (Odoo Experience 2026)

---

## Odoo 20 vs Odoo 19: Apa Bedanya?

| Aspek | Odoo 19 | Odoo 20 |
| --- | --- | --- |
| **Peran AI** | Ask AI, agents, fields, document sort — banyak read/suggest | Agent **menjalankan proses** (create/update record, scheduled actions) |
| **Integrasi AI eksternal** | Terbatas pada ekosistem Odoo | **MCP** — database Odoo terbaca/tertulis dari assistant luar (dengan hak akses user) |
| **Manufacturing** | Peningkatan incremental | **Continuous production**, planning Gantt/Kanban baru |
| **Mobilitas** | Online-first | **Mode offline** — view/edit/create, sync saat online |
| **Website** | AI generate halaman (19) | Prompt → halaman/snippet interaktif + SEO one-click |
| **Core / upgrade** | Perubahan standar | **Access rights disederhanakan**, change tracking chatter, **parent accounts** di CoA |

Jika Anda baru evaluasi AI di Odoo 19, baca dulu [Odoo 19 AI: Enterprise vs Community](https://arisnew.odoo.com/blog/blog-odoo-erp-software-development-1/odoo-19-ai-fitur-enterprise-vs-community-8) — banyak fondasi AI masih relevan; Odoo 20 menambah **eksekusi otomatis** di atasnya.

---

## 1. AI Agent: Otomatisasi Proses dalam Bahasa Natural

Fitur headline Odoo 20: jelaskan proses dalam bahasa sehari-hari, agent **merangkum rencana aksi**, lalu proses **live** — tanpa flow builder manual untuk skenario sederhana.

**Contoh use case** (dari announcement resmi):

- Auto-confirm sales order saat email PO masuk
- Assign lead ke salesperson yang tepat
- Buat project otomatis setelah kontrak ditandatangani
- Notifikasi saat stok di bawah threshold

Perbedaan dengan Odoo 19: agent dapat **write access** terkontrol — bukan hanya query database atau draft teks. Untuk implementator, ini berarti:

1. **Governance** — review prompt, batasi model/tool, audit log chatter
2. **Testing** — pilot di satu tim sebelum production-wide automations
3. **Hak akses** — automations tetap mengikuti ACL user yang men-trigger atau service account yang dikonfigurasi

### MCP: Odoo ↔ Assistant Eksternal

Odoo 20 mendukung koneksi database lewat **Model Context Protocol**, sehingga assistant di luar Odoo (sesuai vendor) dapat read/write data ERP **dengan permission yang sama** seperti user Odoo. Berguna untuk tim yang sudah standar di Copilot/Claude/custom gateway — asal kebijakan keamanan perusahaan mengizinkan.

---

## 2. CRM: Pipeline Tanpa Input Manual Berulang

Odoo 20 memperkuat **prospekting**:

- Direktori **580+ juta entitas** — filter industri & lokasi, generate lead batch
- **IP detection** visitor website → prospect
- Import kontak dari **Outlook/Gmail**
- Scan **kartu nama** → opportunity
- Multi-pipeline: lead baru **routing otomatis** ke board yang benar

Untuk pasar Indonesia, kombinasi website Odoo + CRM tetap menarik untuk B2B yang mengandalkan inbound dan event — asalkan data enrichment memenuhi kebijakan privasi (GDPR-style consent di form website).

---

## 3. Discuss, Phone & Kolaborasi

Semua saluran — **chat, channel, video, AI, live chat, WhatsApp** — lebih terpusat di Discuss:

- Poll cepat, video call **tanpa login** untuk klien (browser)
- Rekaman + **transkrip** — klik baris transkrip untuk loncat ke momen di rekaman
- **Phone**: routing by jam/lokasi/tim, nomor per user, transfer mid-call antar device/kolega

Ini melengkapi integrasi VoIP pihak ketiga (mis. [MiiTel × Odoo](https://arisnew.odoo.com/blog/blog-odoo-erp-software-development-1/integrasi-odoo-miitel-crm-voip-3)) untuk perusahaan yang sudah invest di provider lokal.

---

## 4. Purchasing & Inventory Lebih Proaktif

Highlight operasional:

- Agent AI analisis **vendor renegotiation** dari history PO
- **Min/max stock** dari 30 hari penjualan + draft PO
- Rating kualitas vendor, **3-way match** PO–bill dengan flag selisih
- Deteksi & repack shipment yang beda format penyimpanan
- **Label rak** redesigned

Modul Inventory Indonesia (multi-gudang, delivery) tetap core; fitur AI di sini mengurangi waktu analisis manual untuk planner.

---

## 5. Manufacturing: Continuous Production & Planning

Untuk pabrik/manufaktur ringan:

- **Continuous production** — operator input qty di work order, operasi berikutnya **auto release** tanpa menunggu MO selesai total
- Compare **BoM** side-by-side
- Reschedule MO di **Gantt** (dengan/tanpa buffer)
- **Kanban** manufacturing dengan drag-drop prioritas & color coding

Jika Anda di Odoo 17/18 MRP, ini alasan kuat evaluasi upgrade — terutama saat bottleneck di handoff antar work center.

---

## 6. Website & eCommerce

**Website:** deskripsikan halaman → Odoo generate; AI buat **snippet interaktif**; translate & SEO one-click; layout editor lebih rapi.

**eCommerce:**

- Filter pickup point per negara, thumbnail variant di product page
- Progress bar **free shipping** di checkout
- Pilih **tanggal delivery** di checkout
- Return dari portal, **Pay on Invoice** (order tanpa bayar langsung)

Cocok untuk brand D2C Indonesia yang ingin kurangi dependency agency untuk landing campaign.

---

## 7. POS, Field Service & Offline Mode

**POS:** floor plan editor baru, combo menu otomatis, snooze item unavailable (POS + online sync), catatan customer ke kitchen screen.

**Field Service + Planning:** intervensi drag-drop di kalender, **travel time** otomatis, **map view** lokasi tim, sync kalender eksternal (Google/Outlook).

**Offline mode (Odoo 20):** tim lapangan bisa **baca, edit, buat record** tanpa sinyal; sync saat koneksi kembali. Critical untuk sales/FSM di luar Jawa atau site plant dengan jaringan lemah.

---

## 8. Accounting Assistant & Payroll

**Accounting AI:** tanya receivable, cash flow, dll. dari laporan; **audit reconciliation** cycle-by-cycle dengan rekomendasi; bayar vendor **dari Odoo** (single/bulk/defer).

**Payroll:** dashboard setup warnings, **simulasi pay run** sebelum validasi, jadwal kerja by **jam/hari** (bukan hanya start/end), hitung cost dari **target net salary**, expenses → payroll, simulasi gaji per karyawan.

Untuk entitas Indonesia, tetap perhatikan **lokal payroll & e-Faktur** — fitur global Odoo 20 melengkapi, bukan mengganti modul lokalized compliance.

---

## Checklist Upgrade ke Odoo 20

Sebelum klik upgrade di staging:

| Area | Yang perlu dicek |
| --- | --- |
| **Access rights** | Odoo 20 menyederhanakan grup hak akses — remap role custom & record rules |
| **Chatter tracking** | Model change tracking di chatter — training user + audit workflow |
| **Chart of accounts** | **Parent accounts** — impact laporan keuangan multi-level |
| **Custom modules** | Port ke branch 20.0; test MRP/AI hooks jika pakai server actions AI |
| **Integrasi API** | MCP/AI baru — review security policy perusahaan |
| **Data volume** | Full test copy DB production → staging upgrade Odoo.sh/on-prem |

Timeline realistis: SME 1–2 modul → beberapa minggu UAT; enterprise multi-company → kuartal planning.

---

## Odoo 20 untuk Siapa?

| Profil bisnis | Nilai utama Odoo 20 |
| --- | --- |
| **Distribusi/manufaktur** | Continuous production, inventory AI, PO draft |
| **B2B sales** | CRM directory, automasi assign lead, AI confirm SO |
| **Retail + online** | POS + eCommerce checkout improvements |
| **Jasa lapangan** | Offline mode + Planning/FSM |
| **Finance-led** | Accounting Assistant, payment from Odoo |

---

## Referensi Resmi

- [Meet Odoo 20 — blog Odoo](https://www.odoo.com/blog/odoo-news-5/meet-odoo-20-2439)
- [Odoo 20 Release Notes](https://www.odoo.com/page/release-notes)
- [Odoo Experience 2026](https://www.odoo.com/event/odoo-experience-2026-9099)
- [Opening Keynote — Unveiling Odoo 20 (YouTube)](https://www.youtube.com/watch?v=jM6w0YIYvWs)

### Artikel terkait

- [Odoo 19 AI: Fitur Lengkap + Enterprise vs Community](https://arisnew.odoo.com/blog/blog-odoo-erp-software-development-1/odoo-19-ai-fitur-enterprise-vs-community-8)
- [Odoo FastAPI: REST API & Integrasi](https://arisnew.odoo.com/blog/blog-odoo-erp-software-development-1/odoo-fastapi-rest-api-integrasi-10)
- [Odoo Studio: Kustomisasi Tanpa Coding](https://arisnew.odoo.com/blog/blog-odoo-erp-software-development-1/odoo-studio-kustomisasi-erp-tanpa-coding-2)

### Layanan

- [Jidoka System Indonesia](https://jidokasystem.co.id) — upgrade Odoo, migrasi data, custom module
- [Hubungi saya](https://arisnew.odoo.com/contactus) — assessment readiness Odoo 20 untuk environment Anda

---

## Kesimpulan

**Odoo 20** bukan sekadar “Odoo 19 plus UI baru”. Dua tema dominan: **AI yang mengeksekusi proses bisnis** (plus MCP), dan **operasi yang tidak mati saat offline atau antar work center** (continuous production, offline sync). Di balik demo yang flashy, tim IT harus menyiapkan **upgrade core** — access rights, CoA, custom code — agar go-live tidak surprise.

Ingin roadmap upgrade Odoo 18/19 → 20 atau pilot AI automation untuk satu departemen? [Hubungi saya](https://arisnew.odoo.com/contactus) atau lihat [layanan implementasi](https://arisnew.odoo.com/our-services).
