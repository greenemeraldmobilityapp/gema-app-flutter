# Project Requirement Document (PRD): GEMA App

**Versi:** 2.2 (Flutter Edition — Heritage Modernist Design System)
**Nama Proyek:** GEMA (Green Emerald Mobility App)
**Owner:** Emerald Tech Solution
**Target Launch:** Kabupaten Jepara, Jawa Tengah

## 1. Visi & Tujuan

Membangun Super-App lokal yang mengintegrasikan Marketplace, Logistik (Kurir), dan Jasa (Service) untuk memajukan ekosistem ekonomi digital di Kabupaten Jepara.

## 2. Model Bisnis & Monetisasi

GEMA mengambil keuntungan melalui skema bagi hasil dari setiap transaksi sukses:

- **Komisi Penjual (Merchant):** 15% dari harga produk.
- **Komisi Kurir/Driver:** 15% dari biaya ongkir.
- **Komisi Penyedia Jasa (Service):** 15% dari total nilai jasa.
- **Biaya Aplikasi (Platform Fee):** Rp2.000 flat per transaksi (dibebankan ke pelanggan).

## 3. Peran Pengguna (User Roles)

- **Pelanggan (Buyer):** Belanja produk, pesan kurir/jasa, melakukan pembayaran, dan tracking.
- **Penjual (Merchant):** Mengelola toko dan produk (setelah verifikasi admin).
- **Mitra Kurir (Driver):** Mengantar barang dan makanan.
- **Mitra Jasa (Provider):** Menyediakan jasa spesifik (Tukang ukir, servis AC, dll).
- **Admin (Emerald Tech):** Verifikasi mitra, manajemen keuangan, dan layanan pelanggan.

## 4. Fitur Utama berdasarkan Kategori

### A. GEMA-Food & Marketplace

- Pemesanan barang dari toko terverifikasi di wilayah Jepara.
- Filter produk berdasarkan kategori (Mebel, Kuliner, Fashion, dll).
- Manajemen keranjang dan checkout terintegrasi.
- Multi-item per order (satu toko dalam satu pesanan).

### B. GEMA-Send (Logistik)

- **Sameday:** Pengiriman instan di hari yang sama.
- **Next-Day/Reguler:** Pengambilan barang dilakukan kurir pada hari H pengiriman (pagi hari).
- **Hitung Ongkir:** Otomatis berdasarkan jarak (Haversine Formula untuk tahap awal).

### C. GEMA-Service (Jasa Baru)

- Penyediaan jasa tukang ukir, servis perabot, atau jasa angkut pindahan.
- Sistem pemesanan berdasarkan jadwal.

### D. Sistem Keamanan & Pembayaran (Payment Gateway)

- **Cashless:** Pembayaran via Xendit (QRIS, VA, E-Wallet).
- **COD (Cash on Delivery):**
  - Hanya bisa diambil oleh Driver yang memiliki Saldo Minimal di dompet aplikasi (Saldo > Nilai Komisi + Nilai Barang).
  - Sistem menahan saldo driver (held_balance) saat order diterima.
  - Saldo dipotong otomatis sebagai komisi perusahaan saat pesanan selesai.
  - Jika dibatalkan, held_balance dikembalikan penuh.
- **Withdrawal:** Penjual dan Driver dapat menarik pendapatan mereka ke rekening bank via Xendit.

### E. Sistem Pembatalan (Adopsi Gojek)

- **Pelanggan:** Bisa batal gratis maksimal 5 menit setelah dapat kurir.
- **Penalti:** Pembatalan sepihak setelah kurir berangkat akan dikenakan denda atau blokir akun sementara jika merugikan mitra.

### F. Chat & Komunikasi

- Chat real-time antara Pelanggan ↔ Driver.
- Chat real-time antara Pelanggan ↔ Merchant.
- Tombol darurat (hubungi admin) saat ada masalah di lapangan.

### G. Rating & Review

- Rating 1-5 bintang + komentar setelah order selesai.
- Rating untuk Driver, Merchant, dan Service Provider secara terpisah.
- Average rating ditampilkan di profil toko/driver/provider.

## 5. Alur Kerja Sistem (Workflow)

### 5.1 Status Order Lifecycle

| Status | Deskripsi |
|---|---|
| `pending` | Order baru dibuat, menunggu driver |
| `searching_driver` | Sistem mencari driver terdekat yang saldo cukup |
| `driver_found` | Driver menerima order |
| `driver_to_merchant` | Driver dalam perjalanan ke lokasi penjual |
| `picked_up` | Driver sudah ambil barang dari penjual |
| `driver_to_customer` | Driver dalam perjalanan ke lokasi pelanggan |
| `delivered` | Barang sudah sampai, pembayaran diproses |
| `cancelled` | Order dibatalkan (oleh customer, driver, atau admin) |

### 5.2 Alur Lengkap

1. **Pemesanan:** Pelanggan checkout → Pilih Metode Bayar (Xendit/COD) → Order status `pending`.
2. **Dispatching:** Sistem mencari kurir terdekat (radius 5km) yang memiliki saldo cukup → Status `searching_driver`.
3. **Driver Accept:** Kurir menerima order → Status `driver_found` → `held_balance` di-hold dari saldo driver.
4. **Proses:** Kurir menuju lokasi penjual (`driver_to_merchant`) → Konfirmasi pengambilan (`picked_up`) → Menuju lokasi pelanggan (`driver_to_customer`).
5. **Tracking:** Pelanggan melihat posisi kurir secara real-time di peta (flutter_map).
6. **Selesai:** Kurir konfirmasi sampai (`delivered`) → Held_balance dilepas → Saldo terbagi otomatis (Potong komisi 15%, kirim ke merchant).
7. **Review:** Pelanggan memberikan rating 1-5 bintang + komentar.

### 5.3 COD Flow (Detail)

1. Customer pilih COD saat checkout.
2. Sistem filter driver dengan `balance >= (total_item_price + shipping_fee + app_fee)`.
3. Saat driver accept: `held_balance` = total order di-hold dari balance.
4. Saat delivered:
   - `held_balance` dikembalikan ke `balance`.
   - Komisi merchant (15%) dan komisi driver (15%) dipotong dari `balance` driver.
   - Hasil bersih dikirim ke saldo merchant.
   - Customer bayar tunai ke driver.
5. Saat cancelled: `held_balance` dikembalikan penuh ke driver.

## 6. Spesifikasi Teknologi

- **Framework:** Flutter 3.x (stable) + Dart 3.x
- **State Management:** Riverpod (flutter_riverpod + riverpod_generator)
- **Routing:** go_router (typed routes, deep linking)
- **Backend & Database:** Supabase (PostgreSQL, Auth, Real-time, Storage) via `supabase_flutter` SDK
- **Local Storage:** Isar (NoSQL, offline mode, cache)
- **Peta & Lokasi:** flutter_map + OpenStreetMap (Gratis) + geolocator
- **Payment:** Xendit REST API (via Dio/http)
- **Notifications:** firebase_messaging + flutter_local_notifications
- **CI/CD iOS:** Codemagic (cloud macOS build)
- **Development Environment:** Termux + proot-distro Debian + OpenCode CLI

### Kenapa Flutter?

