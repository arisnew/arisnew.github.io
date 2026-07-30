---
layout: post
title: "REST API di Odoo dengan FastAPI: Integrasi Modern, Cepat, dan Dokumentasi Otomatis"
date: 2026-07-30
description: Panduan REST API Odoo dengan modul OCA fastapi. Type hints, Pydantic validation, dokumentasi Swagger otomatis, JWT — alternatif lebih baik dari custom controller. Odoo 18 Community & Enterprise.
canonical_url: "https://arisnew.odoo.com/blog/blog-odoo-erp-software-development-1/odoo-fastapi-rest-api-integrasi-10"
robots: noindex, follow
odoo_path: /blog/blog-odoo-erp-software-development-1/odoo-fastapi-rest-api-integrasi-10
---

Setiap proyek integrasi Odoo — mobile app, e-commerce, warehouse scanner, payment gateway, atau middleware antar sistem — pada akhirnya butuh **REST API** yang stabil, cepat, dan mudah di-maintain. Pendekatan klasik sering jatuh ke dua opsi: memanggil **JSON-RPC/XML-RPC bawaan Odoo** dari luar, atau menulis **custom HTTP controller** (`@http.route`) dari nol.

Keduanya bisa jalan — tapi dari pengalaman implementasi di lapangan, keduanya juga punya trade-off serius: RPC terlalu *low-level* dan mengekspos struktur internal Odoo; custom controller rawan inkonsistensi, dokumentasi manual, dan bug validasi input.

