**Nama Anggota**
**1. Ahmad Septian Eka Setiawan (2212400012293)**
**2. Ihsanul Hafidzin (221240001288)**


# PRODUCT REQUIREMENTS DOCUMENT (PRD)

**Aplikasi UMKMKU**
**Versi:** 1.1 

---

## 1. Ringkasan Eksekutif

Platform SaaS multi‑tenant untuk UMKM berjejaring franchise. Menyasar 4 masalah: biaya software, kompleksitas, operasional terpisah, dan keterbatasan teknologi. Dengan **role‑based access control (RBAC)** ketat dan **audit trail**, sistem meminimalkan kecurangan input.

**Nilai Utama:**

* Biaya terjangkau, tanpa instalasi server lokal.
* Operasional POS, stok, dan keuangan **terintegrasi**.
* **Kontrol & visibilitas** menyeluruh bagi pemilik tenant & franchisor.
* **Keandalan**: offline sementara + sinkronisasi otomatis.

---

## 2. Masalah yang Dipecahkan

1. **Biaya Tinggi** – Pengganti ERP/POS standalone mahal, berbasis cloud.
2. **Kompleksitas** – UI ringkas sesuai kebutuhan UMKM.
3. **Operasional Terpisah** – Stok, penjualan, dan laporan menyatu.
4. **Keterbatasan Teknologi** – Tanpa setup server rumit.
5. **Kecurangan Karyawan** – RBAC, log peristiwa, dan laporan real‑time.

---

## 3. Tujuan & Metrik Kesuksesan

**Objective:** Memberdayakan UMKM berjejaring franchise lewat platform terintegrasi yang mudah, aman, dan andal.

**Key Results (6 bulan):**

* 100 tenant aktif.
* Retensi 80% setelah 3 bulan.
* Akurasi stok membaik ≥60%.
* ≥5 transaksi/hari per franchise.
* Uptime ≥99.5% bulanan.

---

## 4. Persona

**Penyedia Aplikasi (Super Admin – Platform Owner)**: mengelola semua tenant, paket, penagihan, kesehatan sistem.
**Pemilik UMKM (Admin Tenant (Owner Perusahaan))**: kontrol bisnis miliknya (semua franchise di tenant tersebut).
**Kasir Franchise**: transaksi cepat, akses terbatas sesuai outlet.
**(Opsional) Auditor/Accountant**: read‑only laporan & jurnal.

---

## 5. Cakupan (In‑Scope) & Batasan (Out‑of‑Scope)

**In‑Scope (Fase 1‑2):** Registrasi tenant, autentikasi, manajemen produk, POS kasir, transaksi & struk, dashboard & laporan harian/bulanan/tahunan, export PDF/Excel, multi‑metode pembayaran, offline‑first, backup otomatis.
**Out‑of‑Scope (awal):** Akuntansi penuh (COA lengkap), manufaktur, HR/payroll, loyalty program lanjutan.

---

## 6. Fitur & User Stories (Ringkas)

### Epic 1: Tenant & Autentikasi

* **US‑101 (Regis Tenant)**: Calon tenant daftar via email/HP + data bisnis → akun perusahaan.
* **US‑102 (Login)**: Tenant terdaftar login aman (MFA opsional).
* **US‑103 (Profil Perusahaan)**: Admin tenant kelola logo/nama/alamat.
* **US‑104 (Super Admin: Kelola Tenant)**: Super Admin buat/nonaktifkan tenant, atur paket, kuota, status pembayaran.

### Epic 2: Produk Terpusat

* **US‑201**: Admin tenant CRUD produk (nama, SKU, harga, kategori, stok).
* **US‑202**: Upload gambar produk.
* **Anti‑Fraud**: Hanya admin tenant (atau lebih tinggi) yang boleh ubah **harga/stok master**.

### Epic 3: User & Kasir

* **US‑301**: Admin tenant menambah user kasir per franchise.
* **US‑302**: Atur role & permission khusus kasir.
* **Anti‑Fraud**: Kasir punya kredensial unik; tidak bisa membuat user lain.

### Epic 4: Dashboard Tenant (Owner)

* **US‑401**: Overview: penjualan hari ini, produk terlaris, franchise terbaik.
* **US‑402**: Laporan harian (rekonsiliasi).
* **US‑403**: Laporan bulanan/tahunan + analisis margin.
* **US‑404**: Export PDF/Excel.
* **Anti‑Fraud**: Laporan real‑time dan immutable untuk kasir.

### Epic 5: POS Kasir (Franchise)

* **US‑501**: Login kasir.
* **US‑502**: Lihat katalog produk tenant.
* **US‑503**: Input pesanan cepat (pencarian, barcode opsional).
* **US‑504**: Pilih metode bayar (tunai/QRIS/e‑wallet).
* **US‑505**: Hitung total & kembalian otomatis.
* **Anti‑Fraud**: Kasir tidak bisa ubah harga/menambah produk.

