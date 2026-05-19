# MAKA ERP — After-Sales Enhancement Spec

## Reference
- **Mockup:** https://bimaka.github.io/stages-v5/
- **ERP Live:** https://afsel.maka-system.com
- **Review status:** Stages 1-4 reviewed with Bima, Stages 5-9 pending

---

## Stage 1 — New Service Order (`/orders/new`)
**Current:** Form with Customer → Unit → Entry → Problem → Analysis → MDT sections  
**Target:**

### Job Card Type (5 tabs)
Unit Problem | Unit Accident | Booking Service | Unit TO | Unit Pengganti
*(PDI/TD takeout)*

### Vehicle Information
| Field | Type | Source |
|---|---|---|
| VIN | 4-digit → auto-complete | Input manual |
| NOPOL | Read-only | Auto-fill dari VIN |
| Engine Number | Read-only | Auto-fill dari VIN |
| Color | Read-only | Auto-fill dari VIN |
| STU Date | Read-only | Auto-fill dari VIN |
| IMEI IoT | Read-only | Auto-fill dari VIN |
| Mileage (ODO) | Freetext | Manual |
| Usage Type | Dropdown | Daily Rental (RTO), Personal Use (Retail), Company Fleet (Asset) |

### Customer Information
| Field | Type | Source |
|---|---|---|
| Name | Read-only | Auto-fill dari VIN |
| Phone | Read-only | Auto-fill dari VIN |
| Address | Read-only | Auto-fill dari VIN |
| Type | Dropdown | Asset, Retail, RTO MAKA JKT/SMG/BALI, RTO TRANSGO/BROOM/DASH |

### User Information
| Field | Type |
|---|---|
| Name | Freetext |
| Phone | Freetext |
| Address | Freetext |

### Entry Information
| Field | Type |
|---|---|
| PIC Entry | Dropdown: AFS (Arief,Rico,Empud) + CC (Arfie,Yulia,CC Agents) + OPS (Marthin,Akbar,Fenda,Eko) |
| Entry Method | Dropdown: Towing, Walk in, Booking |
| Destination | Dropdown: WM, RDL, KGD, DPK, GSP, BKS |

### Ticket
| Field | Type |
|---|---|
| TRB Number | Read-only, auto-generated |
| [Create Ticket] | Button |

### Problem (multi-TRB)
| Field | Type |
|---|---|
| Problem Title | Dropdown ~105 item (A-Z) + "— Others —" → freetext |
| [+ Add Problem] | Button → TRB baru |

### Takeout
- Dealer → ganti Destination
- PDI/TD tab
- Need Towing, Menginap
- Part Problem, Component, Severity, Damage Date, Symptoms → pindah ke PQR
- MDT Check → pindah ke PQR
- Analysis Type, PIC → pindah ke Stage 3

---

## Stage 2 — Orders / Data Pool (`/orders`)
**Current:** Flat table: Ticket, Type, Customer, VIN, Status, Date  
**Target:**

- **VIN grouping**: 1 VIN parent row → TRB sub-rows (collapsible ▼)
- Type column takeout
- Kolom: ☐ | TRB Number | Tipe Unit | Tgl Masuk | Jam Masuk | Problem
- Checkbox per TRB (independen)
- "Process Selected → Stage 3" button
- Filter: VIN, Tipe Unit, Lokasi, Status

---

## Stage 3 — Order Detail / Inspeksi (`/orders/{id}`)
**Current:** Tab Problem → Inbound + Inspeksi + QC + sidebar Update/Mech/Triage/Timeline  

### Inbound (read-only, auto-fill dari Stage 1)
VIN, NO POLISI, NO TRB, CUSTOMER, NO HP, TIPE

### Hasil Inspeksi & Routing
| Field | Type |
|---|---|
| Issue Category | Dropdown: 7 kategori (EV Core Battery/E-motor/MCU, Non EV Body/Frame/Electrical/Mechanical, Accident) |
| Detail Issue | Cascading dropdown (filtered by Issue Category) + search + "— Others —" → freetext |
| Routing Auto | Read-only: EV Core → KC, Non EV → SC + Waiting List |
| PIC Inspeksi | Dropdown: CC (Arfie,Yulia,CC Agents), OPS (Marthin,Akbar,Fenda,Eko), AFS (Arief,Rico,Empud) |
| [Submit PQR] | Button (ungu, center) → Stage 4 |

