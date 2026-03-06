# 📊 Recruitment Pipeline Tracker

> Dashboard rekrutmen IT berbasis web yang dibuat sebagai bagian dari program internship **IT Talent Acquisition**. Dirancang untuk melacak kandidat secara real-time, memvisualisasikan pipeline rekrutmen, dan tersinkronisasi otomatis dengan Google Sheets.

---

## ✨ Fitur Utama

- **Pipeline Funnel** — Visualisasi tahap rekrutmen dari Applied hingga Hired, lengkap dengan conversion rate tiap tahap
- **KPI Dashboard** — Total applicants, positions filled, average time-to-hire, dan offer acceptance rate
- **Source of Hire** — Breakdown dari mana kandidat datang (LinkedIn, JobStreet, Glints, Referral, dll.)
- **Candidate List** — Tabel kandidat dengan filter stage & role, promote antar tahap, edit, dan hapus
- **Import Excel / CSV** — Tarik data massal dari file spreadsheet dengan auto-mapping kolom
- **Google Sheets Sync** — Setiap perubahan (tambah, edit, hapus, promote) langsung tersinkron ke Google Sheets secara real-time
- **Export CSV** — Unduh semua data kandidat dalam format CSV
- **Responsive** — Bisa diakses dari laptop maupun mobile

---

## 🛠️ Tech Stack

| Teknologi | Kegunaan |
|---|---|
| HTML / CSS / JavaScript | Frontend — single file, no framework |
| Google Apps Script | Backend API untuk read/write Google Sheets |
| Google Sheets | Database kandidat yang bisa diedit tim |
| XLSX.js | Parsing file Excel & CSV untuk import |
| Netlify / GitHub Pages | Hosting gratis |

---

## 🚀 Cara Pakai

### 1. Buka Dashboard
Akses langsung via link GitHub Pages atau Netlify (lihat bagian **Demo** di bawah).

### 2. Tambah Kandidat
Klik **+ Add Candidate** → isi form → data otomatis tersimpan ke Google Sheets.

### 3. Import dari Excel
Klik **Import Excel / PDF** → upload file `.xlsx` atau `.csv` → petakan kolom → klik Import.

Format kolom yang direkomendasikan:
```
id | name | role | stage | source | applyDate | hireDate
```

### 4. Update Stage
Klik tombol **→ Next** di tabel untuk memindahkan kandidat ke tahap berikutnya.

### 5. Sync dari Sheets
Klik **☁️ Sync dari Sheets** di sidebar untuk menarik data terbaru dari Google Sheets.

---

## ⚙️ Setup Google Sheets (untuk tim)

Agar data bisa diakses dari device mana saja:

1. Buat Google Sheet baru dengan header:
   ```
   id | name | role | stage | source | applyDate | hireDate
   ```

2. Buka **Extensions → Apps Script**, paste kode berikut:

```javascript
const SHEET_NAME = 'Sheet1';

function getSheet() {
  return SpreadsheetApp.getActiveSpreadsheet().getSheetByName(SHEET_NAME);
}

function doGet(e) {
  const sheet = getSheet();
  const rows = sheet.getDataRange().getValues();
  const headers = rows[0];
  const data = rows.slice(1).map(row => {
    const obj = {};
    headers.forEach((h, i) => obj[h] = row[i]);
    return obj;
  }).filter(r => r.id);
  return ContentService
    .createTextOutput(JSON.stringify(data))
    .setMimeType(ContentService.MimeType.JSON);
}

function doPost(e) {
  const sheet = getSheet();
  const payload = JSON.parse(e.postData.contents);
  const action = payload.action;

  if (action === 'save_all') {
    const lastRow = sheet.getLastRow();
    if (lastRow > 1) sheet.deleteRows(2, lastRow - 1);
    payload.data.forEach(c => {
      sheet.appendRow([c.id, c.name, c.role, c.stage, c.source, c.applyDate, c.hireDate || '']);
    });
  } else if (action === 'add') {
    const c = payload.candidate;
    sheet.appendRow([c.id, c.name, c.role, c.stage, c.source, c.applyDate, c.hireDate || '']);
  } else if (action === 'update') {
    const c = payload.candidate;
    const data = sheet.getDataRange().getValues();
    for (let i = 1; i < data.length; i++) {
      if (String(data[i][0]) === String(c.id)) {
        sheet.getRange(i + 1, 1, 1, 7).setValues([[c.id, c.name, c.role, c.stage, c.source, c.applyDate, c.hireDate || '']]);
        break;
      }
    }
  } else if (action === 'delete') {
    const data = sheet.getDataRange().getValues();
    for (let i = 1; i < data.length; i++) {
      if (String(data[i][0]) === String(payload.id)) {
        sheet.deleteRow(i + 1);
        break;
      }
    }
  }

  return ContentService
    .createTextOutput(JSON.stringify({ status: 'ok' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. Deploy sebagai **Web App** → Execute as: Me → Who has access: Anyone
4. Copy URL deployment → paste di **Settings** dashboard

---

## 📁 Struktur File

```
recruitment-pipeline-tracker/
├── recruitment-pipeline-tracker.html   # Seluruh aplikasi (single file)
└── README.md                           # Dokumentasi ini
```

---

## 💡 Tentang Project

Dashboard ini dibuat selama program internship sebagai **IT Talent Acquisition Intern** untuk membantu tim HR dalam:

- Memantau progress rekrutmen secara visual
- Mengurangi ketergantungan pada spreadsheet manual
- Mempermudah pelaporan metrics rekrutmen ke manajemen
- Menyimpan data kandidat secara terpusat dan real-time

---

## 📄 Lisensi

Project ini dibuat untuk keperluan portofolio dan pembelajaran. Bebas digunakan dan dimodifikasi.

---

*Dibuat dengan ❤️ oleh IT Talent Acquisition Intern · 2025*