- **Single Codebase** — Satu kode untuk Android & iOS, hemat waktu & biaya development.
- **Native Performance** — Compiled ke ARM/x86 native code, 60-120 FPS, smooth animations.
- **Hot Reload** — Iterasi cepat saat development, cocok untuk vibe coding dengan AI assistant.
- **Rich Ecosystem** — Package resmi dari Supabase, Google Maps, payment gateway, dll.
- **Play Store & App Store Ready** — Build APK/AAB untuk Android, IPA untuk iOS (via Codemagic).

### Kenapa Riverpod?

- **Compile-safe** — Error terdeteksi saat compile, bukan runtime.
- **Code Generation** — `riverpod_generator` auto-generate providers dari anotasi `@riverpod`.
- **Auto-dispose** — Memory management otomatis, tidak ada memory leak.
- **Testable** — Mudah di-unit test, override provider untuk mocking.
- **Modern Dart** — Mendukung pattern matching, records, sealed classes (Dart 3+).

### Kenapa Isar?

- **Super Cepat** — 10x lebih cepat dari SQLite untuk read/write.
- **Type-safe** — Schema didefinisikan dengan Dart classes, auto-generate.
- **Async-first** — Semua operasi non-blocking, tidak freeze UI.
- **Offline-ready** — Cocok untuk caching data, keranjang belanja, draft order.

---

## SKEMA DATABASE (Supabase PostgreSQL)

> **Catatan:** Skema database Supabase tetap sama (PostgreSQL). Migrasi dikelola via Supabase Dashboard atau SQL migration.
> Di sisi Flutter, kita gunakan `supabase_flutter` SDK untuk query — tidak perlu ORM terpisah.
> Row Level Security (RLS) policies wajib di-setup untuk keamanan data.

### Tabel & Relasi (Lengkap)

| Tabel | Primary Key | Foreign Keys | Deskripsi |
|---|---|---|---|
| `profiles` | `id` (UUID, ref ke `auth.users`) | - | Data pengguna (buyer, merchant, driver, provider, admin) |
| `addresses` | `id` (UUID) | `profile_id` -> `profiles.id` | Multi-alamat per user (rumah, kantor, dll) |
| `wallets` | `id` (UUID) | `profile_id` -> `profiles.id` | Saldo pengguna (balance & held_balance) |
| `transactions` | `id` (UUID) | `profile_id` -> `profiles.id`, `order_id` -> `orders.id` | Ledger/audit trail semua pergerakan saldo |
| `withdrawals` | `id` (UUID) | `profile_id` -> `profiles.id` | Riwayat penarikan saldo ke bank |
| `stores` | `id` (UUID) | `owner_id` -> `profiles.id` | Data toko merchant |
| `offerings` | `id` (UUID) | `store_id` -> `stores.id`, `provider_id` -> `profiles.id` | Produk atau jasa |
| `orders` | `id` (UUID) | `buyer_id`, `driver_id`, `store_id`, `cancelled_by` -> `profiles.id` | Pesanan (food, marketplace, send, service) |
| `order_items` | `id` (UUID) | `order_id` -> `orders.id`, `offering_id` -> `offerings.id` | Detail item per order |
| `driver_locations` | `id` (UUID) | `driver_id` -> `profiles.id` | Posisi real-time kurir di peta |
| `reviews` | `id` (UUID) | `order_id` -> `orders.id`, `reviewer_id`, `target_id` -> `profiles.id` | Rating & review |
| `messages` | `id` (UUID) | `order_id` -> `orders.id`, `sender_id`, `receiver_id` -> `profiles.id` | Chat antar pengguna |
| `notifications` | `id` (UUID) | `profile_id` -> `profiles.id` | Log notifikasi in-app |

### Row Level Security (RLS) Policies

```sql
-- Enable RLS on all tables
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE addresses ENABLE ROW LEVEL SECURITY;
ALTER TABLE wallets ENABLE ROW LEVEL SECURITY;
ALTER TABLE transactions ENABLE ROW LEVEL SECURITY;
ALTER TABLE withdrawals ENABLE ROW LEVEL SECURITY;
ALTER TABLE stores ENABLE ROW LEVEL SECURITY;
ALTER TABLE offerings ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE order_items ENABLE ROW LEVEL SECURITY;
ALTER TABLE driver_locations ENABLE ROW LEVEL SECURITY;
ALTER TABLE reviews ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;

-- Profiles
CREATE POLICY "Profil dapat dilihat oleh semua user terautentikasi" ON profiles FOR SELECT TO authenticated USING (true);
CREATE POLICY "User hanya bisa update profil miliknya sendiri" ON profiles FOR UPDATE TO authenticated USING (auth.uid() = id);

-- Addresses
CREATE POLICY "User bisa lihat alamat sendiri" ON addresses FOR SELECT TO authenticated USING (auth.uid() = profile_id);
CREATE POLICY "User bisa kelola alamat sendiri" ON addresses FOR ALL TO authenticated USING (auth.uid() = profile_id);

-- Wallets
CREATE POLICY "User bisa lihat dompet sendiri" ON wallets FOR SELECT TO authenticated USING (auth.uid() = profile_id);

-- Transactions
CREATE POLICY "User bisa lihat transaksi sendiri" ON transactions FOR SELECT TO authenticated USING (auth.uid() = profile_id);

-- Withdrawals
CREATE POLICY "User bisa lihat withdrawal sendiri" ON withdrawals FOR SELECT TO authenticated USING (auth.uid() = profile_id);
CREATE POLICY "User bisa buat withdrawal" ON withdrawals FOR INSERT TO authenticated WITH CHECK (auth.uid() = profile_id);

-- Stores
CREATE POLICY "Toko bisa dilihat semua orang" ON stores FOR SELECT USING (true);
CREATE POLICY "Hanya pemilik yang bisa buat toko" ON stores FOR INSERT TO authenticated WITH CHECK (auth.uid() = owner_id);
CREATE POLICY "Hanya pemilik yang bisa edit/tutup toko" ON stores FOR UPDATE TO authenticated USING (auth.uid() = owner_id);

-- Offerings
CREATE POLICY "Produk/Jasa bisa dilihat semua orang" ON offerings FOR SELECT USING (true);
CREATE POLICY "Hanya pemilik toko/jasa yang bisa mengelola produk" ON offerings FOR ALL TO authenticated USING (
    EXISTS (SELECT 1 FROM stores WHERE id = store_id AND owner_id = auth.uid()) OR (provider_id = auth.uid())
);

-- Orders
CREATE POLICY "User bisa lihat orderan miliknya" ON orders FOR SELECT TO authenticated USING (
    auth.uid() = buyer_id OR
    auth.uid() = driver_id OR
    auth.uid() = (SELECT owner_id FROM stores WHERE id = store_id)
);
CREATE POLICY "Buyer bisa buat order" ON orders FOR INSERT TO authenticated WITH CHECK (auth.uid() = buyer_id);
CREATE POLICY "Buyer/Driver/Merchant bisa update status" ON orders FOR UPDATE TO authenticated USING (
    auth.uid() = buyer_id OR auth.uid() = driver_id OR auth.uid() = (SELECT owner_id FROM stores WHERE id = store_id)
);

-- Order Items
CREATE POLICY "User bisa lihat item orderan miliknya" ON order_items FOR SELECT TO authenticated USING (
    EXISTS (SELECT 1 FROM orders WHERE id = order_id AND (buyer_id = auth.uid() OR driver_id = auth.uid()))
);

-- Driver Locations
CREATE POLICY "Lokasi driver bisa dilihat semua user terautentikasi" ON driver_locations FOR SELECT TO authenticated USING (true);
CREATE POLICY "Driver bisa update lokasi sendiri" ON driver_locations FOR ALL TO authenticated USING (auth.uid() = driver_id);

-- Reviews
CREATE POLICY "Review bisa dilihat semua orang" ON reviews FOR SELECT USING (true);
CREATE POLICY "User bisa buat review sendiri" ON reviews FOR INSERT TO authenticated WITH CHECK (auth.uid() = reviewer_id);

-- Messages
CREATE POLICY "User bisa lihat pesan yang dikirim/diterima" ON messages FOR SELECT TO authenticated USING (
    auth.uid() = sender_id OR auth.uid() = receiver_id
);
CREATE POLICY "User bisa kirim pesan" ON messages FOR INSERT TO authenticated WITH CHECK (auth.uid() = sender_id);

-- Notifications
CREATE POLICY "User bisa lihat notifikasi sendiri" ON notifications FOR SELECT TO authenticated USING (auth.uid() = profile_id);
```