### Sidebar
| Section | Action |
|---|---|
| Update Status | Status dropdown only + Update button |
| Mechanic | Multi-select: Arief,Rico,Empud,Inta,Suhada,Heri,Agus,Dion,Asef,Jua,Jagat,Zainal. Mandatory |
| Triage | **Takeout** (pindah ke PDI page + approval flow) |
| Timeline | Pertahankan + tambah aging (ex: "2d 4h") + SLA flag (⚠️ >1d inspeksi, 🔴 >7d repair) |

### Takeout dari Update
Issue Category, Detail Issue, MDT Check, PSG Check, QC Passed, QC Notes

---

## Stage 4 — PQR Page (`/orders/{id}/pqr`) **NEW PAGE**
**Mockup reference:** https://bimaka.github.io/stages-v5/stage-4-pqr.html

### 1. PQR Header
| Field | Type |
|---|---|
| PQR Number | Read-only, auto: PQR-XXXX |
| TRB Number | Read-only |
| PQR Date | Read-only, auto |

### 2. Info Unit & Customer
| Field | Type |
|---|---|
| VIN, Engine, Color, Customer Type/Name | Read-only auto-fill |
| Dealer Name | Dropdown 6 pilihan |
| PIC Email | Dropdown 11 email |

### 3. Problem Details
| Field | Type |
|---|---|
| Problem Title, Issue Category | Read-only |
| Severity Level | Dropdown 4 level |
| Likelihood | Dropdown 4 level |
| Usage, Parking | Dropdown |
| Date of Damage | Date |
| Symptoms | Textarea |

### 4. MDT Check + Damage
| Field | Type |
|---|---|
| MDT Battery/MCU/Display | Pass/Fail/NA + upload screen capture |
| PSG Emot Check | Pass/Fail/NA + upload screen capture |
| Explanation of Damage | Textarea |
| Visual Damage | Textarea |
| Attachment | File upload (JPG/PNG/MP4, max 10MB) |

### 5. Part Problem & Claim
| Field | Type |
|---|---|
| Part Problem | Read-only |
| Part Name | Searchable dropdown 235 item → auto-fill Part Number |
| Part Number | Auto-fill |
| Requested Part for Claim | Yes/No |
| ➤ If Yes | Multi-part: Claim Part Name (searchable 235 item) + PN (auto) + Qty + Add Part button |
| Quick Fix | Textarea |

### 6. Verifikasi
4 pertanyaan Yes/No: Service Gratis 1, Modifikasi, Bengkel Umum, Banjir

### 7. Actions
← Back to Order Detail | Submit PQR → Stage 5

---

## Implementation Notes
- VIN format: MKMAAB2C0SJ00 + 4 digits (input 0525 → auto-complete MKMAAB2C0SJ000525)
- KC = EV Core Repair at WM Pasar Rebo (not Knowledge Center)
- All dropdowns support search (ketik filter)
- MDT/PSG upload required for PQR submit
- Mech assignment mandatory before advancing from Inspeksi
- Part numbers from SPR database (235 items, name → code mapping)

---

## Batch Update (Stage 2 Data Pool)

**Fitur:** Update status multiple TRB sekaligus dari Data Pool.

**Rules:**
- Hanya TRB dengan **status sama** yang bisa di-batch update
- Checkbox select per TRB (independen dari VIN)
- Dropdown "Batch Update Status ▼" muncul setelah ≥1 TRB dicentang
- Status tujuan: Inspeksi → Parts → Repair → QC → Handover → Selesai

**Flow:**
```
Data Pool → centang TRB-128, TRB-129, TRB-132
         → [Batch Update ▼] pilih "Inspeksi"
         → semua TRB advance ke Inspeksi sekaligus
```

---

## Mechanics Page (`/mechanics`)
**Current:** Just a list of names  
**Target:** Productivity dashboard

### Per Mechanic Card
| Feature | Detail |
|---|---|
| Nama | Arief, Rico, Empud, Inta, Suhada, Heri, Agus, Dion, Asef, Jua, Jagat, Zainal |
| Active TRB | Count TRB yang sedang dikerjakan |
| Completed | Count TRB selesai (hari/minggu/bulan) |
| Avg Repair Time | Rata-rata waktu perbaikan |
| SLA Hit Rate | % selesai sesuai SLA target |

### Detail View (klik mekanik)
- List TRB aktif: VIN, Problem, Status, Lama pengerjaan
- History completed minggu/bulan ini
- Performance chart (SLA vs actual)

