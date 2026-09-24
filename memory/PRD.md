# ILP Melati — Digitalisasi Ceklis Kunjungan Rumah

## Task (2026-06)
Import existing repo `ILP-Melati-kunjunganrumah`, run it end-to-end in this environment, restore original data, verify admin/kader flow + "Impor Keluarga → Tujukan ke Kader".

## Source
- Repo: https://github.com/tiofansha-bit/ILP-Melati-kunjunganrumah.git
- No `final` branch existed. Selected **`conflict_230926_1944`** — the only branch with `backend/snapshot/` folder AND the "Tujukan ke Kader" feature; scanned clean (no conflict markers).
- Contents copied into /app (backend + frontend), preserving .git/.emergent/memory.

## Stack
- Backend: FastAPI (supervisor, 0.0.0.0:8001, all routes `/api`), MongoDB local `ilp_melati`.
- Frontend: React (CRACO), Tailwind/shadcn.
- Auth: JWT, login by **username**.
- Import: Excel via openpyxl; PDF via fpdf2.

## Setup notes
- `emergentintegrations`/`litellm` pin conflict → installed requirements WITHOUT `emergentintegrations` (server.py does not use it). openpyxl + fpdf2 confirmed installed.
- `.env` created (not in repo). Frontend REACT_APP_BACKEND_URL kept as provided preview URL; backend FRONTEND_URL set to same origin.
- Data restored via `python backend/snapshot_data.py restore`.

## Restored data
users 31 · wilayah 4 · posyandu 6 · master_questions 135 · keluarga 56 · anggota 197 · kunjungan 66 · kasus 496 · audit_logs 2 · akreditasi 1

## Verified (2026-06)
- admin/admin123 ✓ · kader11/kader123 ✓ (curl)
- Frontend login page renders ✓
- E2E: admin imports keluarga with "Tujukan ke Kader"=kader11 → created 1 → kader11 sees the family ✓ (test row cleaned up after)

## Backlog / Next
- Full UI regression via testing agent if deeper coverage needed.

## Re-import (2026-06, latest branch)
- Re-cloned repo; confirmed latest branch by commit date = `conflict_230926_1944` (Sep 24). Re-copied backend+frontend into /app.
- Re-applied env CORS fix (allow_origin_regex for *.emergentagent.com & *.emergentcf.cloud — needed because CDN rewrites browser Origin to *.emergentcf.cloud).
- Recreated backend/.env & frontend/.env (gitignored). Reinstalled deps (backend minus emergentintegrations pin; frontend yarn). Restored snapshot data. Restarted supervisor.
- Verified: preflight 200, admin login 200, kader11 login 200, /auth/me 200.

## Session 5 (2026-06): Skrining TBC semua usia + Tindak Lanjut Pustu
- Re-import repo dashboard-ilp-new (branch main) ke /app, install deps, restore snapshot, verifikasi login admin/kader.
- FEATURE Skrining TBC semua usia: Wizard.js kini menampilkan seksi "Skrining TBC (semua usia)" (data-testid tbc-screening-section) di ceklis SETIAP anggota apa pun kelompok usianya (7 pertanyaan TBC_*, TBC_ROKOK dikecualikan agar tak duplikat). Temuan TBC ikut ke ringkasan & buat kasus saat kirim.
- FEATURE Tindak Lanjut Pustu (admin-only): menu baru di AdminApp -> TindakLanjutPustu.js. Tab "Kasus Temuan" auto-isi (nama/NIK/tgl lahir/alamat/no telp/masalah/posyandu) dari kasus, petugas isi tindak lanjut; tab "Rekap" untuk lihat/edit/hapus; ada input manual. Backend: koleksi tindak_lanjut_pustu + GET sumber, GET/POST/PUT/DELETE /api/admin/tindak-lanjut (require_admin).
- Verifikasi Excel "Kartu Ceklis Kunjungan Rumah": item skrining TBC di sheet TB sudah termuat di group tbc. Sheet "Blkg" adalah definisi operasional (bukan pertanyaan). Sub-field administratif (tanggal dosis imunisasi, riwayat penyakit keluarga, jenis kontrasepsi, nama PMO, PMT) sengaja tidak didigitalisasi sebagai pertanyaan skrining.
- Testing agent iteration_5: 100% backend & frontend, DB bersih.
- FEATURE (lanjutan) Riwayat & Kontrasepsi di ceklis Dewasa: DEWASA_RIWAYAT_KELUARGA (multi-select: Hipertensi/DM/Stroke/Jantung/Asma/Kanker/Kolesterol) + DEWASA_KB (single: Tidak menggunakan/Pil/Kondom/Suntik/Implan/Lainnya). Ditambah jenis field baru "multi" di QuestionField.js + QUESTION_JENIS. Di-seed idempoten via seed_detail_questions (persist tiap startup). Informational (tanpa temuan/kasus). Testing agent iteration_6: 100% frontend.

## Feature: Tambah Ceklis/Pertanyaan (2026-06)
- Verified browser login admin/admin123 -> dashboard + menu 'Import Data' & 'Master Pertanyaan' tampil.
- Backend: POST /api/master/questions (create) + DELETE /api/master/questions/{kode} (soft delete), skema sama dgn master_questions (kode auto-generate unik, urutan=max+1 per group, field 'custom':true).
- Frontend MasterQuestions.js: tombol 'Tambah Pertanyaan' + modal (group, section, jenis, text, definisi, satuan, opsi, wajib, priority, problem_when, report_required) + tombol hapus utk pertanyaan custom + badge 'baru'.
- Pertanyaan ceklis baru otomatis muncul di wizard kunjungan kader untuk group tsb.
- Testing agent: 100% backend & frontend, DB bersih.

## Fix + Features (2026-06, session 4)
- FIX "Network Error di preview": penyebab uvicorn --reload memegang CWD /app/backend yg sudah di-rm-rf saat re-import (FileNotFoundError getcwd) -> backend down intermiten. Solusi: clean supervisor restart. Terverifikasi testing agent 100% (login + semua menu admin, tanpa network error).
- FEATURE Edit Penuh Pertanyaan Custom: EditModal kini menampilkan field jenis, section, satuan, opsi, report_required KHUSUS pertanyaan custom (built-in tetap edit terbatas). Backend PUT /master/questions/{kode} disanitasi utk terima field penuh.
- FEATURE Impor Pertanyaan Excel: GET /admin/import/template/pertanyaan + POST /admin/import/pertanyaan (kolom group,section,text,jenis,satuan,opsi,wajib,priority,problem_when,report_required,definisi; opsi & problem_when dipisah ';'). Card baru 'Impor Pertanyaan' di halaman Import Data. Baris invalid dilewati & dilaporkan.
- Testing agent iteration_4: 100% backend & frontend, DB baseline.