### Triggers & Functions (Supabase SQL)

```sql
-- Fungsi: Buat Profil & Wallet saat Signup
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS trigger AS $$
BEGIN
  INSERT INTO public.profiles (id, full_name, avatar_url)
  VALUES (new.id, new.raw_user_meta_data->>'full_name', new.raw_user_meta_data->>'avatar_url');

  INSERT INTO public.wallets (profile_id, balance, held_balance)
  VALUES (new.id, 0, 0);

  RETURN new;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE PROCEDURE public.handle_new_user();

-- Fungsi: Bagi Hasil dengan Escrow (15% Komisi)
CREATE OR REPLACE FUNCTION public.process_transaction_fees()
RETURNS trigger AS $$
DECLARE
    comm_merchant DECIMAL;
    comm_driver DECIMAL;
    m_owner_id UUID;
    driver_wallet_balance DECIMAL;
BEGIN
    IF (NEW.status = 'delivered' AND OLD.status != 'delivered') THEN
        comm_merchant := NEW.total_item_price * 0.15;
        comm_driver := NEW.shipping_fee * 0.15;

        -- Cek saldo driver cukup
        SELECT balance INTO driver_wallet_balance FROM public.wallets WHERE profile_id = NEW.driver_id;

        IF driver_wallet_balance >= (comm_merchant + comm_driver) THEN
            -- Potong Komisi dari Saldo Driver
            UPDATE public.wallets
            SET balance = balance - (comm_merchant + comm_driver),
                held_balance = GREATEST(held_balance - NEW.total_amount, 0),
                updated_at = NOW()
            WHERE profile_id = NEW.driver_id;

            -- Kirim Hasil ke Merchant
            SELECT owner_id INTO m_owner_id FROM stores WHERE id = NEW.store_id;
            IF m_owner_id IS NOT NULL THEN
                UPDATE public.wallets
                SET balance = balance + (NEW.total_item_price - comm_merchant),
                    updated_at = NOW()
                WHERE profile_id = m_owner_id;
            END IF;

            -- Catat di ledger transaksi
            INSERT INTO public.transactions (profile_id, order_id, type, amount, description)
            VALUES (NEW.driver_id, NEW.id, 'debit', comm_merchant + comm_driver, 'Komisi platform');

            INSERT INTO public.transactions (profile_id, order_id, type, amount, description)
            VALUES (m_owner_id, NEW.id, 'credit', NEW.total_item_price - comm_merchant, 'Pembayaran order');
        ELSE
            -- Saldo driver tidak cukup, tetap proses tapi flag
            UPDATE public.wallets
            SET held_balance = GREATEST(held_balance - NEW.total_amount, 0),
                updated_at = NOW()
            WHERE profile_id = NEW.driver_id;
        END IF;
    END IF;

    -- Handle cancellation: release held_balance
    IF (NEW.status = 'cancelled' AND OLD.status != 'cancelled' AND NEW.driver_id IS NOT NULL) THEN
        UPDATE public.wallets
        SET held_balance = GREATEST(held_balance - (NEW.total_item_price + NEW.shipping_fee + NEW.app_fee), 0),
            updated_at = NOW()
        WHERE profile_id = NEW.driver_id;
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER on_order_status_changed
  AFTER UPDATE ON public.orders
  FOR EACH ROW EXECUTE PROCEDURE public.process_transaction_fees();

-- Fungsi: Hold balance saat driver accept order (COD)
CREATE OR REPLACE FUNCTION public.hold_driver_balance()
RETURNS trigger AS $$
BEGIN
    IF (NEW.driver_id IS NOT NULL AND OLD.driver_id IS NULL) THEN
        UPDATE public.wallets
        SET held_balance = held_balance + (NEW.total_item_price + NEW.shipping_fee + NEW.app_fee),
            updated_at = NOW()
        WHERE profile_id = NEW.driver_id;

        INSERT INTO public.transactions (profile_id, order_id, type, amount, description)
        VALUES (NEW.driver_id, NEW.id, 'hold', NEW.total_item_price + NEW.shipping_fee + NEW.app_fee, 'Hold saldo order COD');
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER on_driver_assigned
  AFTER UPDATE ON public.orders
  FOR EACH ROW EXECUTE PROCEDURE public.hold_driver_balance();
```

### View

```sql
CREATE VIEW order_summary AS
SELECT
    o.id,
    o.status,
    o.type,
    o.payment_method,
    b.full_name as buyer_name,
    s.store_name,
    d.full_name as driver_name,
    o.total_item_price,
    o.shipping_fee,
    o.app_fee,
    o.total_amount,
    o.created_at,
    o.updated_at
FROM orders o
JOIN profiles b ON o.buyer_id = b.id
LEFT JOIN stores s ON o.store_id = s.id
LEFT JOIN profiles d ON o.driver_id = d.id;

CREATE VIEW user_wallet_summary AS
SELECT
    p.id as profile_id,
    p.full_name,
    p.roles,
    w.balance,
    w.held_balance,
    (w.balance + w.held_balance) as total_assets
FROM profiles p
JOIN wallets w ON p.id = w.profile_id;
```

---

## Isar Collections (Local Storage / Offline Mode)

> **Catatan:** Isar digunakan untuk caching data lokal & offline mode di sisi Flutter.
> Schema didefinisikan dalam Dart classes dengan anotasi `@collection`.
> Auto-generate via `dart run build_runner build`.