### Filter & Sort
- Filter: Hari / Minggu / Bulan / Custom date range
- Sort by: Active TRB, Completed, SLA Rate, Avg Time

---

## Parts Page (`/parts`)
**Current:** Empty/minimal  
**Target:** Central hub parts management (internal only — ERP Retail integration dibahas terpisah)

### 1. Stock Overview
- Matrix view: Parts x Lokasi (WM, RDL, KGD, DPK, GSP, BKS)
- 235 part items dengan qty per lokasi
- Color coding: 🟢 >3, 🟡 1-3, 🔴 0

### 2. Inventory Alerts
- ⚠️ Minimum stock (<3) warning per lokasi
- 🔴 Out of stock auto-highlight
- Re-order suggestion → bisa trigger notif ke procurement

### 3. Cross-Location Transfer
- Request parts dari lokasi A ke lokasi B
- Stock visibility antar lokasi
- Approval flow (supervisor → transfer)

### 4. TRB Parts Tracker
- Tabel: TRB | Part Name | Part Number | Status (Ready/Ordered/No Stock) | Lokasi | ETA
- Filter by status, lokasi
- Flag TRB yang parts-nya belum lengkap

### 5. ERP Retail Integration (TBD)
- Current: https://erp.maka-retail.com/app/home untuk PO, arrival doc, invoicing
- Diskusi terpisah dengan Marcos & Dito: integrasi atau standalone

---

## Stage 5 — Parts Estimation (`/orders/{id}/parts`)
**NEW PAGE** — Setelah PQR submitted, atau dari Order Detail stepper "Parts"

### 1. Parts List per TRB (editable)
Parts dari PQR auto-included, bisa ditambah/dikurangi mekanik.
| Part Name | Part Number | Qty | Warranty | Status | Lokasi | ETA |
- **Part Name**: Searchable dropdown 235 item → auto-fill Part Number
- **Qty**: Freetext
- **[+ Tambah Part]**: Button → searchable dropdown + Qty
- **[✕]**: Remove part dari list
- Warranty: ✅ / ❌ (auto dari PQR claim)

### 2. Parts Status
- ✅ Ready — stock di lokasi
- ⏳ Ordered — PO udah dibuat
- 🔄 Transfer — request lintas lokasi
- ❌ No Stock — perlu procurement

### 3. Cost Summary
- Total parts cost per TRB
- Warranty vs non-warranty breakdown
- PQR claim status (approved/pending/denied)

### 4. Actions
- [← Back to Order Detail]
- [Request Parts Transfer] → ke Parts page
- [All Parts Ready → Stage 6 Repair] — disabled kalau ada parts belum ready

### 5. Flags
- ⚠️ Parts pending → tidak bisa advance ke Repair
- ✅ All ready → button aktif

### 6. Cost Estimation (auto dari Price List)
**Price List Parts:** 235 item (SPR-XXX) — pakai **Price include tax**
**Price List Jasa:** ~160 item (SERVS-XXXXX) — pakai **Price include tax**

Per TRB, sistem auto-kalkulasi:
| TRB | Parts | Jasa | Total |
|---|---|---|---|
| TRB-128 | E-Motor Assy: 3.500.000 + Thermal Sensor: 8.658 | Penggantian e-motor: 495.495 + MDT Check: 49.550 | **4.053.703** |

- Parts: searchable dropdown 235 item → auto-fill Item Code + Price (include tax)
- Jasa: searchable dropdown ~160 item → auto-fill Item Code + Price (include tax)
- **[+ Add Part]** / **[+ Add Jasa]** buttons
- Running total auto-update
- Warranty vs non-warranty flag per item

**Data Sources:**
- Parts: `1joxesKfdA-72iR9_iz0gmzd8uQ7GcCU0kz3lFK2ZTA4` tab "Price List Parts"
- Jasa: `1joxesKfdA-72iR9_iz0gmzd8uQ7GcCU0kz3lFK2ZTA4` tab "Price List Jasa"

### 7. Parts & Jasa Sections (Multi-Add)

**Sequence: "Add" button → new row → searchable dropdown → auto-fill → qty → price**