### Epic 6: Transaksi & Pembayaran

* **US‑601**: Cetak/unduh struk.
* **US‑602**: Setiap transaksi terekam (waktu, kasir, outlet, item, qty, harga, diskon, pajak, total, metode bayar, reference).
* **US‑603**: History transaksi hari ini & **tutup kas (cash‑up)**.
* **Anti‑Fraud**: Transaksi final, tidak bisa dihapus/edit; koreksi hanya via **retur/void** dengan otorisasi admin.

---

## 7. Acceptance Criteria (Prioritas MVP)

**US‑101**

* Dapat mendaftar dengan email/HP valid; verifikasi OTP/email.
* Tenant baru memiliki status **Trial/Active** sesuai paket default.

**US‑201**

* CRUD produk hanya oleh Admin tenant; setiap perubahan membuat **ProductRevision log**.
* Harga/stok update terlihat di POS ≤5 detik (jika online).

**US‑503/504/505**

* Kasir bisa menambahkan item ke cart <300 ms per aksi pada jaringan normal.
* Hitung total, pajak, diskon, dan kembalian akurat hingga 2 desimal.
* Metode bayar tercatat; untuk non‑tunai, simpan **payment_ref**.

**US‑602**

* Audit trail berisi: `user_id`, `role`, `action`, `entity`, `before/after`, `timestamp`, `ip/device`.
* Log dapat difilter per tenant/outlet/kasir.

**US‑404**

* Export PDF/Excel untuk laporan harian/bulanan <15 detik untuk ≤10k baris.

---



## 8. Arsitektur Multi‑Tenant

**Model:** *Database terpisah per tenant* (isolasi kuat, mudah backup/restore).
**Catatan:** Metadata global (akun tenant, paket, penagihan) disimpan di **Control DB**.

**Komponen Utama:**

* **Client**: Web (tablet/smartphone), PWA untuk offline.
* **API Gateway/Service**: Auth, Tenant Router, POS, Produk, Laporan, Pembayaran, Audit.
* **Control DB (Global)**: tenants, subscriptions, billing, feature flags.
* **Tenant DB (Per Tenant)**: products, outlets, users, transactions, inventory, reports.
* **Object Storage**: gambar produk, export laporan.
* **Queue/Stream**: event transaksi, sinkronisasi, log.
* **Observability**: logging, metrics, tracing.

**Routing Tenant:** header `X-Tenant-Id`/subdomain → **Tenant Router** → koneksi ke DB tenant terkait.

---

## 9. Skema Data Awal (Ringkas)

**Control DB**

* `tenants(id, name, owner_email, plan, status, created_at)`
* `subscriptions(tenant_id, plan, period, next_billing_at, status)`
* `users_global(id, email_hash, status)` (opsional untuk SSO lintas tenant)

**Tenant DB**

* `users(id, name, email, role, outlet_id, status, last_login_at)`
* `outlets(id, name, address)`
* `products(id, sku, name, category, price, stock, image_url, is_active)`
* `product_revisions(id, product_id, changed_by, before, after, changed_at)`
* `transactions(id, outlet_id, cashier_id, subtotal, tax, discount, total, pay_method, pay_ref, created_at)`
* `transaction_items(id, transaction_id, product_id, qty, price, discount)`
* `cashups(id, outlet_id, cashier_id, open_float, cash_in, cash_out, close_float, note, created_at)`
* `audit_logs(id, user_id, role, action, entity, entity_id, before, after, ip, device, created_at)`

---

## 10. API (Sketsa Endpoint)

* **Auth**: `POST /auth/register`, `POST /auth/login`, `POST /auth/otp/verify`
* **Tenant Mgmt (Super Admin)**: `GET/POST/PATCH /admin/tenants`
* **Produk**: `GET/POST/PATCH /products`, `PATCH /products/{id}/price`, `PATCH /products/{id}/stock`
* **POS**: `POST /pos/cart/price`, `POST /pos/checkout`, `GET /pos/transactions?date=today`
* **Laporan**: `GET /reports/daily`, `GET /reports/monthly`, `POST /reports/export`
* **Audit**: `GET /audit?scope=tenant&...`

**Headers**: `Authorization: Bearer <token>`, `X-Tenant-Id: <tenant_id>`

---

## 11. Offline‑First & Sinkronisasi

* **Mode offline**:

  * Cache katalog produk & harga per outlet.
  * Queue transaksi lokal (IndexedDB/SQLite) + nomor referensi sementara.
* **Saat online kembali**:

  * Batch sync: push transaksi → server mengembalikan ID final & stok terbarui.
  * **Conflict resolution**: *last‑write‑wins* untuk stok dengan **revision check**; jika konflik, tandai sebagai *exception* untuk review admin.
* **Indikator status**: banner “Offline/Syncing/Up‑to‑date”.