```dart
// lib/data/local/isar_collections.dart
import 'package:isar/isar.dart';

part 'isar_collections.g.dart';

@collection
class CartItem {
  Id id = Isar.autoIncrement;
  late String offeringId;
  late String storeId;
  late String name;
  late double price;
  int quantity = 1;
  late String? imageUrl;
  @Enumerated(EnumType.name)
  late OfferingType type;
}

@collection
class CachedOffering {
  Id id = Isar.autoIncrement;
  late String offeringId;
  late String storeId;
  late String? providerId;
  @Enumerated(EnumType.name)
  late OfferingType type;
  late String name;
  late double price;
  String? description;
  String? imageUrl;
  bool isNegotiable = false;
  bool isActive = true;
  late DateTime cachedAt;
}

@collection
class CachedStore {
  Id id = Isar.autoIncrement;
  late String storeId;
  late String ownerId;
  late String storeName;
  String? description;
  String? category;
  double? lat;
  double? lng;
  double averageRating = 0.0;
  bool isActive = true;
  late DateTime cachedAt;
}

@collection
class OfflineOrder {
  Id id = Isar.autoIncrement;
  String? supabaseOrderId;
  late String buyerId;
  String? driverId;
  late String storeId;
  @Enumerated(EnumType.name)
  late OrderType type;
  @Enumerated(EnumType.name)
  late OrderStatus status;
  @Enumerated(EnumType.name)
  late PaymentMethod paymentMethod;
  late double totalItemPrice;
  late double shippingFee;
  late double appFee;
  late double totalAmount;
  double? destLat;
  double? destLng;
  String? destAddress;
  DateTime? scheduledAt;
  late DateTime createdAt;
  bool isSynced = false;
}

enum OfferingType { product, service }
enum OrderType { food, marketplace, send, service }
enum OrderStatus { pending, searching_driver, driver_found, driver_to_merchant, picked_up, driver_to_customer, delivered, cancelled }
enum PaymentMethod { xendit, cod }
```

### Isar Workflow

```bash
dart run build_runner build
```

---

## Flutter Data Layer Architecture

```
lib/
├── data/
│   ├── local/              # Isar collections & DAO
│   ├── remote/             # Supabase services
│   ├── models/             # Dart data classes
│   └── repositories/       # Repository pattern
├── providers/              # Riverpod providers
├── features/               # Feature-first structure
├── core/                   # Shared utilities
│   ├── theme/
│   ├── constants/
│   ├── utils/
│   └── widgets/
└── main.dart
```

---

## Dokumen Panduan UI/UX: GEMA App

### 1. Filosofi Design: "The Heritage Modernist"

GEMA bukan sekadar aplikasi utilitas — ini adalah pengalaman premium yang menghormati warisan kerajinan ukiran kayu Jepara. Design system ini dirancang untuk terasa **editorial, mewah, dan modern**.

**4 Pilar Utama:**

1. **Intentional Asymmetry** — Tolak layout boxy Bootstrap. Gunakan overlapping elements dan skala tipografi yang ekstrem.
2. **Tonal Depth** — Batas antar section didefinisikan oleh pergeseran warna background, bukan border 1px.
3. **"No-Line Rule"** — DILARANG menggunakan `border: 1px solid` untuk memisahkan konten. Semua batas menggunakan pergeseran warna background, transisi tonal, atau negative space.
4. **Editorial Magazine Feel** — Spacious, authoritative, berakar pada kebanggaan lokal Jepara.

### 2. Color Palette (Stitch Design System)

#### Core Brand Colors
| Token | Hex | Flutter Constant | Usage |
|---|---|---|---|
| Primary | `#006D36` | `primary` | Deep emerald. Primary buttons, active states |
| Primary Container | `#50C878` | `primaryContainer` | Emerald terang. Gradient end, secondary accents |
| Secondary | `#006D3D` | `secondary` | Green tone alternatif |
| Secondary Container | `#97F3B5` | `secondaryContainer` | Light green. Chips, badges, active filters |
| Tertiary | `#745B00` | `tertiary` | Warning/amber yellow |
| Tertiary Container | `#D8AD00` | `tertiaryContainer` | Gold/amber. "Premium Selection" badges |

#### Surface Hierarchy (Light Mode) — 7 Level Depth
| Token | Hex | Usage |
|---|---|---|
| `surface` | `#F5FBF1` | Main canvas — very light green-tinted white |
| `surfaceContainerLowest` | `#FFFFFF` | Pure white. Lifted interactive cards |
| `surfaceContainerLow` | `#EFF6EC` | Structural groupings, section backgrounds |
| `surfaceContainer` | `#E9F0E6` | Mid-level containers |
| `surfaceContainerHigh` | `#E3EAE0` | Input field backgrounds |
| `surfaceContainerHighest` | `#DEE4DB` | Chat bubbles, secondary buttons |
| `surfaceVariant` | `#DEE4DB` | Same as highest |

#### Text Colors
| Token | Hex | Usage |
|---|---|---|
| `onSurface` | `#171D17` | Primary text — very dark green-black (BUKAN hitam murni) |
| `onSurfaceVariant` | `#3E4A3F` | Secondary text — desaturated forest green |
| `outline` | `#6E7A6E` | Placeholder text, subtle borders |
| `outlineVariant` | `#BDCABC` | Ghost borders (opacity rendah) |

#### Error & Status
| Token | Hex |
|---|---|
| `error` | `#BA1A1A` |
| `errorContainer` | `#FFDAD6` |
| `onError` | `#FFFFFF` |

#### Fixed/Variants
| Token | Hex |
|---|---|
| `primaryFixed` | `#83FBA5` |
| `primaryFixedDim` | `#66DD8B` |
| `inversePrimary` | `#66DD8B` |
| `inverseSurface` | `#2B322C` |
| `inverseOnSurface` | `#ECF3E9` |

#### Signature Gradient
```dart
static const emeraldGradient = LinearGradient(
  colors: [Color(0xFF006D36), Color(0xFF50C878)],
  begin: Alignment.topLeft,
  end: Alignment.bottomRight,
);
```
Digunakan pada SEMUA primary CTA, hero cards, dan elemen interaktif utama.

### 3. Tipografi (Manrope + Inter)

**Jangan pakai Roboto — terlihat jadul.**

| Role | Font | Size | Weight | Letter Spacing |
|---|---|---|---|---|
| Display (Hero) | Manrope | 36-48px | 800 (ExtraBold) | Tight |
| H1 Screen Title | Manrope | 24px | 900 (Black) | Tight |
| H2 Section Title | Manrope | 18-22px | 700-800 | Tight |
| H3 Card Title | Manrope | 16-18px | 700 (Bold) | Normal |
| Body Large | Inter | 16px | 400-500 | Normal |
| Body Small | Inter | 14px | 400 | Normal |
| Caption/Micro | Inter | 12px | 400-600 | Wide |
| Label/Tag | Inter | 10px | 600-700 | Extra Wide (uppercase) |
| Harga/Angka | Manrope | 16-36px | 800-900 | Tight |

**Font Loading (pubspec.yaml):**
```yaml
fonts:
  - family: Manrope
    fonts:
      - asset: assets/fonts/Manrope-Regular.ttf
      - asset: assets/fonts/Manrope-Medium.ttf (weight: 500)
      - asset: assets/fonts/Manrope-SemiBold.ttf (weight: 600)
      - asset: assets/fonts/Manrope-Bold.ttf (weight: 700)
      - asset: assets/fonts/Manrope-ExtraBold.ttf (weight: 800)
      - asset: assets/fonts/Manrope-Black.ttf (weight: 900)
  - family: Inter
    fonts:
      - asset: assets/fonts/Inter-Regular.ttf
      - asset: assets/fonts/Inter-Medium.ttf (weight: 500)
      - asset: assets/fonts/Inter-SemiBold.ttf (weight: 600)
      - asset: assets/fonts/Inter-Bold.ttf (weight: 700)
```

### 4. Icon System (Material Symbols Outlined)

**Package:** `material_symbols_icons` atau `flutter_material_symbols`

