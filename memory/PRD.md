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