---

## 12. Laporan & Export

* Filter per tanggal/outlet/produk/kasir.
* KPI: total penjualan, AOV, top‑SKU, margin (jika cost ditambahkan), void/retur rate.
* Export PDF/Excel di background job; notifikasi saat siap unduh.

---

## 13. Keamanan, Privasi, & Kepatuhan

* **RBAC** ketat + **MFA opsional** untuk admin.
* **Audit trail** menyeluruh; **immutable** (append‑only, tidak bisa diedit).
* **Tenant isolation** di level DB & storage.
* **Enkripsi**: at‑rest & in‑transit (HTTPS/TLS).
* **Backup**: harian per tenant + uji restore rutin.
* **Rate limiting** & proteksi brute force.
* **PII minimization** (simpan seperlunya).

---

## 14. NFR (Non‑Functional Requirements)

* **Kinerja**: p95 respons API POS <300 ms (online).
* **Skalabilitas**: 1 tenant → 1000 tenant tanpa perubahan arsitektur besar.
* **Ketersediaan**: 99.5%/bulan.
* **Observabilitas**: dashboard error rate, latency, throughput, dan log audit.
* **Portabilitas**: containerized (Docker/K8s) + IaC untuk provisioning.

---

## 15. Rencana Rilis

**Fase 1 (MVP – 2 bulan)**

* Registrasi/login, manajemen produk, POS dasar, transaksi & struk, laporan harian, offline‑cache dasar, backup otomatis.
  **Fase 2 (1 bulan)**
* Laporan bulanan & tahunan, multi metode bayar (tunai/QRIS/e‑wallet), export PDF/Excel, dashboard analitik lanjut.
  **Fase 3 (opsional)**
* Notifikasi stok menipis, barcode scanner, retur/void dengan otorisasi, multi‑gudang, biaya pokok (COGS) & margin.

---

## 16. QA & UAT (Contoh Test Case Ringkas)

* **Auth**: verifikasi OTP, blokir brute force setelah N percobaan.
* **Produk**: uji revisi harga/stok → muncul di POS; audit log tercatat.
* **POS**: simulasi 200 transaksi/jam/outlet; akurasi total, pajak, kembalian.
* **Offline**: matikan jaringan → input 10 transaksi → nyalakan → semua tersinkron, stok terkurangi.
* **Laporan**: export 10k baris <15 detik.
* **RBAC**: kasir tidak bisa ubah produk; admin bisa melihat audit log tenant‑nya saja; Super Admin bisa melihat semua tenant.

---

## 17. Wireframe (Sederhana)

**Dashboard Tenant**
Header: Logo + nama perusahaan
Sidebar: Dashboard, Produk, Outlets, User, Laporan, Audit
Main: Kartu statistik (Penjualan Hari Ini, Total Transaksi, Produk Terlaris), Grafik penjualan (harian/bulanan/tahunan), Transaksi terbaru.

**Dashboard Kasir (POS)**
Header: Nama kasir, outlet, tombol logout
Main: Pencarian cepat, grid produk (gambar+harga), Cart/Order summary, metode bayar (Tunai/QRIS/E‑wallet), history transaksi hari ini.

---

## 18. Risiko & Mitigasi

* **Konflik stok lintas outlet** → gunakan revision & eventual consistency; laporan menandai selisih.
* **Kecurangan via retur/void** → butuh otorisasi admin + log alasan & bukti.
* **Keterbatasan perangkat** → PWA ringan, degradasi bertahap (graceful).
* **Lonjakan beban laporan** → jalankan export di background + cache agregasi.

---

## 19. Ikhtisar Teknis (Stack Rekomendasi)

* **Frontend**: React/PWA (TypeScript), IndexedDB untuk offline cache.
* **Backend**: Node.js/NestJS/Express atau Go (service POS & laporan).
* **DB**: PostgreSQL per tenant; Redis untuk cache/session/queue; S3‑compatible untuk objek.
* **Infra**: API Gateway + K8s; observability (Prometheus/Grafana/ELK).
* **CI/CD**: GitHub Actions; IaC (Terraform).

---

## 20. Catatan Tambahan

* **Super Admin (Penyedia Aplikasi)** dapat mengakses **semua tenant** dan fitur agregat/platform (penagihan/plan, fitur, audit global, kesehatan sistem).
* **Admin Tenant (Owner Perusahaan)** hanya mengakses **tenant miliknya sendiri** dan **seluruh franchise** di tenant tersebut.
* **Kasir** hanya mengakses **franchise (outlet) yang ditugaskan**.
* **Notifikasi stok menipis** ditetapkan sebagai item **Fase berikutnya** dan sudah tercantum pada rencana rilis; konfigurasi ambang batas per produk/outlet.

> Ketentuan ini sudah tercermin pada RBAC Matrix & penerapan scope token (`tenant_id`, `outlet_id`).