---
pattern: "เมื่อ command fail ด้วย FileNotFoundError, ตรวจสอบ pwd ก่อนสรุปว่า file ไม่มี — working directory อาจไม่ใช่ที่คิด ใช้ absolute path หรือ cd ก่อนทุกครั้ง"
date: 2026-10-06
source: rrr: gac-oracle
concepts: [WSL, path-resolution, troubleshooting]
---

## Context

Session แรกของ gac-oracle repo — พยายาม extract ZIP file ด้วย Python zipfile แต่ fail ด้วย `FileNotFoundError` เพราะ Python process ทำงานที่ `/home/gacai/gac-oracle/` แต่ ZIP อยู่ที่ `/home/gacai/gac-oracle/gac-oracle-main.zip` — path ถูกต้องแต่ working directory ผิด

## How to transfer

ทุกครั้งที่ต้องการทำงานกับไฟล์ ใช้ absolute path เสมอ หรือ `cd` ไปที่ file อยู่ก่อนรัน command — โดยเฉพาะใน WSL ที่มี nested filesystem boundaries

```bash
# แทนที่จะรันที่ working directory ปัจจุบัน
python3 -c "import zipfile; zipfile.ZipFile('gac-oracle-main.zip').extractall('.')"

# ใช้ absolute path
python3 -c "import zipfile; zipfile.ZipFile('/home/gacai/gac-oracle/gac-oracle-main.zip').extractall('/home/gacai/gac-oracle/')"

# หรือ cd ก่อน
cd /home/gacai/gac-oracle && python3 -c "import zipfile; zipfile.ZipFile('gac-oracle-main.zip').extractall('.')"
```