Gunakan **font variation settings** untuk kontrol fill:
- **Outlined (default):** `'FILL' 0, 'wght' 400, 'GRAD' 0, 'opsz' 24`
- **Filled (active states):** `'FILL' 1`

#### Icon Mapping Per Screen:
| Screen | Icons |
|---|---|
| **Home** | menu, account_balance_wallet, add_circle, send, restaurant, package_2, build, moped, shopping_basket, arrow_forward, star, home, analytics, chat, person |
| **Login** | eco, person, lock, visibility, google, ios |
| **Marketplace** | search, shopping_cart, star |
| **Track** | motorcycle, location_on, delivery_dining, verified, chat, call |
| **Wallet** | account_balance_wallet, add_card, send_money, history, receipt_long, payments, account_balance |
| **Service** | brush, ac_unit, local_shipping, plumbing, handshake |
| **Checkout** | arrow_back, location_on, shopping_basket, payments, handshake, eco |
| **Chat** | arrow_back, call, more_vert, add_circle, sentiment_satisfied, send, done_all |
| **Activity** | location_on, inventory_2, eco, confirmation_number |
| **Send** | auto_awesome, location_on, flag, map, bolt, check_circle, schedule, inventory_2 |
| **Profile** | notifications, edit, add_circle, history, receipt_long, settings, help, support_agent, logout, chevron_right |

### 5. Border Radius Patterns

| Element | Radius | Flutter |
|---|---|---|
| Primary Buttons | 16px | `BorderRadius.circular(16)` |
| Cards | 16-24px | `BorderRadius.circular(20)` |
| Hero/Wallet Cards | 32px | `BorderRadius.circular(32)` |
| Bottom Nav | Top corners 24px | `BorderRadius.vertical(top: Radius.circular(24))` |
| Bottom Sheets | Top corners 40px | `BorderRadius.vertical(top: Radius.circular(40))` |
| Chips/Pills | Full | `BorderRadius.circular(9999)` |
| Avatars | Full | `BorderRadius.circular(9999)` |
| Input Fields | 16px | `BorderRadius.circular(16)` |
| Image Thumbnails | 12-16px | `BorderRadius.circular(12)` |

### 6. Spacing Rules

| Context | Value |
|---|---|
| Screen horizontal padding | 24px |
| Between major sections | 32-48px |
| Between cards in lists | 16-24px |
| Grid gaps | 16px (bento), 32px (large) |
| Card internal padding | 16-24px |
| Bottom padding (nav clearance) | 128px |

**The "Forbid Dividers" Rule:** Jangan gunakan garis untuk memisahkan list items. Gunakan spacing vertikal 16px atau 24px.

### 7. Glassmorphism Pattern

Digunakan pada:
- **Top App Bar:** `Color(0xFFF5FBF1).withOpacity(0.8)` + `ImageFilter.blur(sigmaX: 12, sigmaY: 12)`
- **Bottom Nav Bar:** Sama seperti Top App Bar
- **Bottom Sheets:** `surface.withOpacity(0.8)` + `ImageFilter.blur(sigmaX: 24, sigmaY: 24)`
- **Floating buttons on gradient cards:** `Colors.white.withOpacity(0.2)` + `ImageFilter.blur(sigmaX: 8, sigmaY: 8)`

### 8. Shadow Patterns (Tinted Emerald)

```dart
// Primary button
BoxShadow(
  color: const Color(0xFF006D36).withOpacity(0.2),
  blurRadius: 32,
  offset: const Offset(0, 12),
)

// FAB
BoxShadow(
  color: const Color(0xFF006D36).withOpacity(0.3),
  blurRadius: 32,
  offset: const Offset(0, 12),
)

// Bottom nav (upward shadow)
BoxShadow(
  color: const Color(0xFF006D36).withOpacity(0.06),
  blurRadius: 24,
  offset: const Offset(0, -4),
)

// Bottom sheet (upward shadow)
BoxShadow(
  color: const Color(0xFF006D36).withOpacity(0.1),
  blurRadius: 40,
  offset: const Offset(0, -12),
)

// Editorial card
BoxShadow(
  color: const Color(0xFF006D36).withOpacity(0.08),
  blurRadius: 48,
  offset: const Offset(0, 24),
)
```

### 9. Jepara Wood Carving Pattern

SVG-based geometric patterns (star/flower motifs) sebagai background overlay:
- **Light Mode:** 2% opacity
- **Dark Mode:** 4% opacity
- **Tujuan:** "Subliminal discovery" — texture, bukan ilustrasi

### 10. Komponen UI Utama (Reusable Widgets)

#### Gradient Button (Primary CTA)
```dart
Container(
  decoration: BoxDecoration(
    gradient: AppTheme.emeraldGradient,
    borderRadius: BorderRadius.circular(16),
    boxShadow: [
      BoxShadow(
        color: const Color(0xFF006D36).withOpacity(0.2),
        blurRadius: 32,
        offset: const Offset(0, 12),
      ),
    ],
  ),
  child: Material(
    color: Colors.transparent,
    child: InkWell(
      borderRadius: BorderRadius.circular(16),
      onTap: onPressed,
      child: Padding(
        padding: const EdgeInsets.symmetric(vertical: 16, horizontal: 32),
        child: Text(
          label,
          style: const TextStyle(
            fontFamily: 'Manrope',
            color: Colors.white,
            fontSize: 16,
            fontWeight: FontWeight.w700,
          ),
        ),
      ),
    ),
  ),
)
```

#### Glass Card
```dart
ClipRRect(
  borderRadius: BorderRadius.circular(20),
  child: BackdropFilter(
    filter: ImageFilter.blur(sigmaX: 12, sigmaY: 12),
    child: Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: Colors.white.withOpacity(0.8),
        borderRadius: BorderRadius.circular(20),
      ),
      child: child,
    ),
  ),
)
```

#### Input Field (Filled, No Border)
```dart
TextField(
  decoration: InputDecoration(
    filled: true,
    fillColor: const Color(0xFFE3EAE0), // surfaceContainerHigh
    border: InputBorder.none,
    focusedBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(16),
      borderSide: const BorderSide(color: Color(0xFF006D36), width: 2),
    ),
    enabledBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(16),
      borderSide: BorderSide.none,
    ),
    contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 16),
    hintText: 'Placeholder text',
    hintStyle: const TextStyle(color: Color(0xFF6E7A6E)), // outline
  ),
)
```

#### Bottom Navigation Bar (Glassmorphic, Rounded Top)
```dart
Container(
  decoration: BoxDecoration(
    color: const Color(0xFFF5FBF1).withOpacity(0.8),
    borderRadius: const BorderRadius.vertical(top: Radius.circular(24)),
    boxShadow: [
      BoxShadow(
        color: const Color(0xFF006D36).withOpacity(0.06),
        blurRadius: 24,
        offset: const Offset(0, -4),
      ),
    ],
  ),
  child: ClipRRect(
    borderRadius: const BorderRadius.vertical(top: Radius.circular(24)),
    child: BackdropFilter(
      filter: ImageFilter.blur(sigmaX: 12, sigmaY: 12),
      child: BottomNavigationBar(
        backgroundColor: Colors.transparent,
        elevation: 0,
        type: BottomNavigationBarType.fixed,
        selectedItemColor: const Color(0xFF006D36),
        unselectedItemColor: const Color(0xFF6E7A6E),
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.analytics), label: 'Activity'),
          BottomNavigationBarItem(icon: Icon(Icons.chat), label: 'Chat'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    ),
  ),
)
```