**Modul `fastapi` dari [OCA/rest-framework](https://github.com/OCA/rest-framework/tree/18.0/fastapi)** menawarkan jalan ketiga yang jauh lebih produktif: integrasi **FastAPI** — framework Python modern berbasis type hints — langsung ke dalam ekosistem Odoo. Saya sudah mengimplementasikannya di beberapa proyek (dengan tambahan custom JWT, request logging, rate limiting), dan hasilnya **signifikan lebih baik** dibanding custom API sendiri: lebih cepat develop, error lebih sedikit, dan dokumentasi API otomatis tersedia via Swagger UI.

Artikel ini membahas mengapa FastAPI + Odoo worth it, cara kerjanya, dan pola implementasi yang saya rekomendasikan untuk kebutuhan integrasi REST API production.

## Masalah Pendekatan API Konvensional di Odoo

### JSON-RPC / XML-RPC Bawaan

Odoo sudah punya API RPC sejak awal. Cocok untuk script internal atau Odoo client library, tetapi **kurang ideal sebagai public REST API**:

| Aspek | JSON-RPC Odoo | Dampak |
| --- | --- | --- |
| Protokol | RPC procedure call, bukan REST resource | Sulit dipahami tim frontend/mobile |
| Dokumentasi | Tidak ada OpenAPI/Swagger | Integrator harus trial-error |
| Validasi input | Manual di setiap call | Rentan bug & injection |
| Security model | User/password + model ACL | Terlalu permisif jika tidak hati-hati |
| Versioning | Tidak native | Breaking change sulit dikontrol |

### Custom HTTP Controller (`@http.route`)

Banyak tim menulis controller sendiri:

```python
@http.route('/api/v1/orders', type='json', auth='user', methods=['POST'])
def create_order(self, **kwargs):
    # validasi manual, error handling manual, docs? tidak ada
    ...
```

Masalah yang sering muncul:

- **Tidak ada schema validation** — input divalidasi manual, inkonsisten antar endpoint
- **Dokumentasi terpisah** — Swagger/Postman collection sering outdated
- **Error response tidak standar** — setiap developer format berbeda
- **Testing sulit** — tidak ada test client bawaan
- **Coupling tinggi** — controller langsung expose struktur model Odoo ke luar

## Apa Itu Modul OCA FastAPI?

Modul **`fastapi`** (maintainer: [ACSONE SA/NV](https://acsone.eu)) adalah addon OCA yang memungkinkan Anda membangun **REST API berbasis FastAPI** di dalam Odoo, dengan:

- **Type hints Python** — editor auto-complete, fewer runtime errors
- **Pydantic models** — validasi request/response otomatis
- **OpenAPI/Swagger UI** — dokumentasi interaktif auto-generated
- **Dependency injection** — auth, Odoo env, pagination, reusable di semua route
- **Integrasi native Odoo** — ACL, record rules, transaction, retry mechanism

FastAPI sendiri dikenal sebagai salah satu framework Python **tercepat** (setara NodeJS/Go berkat Starlette + Pydantic), dan modul OCA ini menjembatani WSGI Odoo ke ASGI FastAPI via middleware khusus.

```
┌──────────────┐     HTTP      ┌─────────────────┐     WSGI→ASGI    ┌──────────────┐
│ Mobile App   │──────────────▶│  Odoo Server    │─────────────────▶│  FastAPI App │
│ E-commerce   │   REST/JSON   │  fastapi addon  │   middleware     │  + Routers   │
│ Middleware   │               │  /fastapi/...   │                  │  + Pydantic  │
└──────────────┘               └─────────────────┘                  └──────┬───────┘
                                                                           │
                                                                    odoo_env → ORM
```

## Mengapa Saya Pilih FastAPI (Pengalaman Lapangan)

Setelah mengimplementasikan modul ini di beberapa proyek integrasi — dengan tambahan custom **JWT authentication**, **request/response logging**, dan **error tracking** — perbandingannya dengan custom API controller sangat jelas:

| Kriteria | Custom Controller | OCA FastAPI |
| --- | --- | --- |
| Kecepatan development | Lambat (boilerplate banyak) | **2–3× lebih cepat** |
| Validasi input | Manual, inkonsisten | **Otomatis via Pydantic** |
| Dokumentasi API | Manual, sering outdated | **Swagger UI auto-generated** |
| Type safety | Tidak ada | **Python type hints** |
| Error handling | Ad-hoc per endpoint | **Standar HTTP status + Odoo exceptions** |
| Testing | Setup sendiri | **FastAPITransactionCase bawaan** |
| Auth extensibility | Rewrite dari nol | **Dependency override (JWT, API Key, Basic)** |
| Performa | OK | **Sangat baik** (Starlette async) |
| Maintainability | Sulit saat tim besar | **Struktur router/schemas jelas** |

Yang paling terasa di lapangan: **minim error** karena Pydantic menolak request invalid sebelum masuk business logic, dan tim integrator eksternal bisa **self-service** via Swagger UI tanpa tanya-tanya format field.

## Instalasi & Prasyarat

### 1. Dependensi Python

```bash
pip install odoo-addon-fastapi
# atau dari source OCA rest-framework branch 18.0
```

### 2. Install Modul Odoo

Tambahkan repo [OCA/rest-framework](https://github.com/OCA/rest-framework) ke addons path, install modul **`fastapi`**.

Modul tersedia untuk Odoo 16–19. Artikel ini merujuk **Odoo 18** (branch `18.0`).

### 3. Struktur Addon Custom

Rekomendasi struktur direktori dari dokumentasi OCA:

```
my_api/
├── models/
│   └── fastapi_endpoint.py    # inherit fastapi.endpoint
├── routers/
│   ├── __init__.py
│   ├── partners.py
│   └── sale_orders.py
├── schemas/
│   ├── __init__.py
│   └── partner.py             # Pydantic models
├── security/
│   └── ir.model.access.csv
├── dependencies.py            # custom auth (JWT, dll.)
└── __manifest__.py
```

## Langkah Implementasi: Dari Nol ke Endpoint Live

### Step 1: Daftarkan App di `fastapi.endpoint`

```python
from fastapi import APIRouter
from odoo import fields, models

demo_api_router = APIRouter()


class FastapiEndpoint(models.Model):
    _inherit = "fastapi.endpoint"

    app: str = fields.Selection(
        selection_add=[("my_api", "My API")],
        ondelete={"my_api": "cascade"},
    )

    def _get_fastapi_routers(self):
        if self.app == "my_api":
            return [demo_api_router]
        return super()._get_fastapi_routers()
```

### Step 2: Definisikan Pydantic Schema

```python
from pydantic import BaseModel, ConfigDict


class PartnerInfo(BaseModel):
    name: str
    email: str | None = None
    model_config = ConfigDict(from_attributes=True)
```

### Step 3: Buat Route Handler

```python
from typing import Annotated

from fastapi import Depends
from odoo.api import Environment
from odoo.addons.fastapi.dependencies import odoo_env


@demo_api_router.get("/partners", response_model=list[PartnerInfo])
def get_partners(
    env: Annotated[Environment, Depends(odoo_env)],
) -> list[PartnerInfo]:
    return [
        PartnerInfo.model_validate(partner)
        for partner in env["res.partner"].search([], limit=50)
    ]
```

### Step 4: Konfigurasi Endpoint di Odoo UI

1. Buka **Settings → Technical → FastAPI Endpoints**
2. Buat endpoint baru: pilih app **My API**, set path (mis. `/my_api`)
3. Assign **user khusus** dengan ACL minimal (bukan admin!)
4. Klik link **Docs** → Swagger UI langsung tersedia

Prinsip penting dari OCA: **jangan expose seluruh model Odoo**. API adalah kontrak high-level untuk use case spesifik — bukan mirror database.

## Fitur Kunci yang Membuat Integrasi Lebih Mudah

### 1. Dokumentasi Swagger / OpenAPI Otomatis

Setiap route dengan `response_model` dan type hints otomatis muncul di **Swagger UI** (`/docs`). Tim mobile/frontend bisa:

- Lihat semua endpoint, parameter, dan response schema
- **Try it out** langsung dari browser
- Export OpenAPI spec untuk generate client SDK (TypeScript, Kotlin, Swift)

Ini alone menghemat puluhan jam komunikasi "field ini formatnya apa?".

### 2. Dependency Injection — Auth, Env, Pagination

Modul menyediakan dependency siap pakai:

| Dependency | Fungsi |
| --- | --- |
| `odoo_env` | Akses ORM Odoo |
| `fastapi_endpoint` | Konfigurasi endpoint instance |
| `authenticated_partner` | Partner terautentikasi |
| `authenticated_partner_env` | Env + context partner (untuk record rules) |
| `paging` | Pagination standar untuk search endpoint |

Auth mechanism **tidak hardcoded** — bisa di-override per app via `dependency_overrides`. OCA demo menyediakan Basic Auth dan API Key; di proyek saya, saya extend dengan **JWT Bearer token**:

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()


def jwt_authenticated_partner(
    credentials: Annotated[HTTPAuthorizationCredentials, Depends(security)],
    env: Annotated[Environment, Depends(odoo_env)],
):
    payload = decode_jwt(credentials.credentials)  # custom JWT logic
    partner = env["res.partner"].browse(payload["partner_id"])
    if not partner.exists():
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED)
    return partner
```

Register override di `_get_app()` endpoint model — auth method bisa dipilih per endpoint instance dari UI Odoo.

### 3. Security via ACL & Record Rules Odoo

Pola security yang saya rekomendasikan (dan didokumentasikan OCA):

1. Buat **user khusus** per API app — tanpa akses admin
2. Buat **security group** yang imply `FastAPI Endpoint Runner`
3. Definisikan **ir.model.access** per model yang di-expose
4. Tambahkan **ir.rule** dengan domain `authenticated_partner_id` untuk isolasi data per partner

```xml
<record id="api_sale_order_rule" model="ir.rule">
  <field name="name">API: partner sees own orders only</field>
  <field name="model_id" ref="sale.model_sale_order"/>
  <field name="domain_force">[('partner_id', '=', authenticated_partner_id)]</field>
  <field name="groups" eval="[(4, ref('my_api_group'))]"/>
</record>
```

Keamanan API **leverage mekanisme Odoo native** — bukan reinvent the wheel.

### 4. Pagination & Search Pattern

Untuk list endpoint, gunakan helper bawaan:

```python
from odoo.addons.fastapi.dependencies import paging, authenticated_partner_env
from odoo.addons.fastapi.schemas import PagedCollection, Paging


@router.get("/sale_orders", response_model=PagedCollection[SaleOrder])
def list_sale_orders(
    paging: Annotated[Paging, Depends(paging)],
    env: Annotated[Environment, Depends(authenticated_partner_env)],
) -> PagedCollection[SaleOrder]:
    domain = []  # filtered by record rules automatically
    count = env["sale.order"].search_count(domain)
    orders = env["sale.order"].search(
        domain, limit=paging.limit, offset=paging.offset
    )
    return PagedCollection[SaleOrder](
        count=count,
        items=[SaleOrder.model_validate(o) for o in orders],
    )
```

Response konsisten: `{ "count": 150, "items": [...] }` — client tidak perlu tebak format pagination.

### 5. Multi-Language via Accept-Language Header

FastAPI addon otomatis parse header `Accept-Language` (RFC 7231) dan set context Odoo. Field translated (`name`, `description`) otomatis return dalam bahasa yang diminta — tanpa kode extra.

### 6. Testing dengan FastAPITransactionCase

```python
from odoo.addons.fastapi.tests.common import FastAPITransactionCase


class TestMyAPI(FastAPITransactionCase):

    def test_list_partners(self):
        with self._create_test_client(router=demo_api_router) as client:
            response = client.get("/partners")
        self.assertEqual(response.status_code, 200)
        self.assertIsInstance(response.json(), list)
```

Dependency bisa di-mock — test auth failure, permission denied, invalid payload, semua tanpa HTTP server nyata.

## Custom Extension: JWT, Logging, Rate Limiting

Modul OCA fastapi dirancang extensible. Di implementasi production saya, layer tambahan yang paling berguna:

### JWT Authentication

Ganti Basic Auth/API Key dengan JWT untuk mobile app dan SPA:

- Login endpoint issue token (access + refresh)
- Semua route protected pakai `HTTPBearer` dependency
- Token payload: `partner_id`, `user_id`, `exp`, `scopes`
- Revocation list di Redis atau Odoo model `api.token.blacklist`

### Request/Response Logging

Middleware FastAPI untuk log setiap request:

```python
@app.middleware("http")
async def log_requests(request: Request, call_next):
    start = time.time()
    response = await call_next(request)
    duration = time.time() - start
    _logger.info(
        "%s %s → %s (%.3fs)",
        request.method, request.url.path,
        response.status_code, duration,
    )
    return response
```

Sangat membantu debug integrasi dengan pihak ketiga — "endpoint X return 422, payload apa yang dikirim?".

### Rate Limiting

Protect endpoint publik dari abuse dengan middleware atau dependency (mis. `slowapi`):

```python
@router.get("/public/catalog")
@limiter.limit("60/minute")
def get_catalog(request: Request, ...):
    ...
```

## Best Practice Desain API (dari OCA + Pengalaman Lapangan)

1. **Route handler tipis** — business logic di Odoo abstract model (`my.api.service`), bukan di handler
2. **Satu endpoint = satu transaksi** — jangan pecah use case ke multiple call yang butuh rollback manual
3. **Jangan expose `id` Odoo sembarangan** — gunakan field `code`/`ref`/`provider` yang stabil antar database
4. **Naming konsisten** — snake_case, plural untuk collection (`/sale_orders`, bukan `/sale_order`)
5. **Pydantic model terpisah** untuk request (create) vs response (read) — field `id` hanya di response
6. **Error response standar** — modul OCA sudah map `ValidationError`, `AccessError`, `MissingError` ke HTTP status yang tepat
7. **Versioning** — prefix path `/api/v1/` sejak awal, siapkan v2 tanpa breaking client lama
8. **Defensive search** — selalu limit default, jangan pernah `search([])` tanpa pagination

## Perbandingan: Kapan Pakai Apa?

| Kebutuhan | JSON-RPC | Custom Controller | OCA FastAPI |
| --- | --- | --- | --- |
| Script internal / Odoo2Odoo | ✅ Ideal | Overkill | Overkill |
| Public REST API | ❌ | ⚠️ Bisa, tapi rawan | ✅ **Ideal** |
| Mobile app backend | ❌ | ⚠️ | ✅ **Ideal** |
| E-commerce sync | ❌ | ⚠️ | ✅ **Ideal** |
| Swagger docs | ❌ | Manual | ✅ Otomatis |
| Type validation | ❌ | Manual | ✅ Pydantic |
| Odoo Community | ✅ | ✅ | ✅ |
| Learning curve | Rendah | Sedang | Sedang |

## Use Case Integrasi yang Cocok

- **Mobile app** — sales force, inventory scanner, delivery driver
- **E-commerce** — sync produk, stok, order dari Shopify/WooCommerce/Marketplace
- **Payment gateway** — webhook + API callback
- **Middleware ERP** — sync Odoo ↔ SAP/Oracle/legacy system
- **Customer portal custom** — SPA React/Vue dengan auth JWT
- **IoT / warehouse** — barcode scanner, WMS integration
- **Partner B2B portal** — order placement, tracking, invoice download

Contoh nyata: alih-alih tim mobile memanggil `call_kw('sale.order', 'create', ...)` via JSON-RPC, mereka hit `POST /api/v1/sale_orders` dengan payload ter-validasi — dan debug via Swagger jika ada masalah.

## Known Limitations

- **WebSocket belum supported** — roadmap OCA masih open (tantangan bridge WSGI→ASGI untuk koneksi persistent)
- **Setup awal butuh developer Odoo** — registrasi app, ACL, dan router; setelah live, tim integrator bisa self-service via Swagger
- **Status Beta di OCA** — modul masih labeled Beta, meski sudah battle-tested di banyak implementasi production ACSONE & komunitas
- **Custom exception handler terbatas** — error harus bubble up ke Odoo transaction layer; handler custom di level FastAPI di-ignore oleh design

## Referensi & Dokumentasi

### Modul & Source Code

- [**OCA fastapi (18.0)**](https://github.com/OCA/rest-framework/tree/18.0/fastapi) — modul utama
- [**OCA rest-framework**](https://github.com/OCA/rest-framework) — ecosystem REST (fastapi, base_rest, graphql)
- [**FastAPI documentation**](https://fastapi.tiangolo.com/) — framework Python
- [**Pydantic documentation**](https://docs.pydantic.dev/) — data validation
- [**extendable-fastapi**](https://github.com/OCA/rest-framework) — extend Pydantic models across addons

### Artikel Terkait

- [Integrasi Odoo dengan MiiTel](https://arisnew.odoo.com/blog/blog-odoo-erp-software-development-1/integrasi-odoo-miitel-crm-voip-6) — contoh integrasi real-time antar sistem
- [Odoo Studio: Kustomisasi Tanpa Coding](https://arisnew.odoo.com/blog/blog-odoo-erp-software-development-1/odoo-studio-kustomisasi-erp-tanpa-coding-2) — untuk kebutuhan non-API

### Layanan Implementasi

- [Halaman layanan Odoo](https://arisnew.odoo.com/our-services) — custom API & integrasi REST
- [Hubungi kami](https://arisnew.odoo.com/contactus) — konsultasi arsitektur API Odoo

## Mulai dengan FastAPI di Odoo

Langkah praktis:

1. Setup staging Odoo 18 + install modul `fastapi` dari OCA
2. Buat addon `my_api` dengan satu route sederhana (GET `/health` atau `/partners`)
3. Konfigurasi endpoint + user security di UI Odoo
4. Buka Swagger UI — verifikasi docs auto-generated
5. Tambah auth (API Key → JWT), ACL, record rules
6. Tulis test dengan `FastAPITransactionCase`
7. Deploy production + monitoring (logging, rate limit)

REST API di Odoo tidak harus painful. Dengan **OCA FastAPI**, Anda dapat framework modern — type-safe, terdokumentasi, dan production-ready — tanpa meninggalkan keamanan dan transaksi native Odoo. Dari pengalaman implementasi langsung, ini investasi setup kecil di awal yang **menghemat bulan maintenance** di kemudian hari.

Butuh bantuan implementasi REST API Odoo — arsitektur endpoint, JWT auth, integrasi mobile/e-commerce, atau migrasi dari custom controller lama? [Hubungi saya](https://arisnew.odoo.com/contactus) untuk konsultasi gratis.
