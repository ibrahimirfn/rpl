# Tugas Reverse Engineering Week 12

**Ibrahim Irfanul Haq (H1A023003)**

Dokumen ini menjelaskan hubungan struktural antara entitas-entitas kunci dalam sistem basis data layanan kesehatan.

## Gambaran Umum

Tugas ini mengeksplorasi hubungan antara model data kesehatan inti: Visit, Encounter, Provider, dan Person. Memahami hubungan ini sangat penting untuk bekerja dengan sistem informasi kesehatan.

## Skema Database

### Diagram Relasi Entitas

```sql
-- Tabel Person (entitas dasar)
CREATE TABLE person (
    person_id INT PRIMARY KEY,
    name VARCHAR(255),
    address TEXT,
    birth_date DATE,
    gender VARCHAR(10)
);

-- Tabel Provider (perpanjangan dari Person)
CREATE TABLE provider (
    provider_id INT PRIMARY KEY,
    person_id INT NOT NULL,
    specialty VARCHAR(100),
    license_number VARCHAR(50),
    FOREIGN KEY (person_id) REFERENCES person(person_id)
);

-- Tabel Visit (wadah kunjungan)
CREATE TABLE visit (
    visit_id INT PRIMARY KEY,
    patient_id INT NOT NULL,
    location VARCHAR(100),
    start_date DATETIME,
    end_date DATETIME,
    FOREIGN KEY (patient_id) REFERENCES person(person_id)
);

-- Tabel Encounter (kejadian diskret)
CREATE TABLE encounter (
    encounter_id INT PRIMARY KEY,
    visit_id INT NOT NULL,
    provider_id INT,
    encounter_type VARCHAR(50),
    encounter_date DATETIME,
    diagnosis TEXT,
    FOREIGN KEY (visit_id) REFERENCES visit(visit_id),
    FOREIGN KEY (provider_id) REFERENCES provider(provider_id)
);
```

## Contoh Penggunaan

```python
# Contoh: Query untuk mendapatkan semua encounter dalam satu visit
def get_visit_encounters(visit_id):
    query = """
    SELECT e.encounter_id, e.encounter_type, e.encounter_date,
           p.name as provider_name
    FROM encounter e
    JOIN provider pr ON e.provider_id = pr.provider_id
    JOIN person p ON pr.person_id = p.person_id
    WHERE e.visit_id = ?
    ORDER BY e.encounter_date
    """
    return execute_query(query, [visit_id])

# Contoh: Mendapatkan detail provider
def get_provider_info(provider_id):
    query = """
    SELECT pr.provider_id, pr.specialty, pr.license_number,
           p.name, p.address
    FROM provider pr
    JOIN person p ON pr.person_id = p.person_id
    WHERE pr.provider_id = ?
    """
    return execute_query(query, [provider_id])

# Contoh: Membuat encounter baru
def create_encounter(visit_id, provider_id, encounter_type, diagnosis):
    query = """
    INSERT INTO encounter (visit_id, provider_id, encounter_type, 
                          encounter_date, diagnosis)
    VALUES (?, ?, ?, NOW(), ?)
    """
    return execute_query(query, [visit_id, provider_id, 
                                 encounter_type, diagnosis])
```

## Hubungan Antar Entitas

### 1. Hubungan Visit dan Encounter

**Visit (Kunjungan)**: Mewakili periode waktu ketika pasien aktif berinteraksi dengan sistem layanan kesehatan di suatu lokasi. Visit adalah wadah yang dapat berlangsung selama satu hari atau lebih.

**Encounter (Perjumpaan)**: Mewakili peristiwa diskret atau transaksi klinis tunggal, seperti:
- Mencatat tanda vital
- Membuat diagnosis
- Mengisi formulir

**Hubungan Struktural**: Satu Visit dapat mencakup banyak Encounter.

```
Visit (1) ──── (Banyak) Encounter
```

### 2. Hubungan Provider dan Person

**Person (Individu)**: Adalah entitas dasar yang menyimpan data demografi (nama, alamat, atribut) untuk semua orang dalam sistem (pasien, pengguna, penyedia).

**Provider (Penyedia Layanan)**: Adalah peran khusus yang diberikan kepada seseorang yang memberikan layanan kesehatan.

**Hubungan Struktural**: Tabel provider memiliki kunci asing (`person_id`) yang menunjuk langsung ke tabel person, menjadikan Provider sebagai perpanjangan dari entitas Person.

```
Person (1) ──── (1) Provider
             person_id (FK)
```

## Ringkasan Skema Database

| Entitas | Tujuan | Relasi Kunci |
|---------|--------|--------------|
| Person | Data demografi dasar untuk semua individu | Direferensikan oleh Provider |
| Provider | Peran penyedia layanan kesehatan | Memperpanjang Person melalui FK `person_id` |
| Visit | Wadah untuk periode interaksi pasien | Berisi banyak Encounter |
| Encounter | Transaksi/kejadian klinis diskret | Termasuk dalam satu Visit |

## Konsep Kunci

- **One-to-Many (Satu-ke-Banyak)**: Satu Visit mencakup banyak Encounter
- **One-to-One Extension (Ekstensi Satu-ke-Satu)**: Provider memperpanjang Person melalui hubungan kunci asing
- **Role-Based Design (Desain Berbasis Peran)**: Provider merepresentasikan peran khusus dari entitas Person

## Contoh Kasus Penggunaan

### Skenario 1: Pasien Datang ke Rumah Sakit

1. Pasien datang → **Visit** dibuat
2. Perawat mencatat tanda vital → **Encounter** pertama
3. Dokter melakukan pemeriksaan → **Encounter** kedua
4. Pasien pulang → **Visit** berakhir

### Skenario 2: Mencari Informasi Provider

```python
# Mendapatkan semua provider dengan spesialisasi tertentu
def get_providers_by_specialty(specialty):
    query = """
    SELECT pr.provider_id, pr.license_number,
           p.name, p.address
    FROM provider pr
    JOIN person p ON pr.person_id = p.person_id
    WHERE pr.specialty = ?
    """
    return execute_query(query, [specialty])
```