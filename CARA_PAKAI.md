# Cara pakai workflow dex2rs-protect

## 1. Taruh file di repo

```
repo-kamu/
  .github/workflows/dex2rs-protect.yml   <- copy dari folder ini
  Miku_Moe_3.7_0_jq7i.apk               <- APK input (nama bisa diubah saat Run)
```

File `filter.txt` tidak wajib di-copy, karena workflow sudah
membuatnya sendiri dari heredoc. File `filter.txt` di folder ini
adalah salinan yang sama, buat dipakai kalau kamu jalanin dex2rs
manual di lokal.

## 2. Jalankan

Otomatis: setiap push ke branch `main` langsung menjalankan workflow
ini. Tidak perlu tekan Run workflow lagi.

Manual juga bisa: GitHub repo > tab Actions > Dex2rs Protect Miku Moe
> Run workflow. Isi `apk_path` kalau APK tidak di root / namanya beda.
Isi `output_name` kalau mau ganti nama hasil.

## 3. Ambil hasil

Artifacts: `Miku_Moe-protected-dex2rs`
Isinya: `Miku_Moe_3.7_protected.apk`

Cek di dalam APK hasil, harus ada persis 2 ini:

```
lib/arm64-v8a/libmikumoe.so      (64-bit)
lib/armeabi-v7a/libmikumoe.so    (32-bit)
```

## 4. Command inti yang dijalankan workflow

```
dex2rs -i Miku_Moe_3.7_0_jq7i.apk \
  -o Miku_Moe_3.7_protected.apk \
  -f filter.txt \
  -p -d -C -k \
  --abis arm64-v8a,armeabi-v7a \
  -l mikumoe \
  -r -c miku.solid.moe
```

## 5. Yang harus kamu tahu sebelum install hasil

- Signing default dex2rs = debug keystore bawaan. Install fresh bisa,
  update-install menimpa APK release-signed tidak bisa. Kalau butuh
  update-install, re-sign dengan keystore rilis kamu.
- Build Rust untuk 459 class miku.moe.app + 2 ABI bisa makan waktu
  belasan menit. Workflow di-set timeout 90 menit.
- Workflow ini manual saja (workflow_dispatch), tidak jalan tiap push.

## Tambahan 2026-10-27: cari string miku di luar miku.moe.app
Workflow kini punya step "Cari string miku di luar miku.moe.app (+ perluas filter)"
sebelum protect. Step itu menjalankan dexdump -d atas semua classes*.dex,
mencari const-string berisi "miku" (case-insens), dan menambahkan kelas
top-level obfuscated penemunya (67 kelas di APK 3.7, mis. Lal2, Lcs0, Ltz3,
Lwg0, Lzu1) ke filter.txt sebagai L<kelas>;.*;.* . Kelas library penemu
(androidx.room.f, cuma nerima nama DB miku_manga_room.db) TIDAK ditambahkan.
Laporan lengkapnya diunduh dari artifact sebagai miku-strings-outside.txt,
dengan kelompok "true miku" dipisah dari "komiku" (Komiku mengandung substring
"miku" tapi itu sumber lain, bukan brand Miku Moe).

## Perbaruan: True Miku saja, OTOMATIS tiap run (bukan hardcode)
Step cari-string sekarang HANYA True Miku: string, setelah semua "komiku"
dihapus (lower-case), masih mengandung "miku". Ini mencakup miku_* dan
domain *mikumoe*, dan mengecualikan string Komiku murni. Kelas yang HANYA
pakai Komiku tidak ditambahkan; kelas yang punya True Miku (meski juga
punya Komiku) tetap ditambahkan karena True-nya.
PENTING: step ini jalan di SETIAP run, me-dexdump APK input run itu sendiri.
Jadi kalau update aplikasi lagi dan nama kelas R8 berubah total, script
tetap nyari sendiri kelas-kelas baru yang ada teks True Miku-nya, tanpa
edit daftar kelas. Daftar 55 kelas dari APK 3.7 di laporan hanya dokumentasi,
bukan sumber filter. Di APK 3.7, hasilnya: 55 kelas ditambahkan (total filter
56 aturan: 1 base + 55), 69 pemakaian Komiku-only tidak ditambahkan, library
androidx.room.f dilewati.

## Zip self-push: SCRIPT SAJA, tanpa APK
Zip `dex2rs-protect-mikumoe-repo-selfpush.zip` sengaja TIDAK berisi APK.
Alasannya: APK sudah ada di kamu, dan kalau APK lama ikut masuk repo,
workflow bisa salah pilih APK lama (default / *.apk pertama) padahal kamu
sudah push APK update. Jadi caranya: unzip zip ini ke repo, lalu taruh
APK kamu sendiri di root repo (nama bebas, workflow auto-detect), baru push.