#### Status Badge (Pill Style)
```dart
// OTW → secondaryContainer, Pending → tertiaryContainer
Container(
  padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
  decoration: BoxDecoration(
    color: const Color(0xFF97F3B5), // secondaryContainer
    borderRadius: BorderRadius.circular(9999),
  ),
  child: Row(
    mainAxisSize: MainAxisSize.min,
    children: [
      Container(
        width: 8, height: 8,
        decoration: const BoxDecoration(
          color: Color(0xFF006D36),
          shape: BoxShape.circle,
        ),
      ),
      const SizedBox(width: 6),
      const Text('OTW', style: TextStyle(fontSize: 12, fontWeight: FontWeight.w600)),
    ],
  ),
)
```

### 11. Screen-Specific Patterns

#### The "Destination Rule" — Task-Focused Screens
Screen berikut **TIDAK menampilkan bottom navigation bar** karena fokus pada task:
- **Tracking** — Full screen map + bottom sheet
- **Checkout** — Two-column layout, eco-tracker chip
- **Merchant Registration** — Task-focused flow

#### Bottom Sheet (Tracking Screen)
- `BorderRadius.vertical(top: Radius.circular(40))`
- Pull handle: `width: 48, height: 6, color: outlineVariant.withOpacity(0.3)`
- Negative margin `-mt-12` untuk overlap map
- Glassmorphic: `surface.withOpacity(0.8)` + `ImageFilter.blur(sigmaX: 24, sigmaY: 24)`

#### Bento Grid (Home Services)
- 3-column grid layout
- Each service: 80x80px icon container (`surfaceContainerLowest`, `rounded-2xl`) + label
- Asymmetric: beberapa item bisa span 2 columns

#### Wallet Card (Hero)
- Full-width gradient card (`rounded-32px`)
- Jepara pattern overlay 10% opacity
- Internal elements: `Colors.white.withOpacity(0.2)` + `backdrop-blur`
- Large wallet icon watermark (opacity 10%)

### 12. Dual Themes (Light/Dark)

| Elemen UI | Light Mode | Dark Mode |
|---|---|---|
| Surface | `#F5FBF1` | `~#0D1A0F` (dark green-black) |
| Card Surface | `#FFFFFF` | `~#1A241C` |
| Text Primary | `#171D17` | `#ECF3E9` |
| Text Secondary | `#3E4A3F` | Lighter green-grey |
| Primary | `#006D36` | `#66DD8B` (inverse-primary) |
| Top/Bottom Nav | `#F5FBF1/80` + blur | `zinc-950/80` + blur |
| Jepara Pattern | 2% opacity | 4% opacity |

### 13. Design System & Flutter Stack

| Komponen | Pilihan | Alasan |
|---|---|---|
| **UI Framework** | Material 3 (built-in) | Native Flutter, konsisten |
| **Icons** | `material_symbols_icons` | FILL variation, sesuai Stitch design |
| **Fonts** | Manrope + Inter | Editorial feel, geometric + readable |
| **Glass Effect** | `dart:ui` ImageFilter.blur | Native, performa tinggi |
| **Animations** | `flutter_animate` + implicit | Smooth, staggered support |
| **Forms** | Formz + native TextFormField | Type-safe validation |
| **Maps** | flutter_map | OpenStreetMap, gratis |
| **Charts** | fl_chart | Beautiful, lightweight |
| **Toast** | `toastification` | Modern, elegant |
| **Shimmer** | `shimmer` | Gradient loading |
| **Image Cache** | cached_network_image | Auto-caching |
| **State Management** | Riverpod + riverpod_generator | Compile-safe, auto-dispose |

### 14. Flutter Project Structure

```
gema_app/
├── android/
├── ios/
├── lib/
│   ├── main.dart
│   ├── app.dart
│   ├── core/
│   │   ├── theme/
│   │   │   ├── app_theme.dart       # ThemeData + Stitch color tokens
│   │   │   ├── colors.dart          # Surface hierarchy, gradients
│   │   │   ├── typography.dart      # Manrope + Inter
│   │   │   └── shadows.dart         # Tinted emerald shadows
│   │   ├── constants/
│   │   ├── utils/
│   │   │   ├── haversine.dart
│   │   │   ├── formatters.dart
│   │   │   └── validators.dart
│   │   └── widgets/
│   │       ├── glass_card.dart
│   │       ├── gradient_button.dart
│   │       ├── glass_app_bar.dart
│   │       ├── glass_bottom_nav.dart
│   │       ├── filled_input.dart
│   │       ├── status_badge.dart
│   │       ├── shimmer_loader.dart
│   │       ├── empty_state.dart
│   │       ├── error_screen.dart
│   │       ├── service_card.dart
│   │       └── wallet_card.dart
│   ├── data/
│   │   ├── local/
│   │   ├── remote/
│   │   ├── models/
│   │   └── repositories/
│   ├── providers/
│   └── features/
│       ├── auth/
│       ├── home/
│       │   ├── screens/home_screen.dart
│       │   └── widgets/
│       │       ├── wallet_card.dart
│       │       ├── service_bento_grid.dart
│       │       └── promo_carousel.dart
│       ├── marketplace/
│       ├── send/
│       ├── service/
│       ├── orders/
│       ├── tracking/
│       ├── chat/
│       ├── wallet/
│       └── profile/
├── assets/
│   ├── images/
│   ├── fonts/                  # Manrope, Inter
│   └── patterns/               # Jepara wood carving SVG patterns
├── test/
├── pubspec.yaml
└── codemagic.yaml
```

### 15. Emerald Theme (Flutter ThemeData — Stitch Design System)

