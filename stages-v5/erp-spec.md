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
