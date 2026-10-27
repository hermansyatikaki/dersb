# Dex2rs Protect Miku Moe

Repo ini berisi APK Miku Moe 3.7 + workflow GitHub Actions untuk
memproteksinya dengan dex2rs (Dalvik bytecode -> Rust native).

## Isi repo

- APK input — TIDAK ikut di zip self-push, taruh APK kamu sendiri di root repo (nama bebas, workflow auto-detect)
- `.github/workflows/dex2rs-protect.yml` — workflow protect
- `filter.txt` — filter (salinan, workflow membuat sendiri dari heredoc)
- `CARA_PAKAI.md` — panduan singkat

## Cara jalan

Setiap push ke branch `main` langsung menjalankan workflow
`Dex2rs Protect Miku Moe`. Hasilnya ada di tab Actions > Artifacts:
`Miku_Moe-protected-dex2rs` berisi `Miku_Moe_3.7_protected.apk`.

## Spesifikasi protect

- Semua kelas `miku.moe.app` (filter `Lmiku/moe/app/.*;.*`)
- 2 ABI: `arm64-v8a` (64-bit) dan `armeabi-v7a` (32-bit)
- Library: `libmikumoe.so` per ABI
- Loader custom: `miku.solid.moe` (dipaksa dengan `-r`)
- Hardening: `-p -d -C` (obfuscate, dynamic-register, compress-libs)
- Filter diperluas OTOMATIS tiap run: kelas di luar `miku.moe.app` yang pakai string True Miku (55 kelas di APK 3.7, total filter 56 aturan; Komiku-only tidak ditambahkan, lihat `miku-strings-outside.txt`)

## Peringatan signing

Output di-sign debug keystore bawaan dex2rs. Bisa install fresh,
tapi tidak bisa update-install menimpa APK yang di-sign keystore
rilis. Untuk update-install, re-sign output dengan keystore rilis.
