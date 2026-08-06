# CI / UAT - Playstore Workflow

README untuk cabang ci/uat-playstore-workflow. Cabang ini berisi konfigurasi dan workflow yang terkait dengan proses build dan release ke Google Play (UAT). Sesuaikan file workflow di .github/workflows sesuai kebutuhan.

## Apa yang ada di cabang ini
- .github/workflows/
  - workflow untuk build APK/AAB
  - langkah-langkah deploy ke internal testing / UAT (mis. menggunakan google-play-deploy action)
- skrip build khusus (jika ada)

## Menjalankan secara lokal
Untuk men-debug langkah yang dijalankan oleh GitHub Actions secara lokal, Anda bisa menggunakan act (https://github.com/nektos/act) atau menjalankan skrip build manual di mesin lokal.

Contoh menjalankan build (untuk proyek Android):
```bash
# contoh gradle
./gradlew assembleRelease
# atau untuk bundle
./gradlew bundleRelease
```

## Konfigurasi rahasia
Simpan secret di GitHub repository Settings → Secrets:
- GOOGLE_PLAY_JSON (file JSON dari Google Service Account untuk Play Console)
- SERVICE_ACCOUNT_EMAIL
- KEYSTORE (jika perlu)
- KEYSTORE_PASSWORD
- KEY_ALIAS
- KEY_PASSWORD

## Contoh workflow (singkat)
Contoh langkah GitHub Actions untuk deploy ke internal testing:
```yaml
name: Build & Deploy UAT
on:
  push:
    branches: [ ci/uat-playstore-workflow ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
      - name: Build bundle
        run: ./gradlew bundleRelease
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-bundle
          path: app/build/outputs/bundle/release/*.aab

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: app-bundle
      - name: Deploy to Google Play (Internal)
        uses: r0adkll/upload-google-play@v1
        with:
          serviceAccountJson: ${{ secrets.GOOGLE_PLAY_JSON }}
          packageName: com.example.app
          releaseFiles: '*.aab'
          track: internal
```

> Ganti packageName dan action deploy sesuai action yang Anda pilih (contoh di atas menggunakan r0adkll/upload-google-play).

## Catatan Keamanan
- Jangan commit file kunci/credential ke repo. Gunakan Secrets.
- Batasi akses branch proteksi untuk cabang release/UAT.

## Troubleshooting
- Jika build gagal karena dependensi, pastikan cache Gradle diatur dan versi Java sesuai.
- Untuk masalah upload, periksa format file (AAB vs APK) dan izin service account.

---
README ini dibuat otomatis. Silakan tinjau dan sesuaikan nama package, langkah build, dan action deploy sesuai kebutuhan proyek Anda.