```dart
// lib/core/theme/app_theme.dart
import 'dart:ui';
import 'package:flutter/material.dart';

class AppTheme {
  // ==========================================
  // COLOR PALETTE (Stitch Design Tokens)
  // ==========================================
  static const primary = Color(0xFF006D36);
  static const primaryContainer = Color(0xFF50C878);
  static const secondary = Color(0xFF006D3D);
  static const secondaryContainer = Color(0xFF97F3B5);
  static const tertiary = Color(0xFF745B00);
  static const tertiaryContainer = Color(0xFFD8AD00);

  static const surface = Color(0xFFF5FBF1);
  static const surfaceContainerLowest = Color(0xFFFFFFFF);
  static const surfaceContainerLow = Color(0xFFEFF6EC);
  static const surfaceContainer = Color(0xFFE9F0E6);
  static const surfaceContainerHigh = Color(0xFFE3EAE0);
  static const surfaceContainerHighest = Color(0xFFDEE4DB);

  static const onSurface = Color(0xFF171D17);
  static const onSurfaceVariant = Color(0xFF3E4A3F);
  static const outline = Color(0xFF6E7A6E);
  static const outlineVariant = Color(0xFFBDCABC);

  static const error = Color(0xFFBA1A1A);
  static const errorContainer = Color(0xFFFFDAD6);
  static const onError = Color(0xFFFFFFFF);

  static const inverseSurface = Color(0xFF2B322C);
  static const inverseOnSurface = Color(0xFFECF3E9);
  static const inversePrimary = Color(0xFF66DD8B);

  // Gradients
  static const emeraldGradient = LinearGradient(
    colors: [primary, primaryContainer],
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
  );

  // Shadows
  static List<BoxShadow> get primaryShadow => [
        BoxShadow(
          color: primary.withOpacity(0.2),
          blurRadius: 32,
          offset: const Offset(0, 12),
        ),
      ];

  static List<BoxShadow> get fabShadow => [
        BoxShadow(
          color: primary.withOpacity(0.3),
          blurRadius: 32,
          offset: const Offset(0, 12),
        ),
      ];

  static List<BoxShadow> get bottomNavShadow => [
        BoxShadow(
          color: primary.withOpacity(0.06),
          blurRadius: 24,
          offset: const Offset(0, -4),
        ),
      ];

  static List<BoxShadow> get bottomSheetShadow => [
        BoxShadow(
          color: primary.withOpacity(0.1),
          blurRadius: 40,
          offset: const Offset(0, -12),
        ),
      ];

  // ==========================================
  // TYPOGRAPHY
  // ==========================================
  static const headlineFont = 'Manrope';
  static const bodyFont = 'Inter';

  // ==========================================
  // LIGHT THEME
  // ==========================================
  static ThemeData get lightTheme => ThemeData(
        useMaterial3: true,
        brightness: Brightness.light,
        fontFamily: bodyFont,
        scaffoldBackgroundColor: surface,
        colorScheme: ColorScheme.light(
          primary: primary,
          onPrimary: Colors.white,
          primaryContainer: primaryContainer,
          secondary: secondary,
          secondaryContainer: secondaryContainer,
          tertiary: tertiary,
          tertiaryContainer: tertiaryContainer,
          surface: surface,
          surfaceContainerLowest: surfaceContainerLowest,
          surfaceContainerLow: surfaceContainerLow,
          surfaceContainer: surfaceContainer,
          surfaceContainerHigh: surfaceContainerHigh,
          surfaceContainerHighest: surfaceContainerHighest,
          error: error,
          errorContainer: errorContainer,
          onError: onError,
          onSurface: onSurface,
          onSurfaceVariant: onSurfaceVariant,
          outline: outline,
          outlineVariant: outlineVariant,
        ),
        appBarTheme: AppBarTheme(
          backgroundColor: Colors.transparent,
          elevation: 0,
          centerTitle: false,
          titleTextStyle: const TextStyle(
            fontFamily: headlineFont,
            fontSize: 20,
            fontWeight: FontWeight.w800,
            color: onSurface,
            letterSpacing: -0.5,
          ),
          iconTheme: const IconThemeData(color: onSurface),
        ),
        cardTheme: CardTheme(
          elevation: 0,
          color: surfaceContainerLowest,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(20),
          ),
        ),
        elevatedButtonTheme: ElevatedButtonThemeData(
          style: ElevatedButton.styleFrom(
            backgroundColor: primary,
            foregroundColor: Colors.white,
            elevation: 0,
            padding: const EdgeInsets.symmetric(vertical: 16, horizontal: 32),
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(16),
            ),
            textStyle: const TextStyle(
              fontFamily: headlineFont,
              fontSize: 16,
              fontWeight: FontWeight.w700,
            ),
          ),
        ),
        inputDecorationTheme: InputDecorationTheme(
          filled: true,
          fillColor: surfaceContainerHigh,
          border: InputBorder.none,
          focusedBorder: OutlineInputBorder(
            borderRadius: BorderRadius.circular(16),
            borderSide: const BorderSide(color: primary, width: 2),
          ),
          enabledBorder: OutlineInputBorder(
            borderRadius: BorderRadius.circular(16),
            borderSide: BorderSide.none,
          ),
          errorBorder: OutlineInputBorder(
            borderRadius: BorderRadius.circular(16),
            borderSide: const BorderSide(color: error, width: 1.5),
          ),
          contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 16),
          hintStyle: const TextStyle(color: outline, fontSize: 14),
        ),
        bottomNavigationBarTheme: const BottomNavigationBarThemeData(
          backgroundColor: Colors.transparent,
          elevation: 0,
          type: BottomNavigationBarType.fixed,
          selectedItemColor: primary,
          unselectedItemColor: outline,
        ),
        textTheme: const TextTheme(
          displayLarge: TextStyle(fontFamily: headlineFont, fontSize: 48, fontWeight: FontWeight.w800, color: onSurface, letterSpacing: -1.5),
          displayMedium: TextStyle(fontFamily: headlineFont, fontSize: 36, fontWeight: FontWeight.w800, color: onSurface, letterSpacing: -1),
          headlineMedium: TextStyle(fontFamily: headlineFont, fontSize: 22, fontWeight: FontWeight.w700, color: onSurface, letterSpacing: -0.5),
          titleLarge: TextStyle(fontFamily: headlineFont, fontSize: 18, fontWeight: FontWeight.w700, color: onSurface),
          titleMedium: TextStyle(fontFamily: headlineFont, fontSize: 16, fontWeight: FontWeight.w700, color: onSurface),
          bodyLarge: TextStyle(fontFamily: bodyFont, fontSize: 16, fontWeight: FontWeight.w400, color: onSurface),
          bodyMedium: TextStyle(fontFamily: bodyFont, fontSize: 14, fontWeight: FontWeight.w400, color: onSurfaceVariant),
          labelLarge: TextStyle(fontFamily: headlineFont, fontSize: 14, fontWeight: FontWeight.w700),
          labelSmall: TextStyle(fontFamily: bodyFont, fontSize: 12, fontWeight: FontWeight.w600, color: onSurfaceVariant, letterSpacing: 1),
        ),
      );

  // ==========================================
  // DARK THEME
  // ==========================================
  static ThemeData get darkTheme => ThemeData(
        useMaterial3: true,
        brightness: Brightness.dark,
        fontFamily: bodyFont,
        scaffoldBackgroundColor: const Color(0xFF0D1A0F),
        colorScheme: ColorScheme.dark(
          primary: inversePrimary,
          onPrimary: const Color(0xFF0D1A0F),
          primaryContainer: primaryContainer,
          secondary: secondaryContainer,
          secondaryContainer: secondary,
          surface: const Color(0xFF1A241C),
          surfaceContainerLowest: const Color(0xFF141E16),
          surfaceContainerLow: const Color(0xFF1E2A20),
          surfaceContainer: const Color(0xFF243026),
          surfaceContainerHigh: const Color(0xFF2A362C),
          surfaceContainerHighest: const Color(0xFF303C32),
          error: const Color(0xFFFFB4AB),
          errorContainer: const Color(0xFF93000A),
          onSurface: inverseOnSurface,
          onSurfaceVariant: const Color(0xFFBDCABC),
          outline: const Color(0xFF889488),
          outlineVariant: const Color(0xFF4A564A),
        ),
        appBarTheme: AppBarTheme(
          backgroundColor: Colors.transparent,
          elevation: 0,
          centerTitle: false,
          titleTextStyle: const TextStyle(
            fontFamily: headlineFont,
            fontSize: 20,
            fontWeight: FontWeight.w800,
            color: inverseOnSurface,
            letterSpacing: -0.5,
          ),
          iconTheme: const IconThemeData(color: inverseOnSurface),
        ),
        cardTheme: CardTheme(
          elevation: 0,
          color: const Color(0xFF1A241C),
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(20),
          ),
        ),
        elevatedButtonTheme: ElevatedButtonThemeData(
          style: ElevatedButton.styleFrom(
            backgroundColor: inversePrimary,
            foregroundColor: const Color(0xFF0D1A0F),
            elevation: 0,
            padding: const EdgeInsets.symmetric(vertical: 16, horizontal: 32),
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(16),
            ),
            textStyle: const TextStyle(
              fontFamily: headlineFont,
              fontSize: 16,
              fontWeight: FontWeight.w700,
            ),
          ),
        ),
        inputDecorationTheme: InputDecorationTheme(
          filled: true,
          fillColor: const Color(0xFF2A362C),
          border: InputBorder.none,
          focusedBorder: OutlineInputBorder(
            borderRadius: BorderRadius.circular(16),
            borderSide: const BorderSide(color: inversePrimary, width: 2),
          ),
          enabledBorder: OutlineInputBorder(
            borderRadius: BorderRadius.circular(16),
            borderSide: BorderSide.none,
          ),
          contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 16),
          hintStyle: const TextStyle(color: Color(0xFF889488), fontSize: 14),
        ),
        bottomNavigationBarTheme: const BottomNavigationBarThemeData(
          backgroundColor: Colors.transparent,
          elevation: 0,
          type: BottomNavigationBarType.fixed,
          selectedItemColor: inversePrimary,
          unselectedItemColor: Color(0xFF889488),
        ),
        textTheme: const TextTheme(
          displayLarge: TextStyle(fontFamily: headlineFont, fontSize: 48, fontWeight: FontWeight.w800, color: inverseOnSurface, letterSpacing: -1.5),
          displayMedium: TextStyle(fontFamily: headlineFont, fontSize: 36, fontWeight: FontWeight.w800, color: inverseOnSurface, letterSpacing: -1),
          headlineMedium: TextStyle(fontFamily: headlineFont, fontSize: 22, fontWeight: FontWeight.w700, color: inverseOnSurface, letterSpacing: -0.5),
          titleLarge: TextStyle(fontFamily: headlineFont, fontSize: 18, fontWeight: FontWeight.w700, color: inverseOnSurface),
          titleMedium: TextStyle(fontFamily: headlineFont, fontSize: 16, fontWeight: FontWeight.w700, color: inverseOnSurface),
          bodyLarge: TextStyle(fontFamily: bodyFont, fontSize: 16, fontWeight: FontWeight.w400, color: inverseOnSurface),
          bodyMedium: TextStyle(fontFamily: bodyFont, fontSize: 14, fontWeight: FontWeight.w400, color: Color(0xFFBDCABC)),
          labelLarge: TextStyle(fontFamily: headlineFont, fontSize: 14, fontWeight: FontWeight.w700),
          labelSmall: TextStyle(fontFamily: bodyFont, fontSize: 12, fontWeight: FontWeight.w600, color: Color(0xFFBDCABC), letterSpacing: 1),
        ),
      );
}
```