#### Parts Section
```
PARTS
┌──────────────────────────────────────────────────────────────┐
│ 🔍 Cari part...                                              │
│ ┌──────────────────────────────────────────────────────────┐ │
│ │ Part Name                   | Code     | Qty | Price     │ │
│ │ E-Motor Assy                | SPR-112  | 1   | 3.500.000 │ │
│ │ Thermal Sensor              | TS-002   | 1   | 8.658     │ │
│ └──────────────────────────────────────────────────────────┘ │
│                                              [+ Add Part]    │
└──────────────────────────────────────────────────────────────┘
```

#### Jasa Section
```
JASA
┌──────────────────────────────────────────────────────────────┐
│ 🔍 Cari jasa...                                              │
│ ┌──────────────────────────────────────────────────────────┐ │
│ │ Jasa Name                   | Code       | Qty | Price   │ │
│ │ Penggantian e-motor         | SERVS-082  | 1   | 495.495 │ │
│ │ MDT Check                   | SERVS-140  | 1   | 49.550  │ │
│ └──────────────────────────────────────────────────────────┘ │
│                                              [+ Add Jasa]    │
└──────────────────────────────────────────────────────────────┘
```

#### Total Estimasi
```
                    Parts Subtotal: 3.508.658
                    Jasa Subtotal:    545.045
                    ────────────────────────
                    TOTAL ESTIMASI: 4.053.703
```

**Rules:**
- Parts carry-over dari PQR (auto-included)
- Setiap [Add] → row baru dengan searchable dropdown → auto-fill Code + Price
- Qty default 1, editable
- Harga = Price include tax dari Price List
- Running total update real-time
- Warranty flag per item (✅/❌)

---

## Stage 6 — Repair (`/orders/{id}/repair`)
**NEW PAGE** — Dari Order Detail stepper "Repair"

### 1. Header
| Field | Type |
|---|---|
| TRB Number | Read-only |
| Problem Title | Read-only |
| PIC | Dropdown: Arief, Rico, Empud (team AFS) |
| Status | In Progress / On Hold / Done |

### 2. Parts & Jasa Checklist (carry-over dari Stage 5)
Checklist centang pas item selesai dikerjakan:
- Parts list dengan checkbox per item
- Jasa list dengan checkbox per item

### 3. Progress Notes
- [Add Progress Note] → textarea + timestamp
- Timeline log: "10:00 — Repair started (Rico)"

### 4. Actions
- [Mark Complete → Stage 7 QC]

---

## Stage 7 — QC & Testride (`/orders/{id}/qc`)
**NEW PAGE** — Dari Order Detail stepper "QC"

### Flow
Stage 6 (Repair done) → Stage 7 QC → ✅ Yes → Stage 8
                                    → ❌ No → 🔄 Stage 6

### 1. QC Check
| Field | Type |
|---|---|
| PIC QC | Dropdown: Arief, Rico, Empud |
| QC Date | Date |
| Visual Check | Pass / Fail |
| Functional Check | Pass / Fail |
| MDT Re-check | Pass / Fail + upload screen capture |
| Testride | Pass / Fail |
| Testride ODO | Freetext |
| QC Notes | Textarea |

### 2. Decision
| Field | Type |
|---|---|
| Problem Solved? | Yes / No |
| ➤ If No | Reason textarea + [Loop to Stage 6 Repair] |
| ➤ If Yes | [Advance → Stage 8 Handover] |

---

## Stage 8 — Handover (`/orders/{id}/handover`)
**NEW PAGE** — Dari Order Detail stepper "Handover"

### 1. Handover Info
| Field | Type |
|---|---|
| Tanggal Serah Terima | Date |
| PIC Serah Terima | Dropdown: CC (Arfie, Yulia, CC Agents) + OPS (Marthin, Akbar, Fenda, Eko) |
| Diterima Oleh | Freetext |
| No HP Penerima | Read-only |

### 2. Opsi Unit
| Field | Type |
|---|---|
| Status Unit | Dropdown: Dikembalikan ke Customer / Unit RTS |

### 3. Jika Unit RTS
Customer tidak lanjut sewa-milik → unit masuk pool Ready-to-Sell:
- Auto update MASTER_UNIT status → READY ASSET / READY TO RENT
- Notif ke OPS (Marthin, Akbar)

### 4. Kondisi Unit
| Field | Type |
|---|---|
| ODO Akhir | Freetext |
| Kondisi Fisik | OK / Ada Catatan |
| Catatan | Textarea |

### 5. Konfirmasi
- [Konfirmasi Handover → Stage 9 Done]
- WA notification "Unit Siap Ambil" (jika customer take over)
