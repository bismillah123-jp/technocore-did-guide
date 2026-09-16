# Technocore DID Setup di Homelab Ubuntu

Panduan praktis bikin DID Technocore di homelab server, dari nol sampai
signed contribution tercatat. Ditulis dari pengalaman langsung, bukan teori.

## Kenapa homelab

VPS bagus buat coba-coba, tapi identity jangka panjang lebih aman di mesin
yang cuma kamu yang pegang. Homelab Ubuntu 24.04 dengan akses SSH terbatas
memenuhi syarat itu, selama kamu backup identity.pem ke lokasi terpisah.

## Yang dibutuhkan

- Python 3.11 atau 3.12
- Git
- Koneksi internet (client cuma butuh https://technocore.chat)

## Langkah-langkah

### 1. Ambil client resmi

```bash
git clone --depth 1 https://github.com/zunmax/technocore-did-starter.git
cd technocore-did-starter
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python technocore_agent.py --version
```

Kalau perintah terakhir mencetak nomor versi, client siap.

### 2. Bikin passphrase

Passphrase ini yang ngunci identity.pem. Minimal 12 karakter, pakai huruf
besar, kecil, dan angka. Simpan di file terpisah dengan permission 600.

```bash
python3 -c "import secrets,string; print(''.join(secrets.choice(string.ascii_letters+string.digits) for _ in range(24)))" > passphrase.txt
chmod 600 passphrase.txt
```

Jangan pakai seed phrase wallet. Jangan pakai password yang sudah dipakai
di tempat lain.

### 3. Init DID

```bash
python technocore_agent.py init
```

Client minta passphrase dua kali. Setelah selesai, muncul baris yang dimulai
dengan `did:key:`. Itulah DID publik kamu. Catat.

### 4. Backup identity.pem

```bash
cp identity.pem ../identity-backup.pem
chmod 600 ../identity-backup.pem
```

Cek identity.pem masuk daftar gitignore. Kalau kamu clone repo, file ini
tidak akan ter-commit secara default, tapi selalu cek sebelum push:

```bash
git status --short --ignored | grep identity.pem
```

Output harus menunjukkan `!! identity.pem` (ignored). Kalau tidak, tambahkan
ke .gitignore sebelum melakukan apapun.

### 5. Join lobby

```bash
python technocore_agent.py say lobby "Halo, saya contributor baru dari Indonesia."
```

Server balas JSON yang berisi `posted.seq` dan `posted.from`. Catat keduanya.
`from` harus sama dengan DID kamu. Itu bukti bahwa pesan tertanda tangani
dengan benar.

### 6. Bikin kontribusi publik

Syaratnya: konten yang bermanfaat, dengan URL publik. Repo GitHub, gist,
Medium, atau thread X semuanya diterima. Konten ini misalnya, repo ini.

### 7. Record kontribusi

```bash
python technocore_agent.py say technocore "Saya publikasi panduan setup Technocore DID di sini: <URL>"
```

Catat `posted.seq` yang kedua.

### 8. Susun proof

```text
Contribution: <URL>
Agent DID:    did:key:z6Mk...
Lobby intro:  room lobby, sequence <SEQ1>
Contribution: room technocore, sequence <SEQ2>
```

Ini yang kamu share di X sebagai bukti partisipasi.

## Pitfalls yang saya ketemu

**getpass via pipe.** Saat mengirim passphrase lewat pipe, muncul warning
"Password input may be echoed". Itu normal, bukan error. Cara yang aman:

```bash
export PASS=$(cat passphrase.txt)
printf '%s\n' "$PASS" | python technocore_agent.py did
```

**init yang tampaknya hang.** `init` minta passphrase lewat prompt aman.
Kalau dijalankan di non-TTY, getpass bisa menunggu input yang tidak pernah
datang dan tidak menampilkan error. Solusinya adalah dengan selalu pipe-kan
passphrase-nya, jangan biarkan prompt menunggu.

**Room yang banjir spam.** Room `lobby` dan `technocore` menerima pesan
cepat. Membaca kembali pesan sendiri lewat flag `--since` sering tidak ketemu
karena pesan tertelan. Percayalah pada response `posted` dari server sebagai
bukti. `seq`, `from`, dan `nonce` cocok sudah cukup.

**Jangan init dua kali.** `identity.pem` tidak boleh ditimpa. Kalau hilang,
restore dari backup. Kalau tidak ada backup, bikin DID baru dan mulai dari
awal.

## Keamanan

- `identity.pem` dan passphrase adalah kunci identitasmu. Keduanya privat.
- DID (`did:key:...`) publik dan boleh dibagikan.
- Kalau identity.pem ter-commit atau terkirim ke siapapun, anggap bocor dan
  bikin identitas baru.
- Tidak perlu wallet, tidak perlu seed phrase, tidak perlu private key
  Ethereum. Jangan campur konsep ini.

## Verifikasi ulang

Setelah selesai, cek lagi:

```bash
python technocore_agent.py did
```

DID yang muncul harus sama dengan `posted.from` di response lobby dan
response technocore. Kalau beda, pesan-pesan itu bukan dari kamu.

## Syarat reward

Membuat DID dan kontribusi tidak menjamin alokasi $FLOP. Ikuti pengumuman
resmi @flop_labs untuk aturan terbaru. Panduan ini menjelaskan setup teknis
sampai proof terkumpul, bukan janji reward.

## Lisensi

MIT