---

## Master Checklist Data Teknis

Sebelum memulai development, pastikan kamu sudah menyimpan ini:

- [ ] Supabase URL & Key (Untuk database, auth, storage, realtime).
- [ ] Xendit API Key (Untuk pembayaran & withdrawal).
- [ ] OpenRouter API Key (Untuk asisten coding).
- [ ] Logo GEMA (Format SVG/PNG untuk Flutter assets).
- [ ] Firebase Project + google-services.json & GoogleService-Info.plist (Untuk push notifications).
- [ ] Codemagic Account (Untuk build iOS di cloud).
- [ ] Apple Developer Account ($99/tahun, untuk submit ke App Store).
- [ ] Google Play Developer Account ($25 sekali bayar, untuk submit ke Play Store).
- [ ] Font files: Manrope (6 weights) + Inter (4 weights) di `assets/fonts/`.
- [ ] Jepara wood carving SVG patterns di `assets/patterns/`.

---

## Roadmap Development (Phased Approach)

### Phase 1: Foundation (Minggu 1-2)
- [ ] Setup Flutter project + dependencies (Riverpod, Supabase, Isar, go_router, material_symbols_icons)
- [ ] Setup Supabase database schema & RLS policies
- [ ] Auth (login/register dengan Supabase Auth)
- [ ] Profile management & address management
- [ ] Wallet system (balance, held_balance)
- [ ] Isar local database setup
- [ ] Implement Stitch design system (colors, typography, shadows, glassmorphism, gradients)

### Phase 2: Marketplace & Stores (Minggu 3-4)
- [ ] CRUD Toko & Produk/Jasa
- [ ] Katalog produk dengan bento grid layout
- [ ] Shopping cart (Isar offline) & checkout
- [ ] Image upload ke Supabase Storage
- [ ] Product detail & store detail screens

### Phase 3: Orders & Payment (Minggu 5-6)
- [ ] Order creation & status management
- [ ] Xendit payment integration (QRIS, VA, E-Wallet)
- [ ] COD flow dengan held_balance
- [ ] Transaction ledger & withdrawal
- [ ] Real-time order status (Supabase Realtime)

### Phase 4: Driver & Logistics (Minggu 7-8)
- [ ] Driver dispatching system
- [ ] Real-time tracking dengan flutter_map
- [ ] Haversine formula untuk ongkir
- [ ] Order status lifecycle lengkap
- [ ] Background location updates (geolocator)
- [ ] Glassmorphic bottom sheet untuk tracking

### Phase 5: Social & Polish (Minggu 9-10)
- [ ] Chat system (buyer ↔ driver, buyer ↔ merchant)
- [ ] Rating & review system
- [ ] Push notifications (Firebase)
- [ ] Offline mode & sync queue (Isar)
- [ ] Micro-interactions & animations polish

### Phase 6: Testing & Launch (Minggu 11-12)
- [ ] Unit & widget testing
- [ ] Security audit (RLS policies, secure storage)
- [ ] Performance optimization (image caching, lazy loading)
- [ ] Build APK/AAB (Android) + IPA (iOS via Codemagic)
- [ ] Submit ke Google Play Store & Apple App Store

---

## CI/CD Pipeline (Codemagic)

```yaml
# codemagic.yaml
workflows:
  ios-release:
    name: iOS Release
    instance_type: mac_mini_m1
    max_build_duration: 60
    environment:
      ios_signing:
        distribution_type: app_store
        bundle_identifier: com.emeraldtech.gema
    scripts:
      - name: Install dependencies
        script: |
          flutter pub get
      - name: Build iOS
        script: |
          flutter build ipa --release --export-options-plist=/Users/builder/export_options.plist
    artifacts:
      - build/ios/ipa/*.ipa
    publishing:
      app_store_connect:
        auth: integration
        submit_to_testflight: true
        submit_to_app_store: true

  android-release:
    name: Android Release
    instance_type: linux
    max_build_duration: 60
    environment:
      android_signing:
        - keystore_reference
    scripts:
      - name: Install dependencies
        script: |
          flutter pub get
      - name: Build Android
        script: |
          flutter build appbundle --release
    artifacts:
      - build/**/outputs/**/*.aab
    publishing:
      google_play:
        credentials: $GCLOUD_SERVICE_ACCOUNT_CREDENTIALS
        track: internal
```