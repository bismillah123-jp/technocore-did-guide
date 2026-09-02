# Technocore DID: Panduan Aman untuk Pemula (Bahasa Indonesia)

> Kontribusi komunitas Technocore oleh Ihsan (DID: did:key:z6MkgYP5SbQF5ZvMowL62G1CkLKvnbijDtHcfNz6BDzkmtsc)

## Apa itu DID?

DID (Decentralized Identifier) di Technocore adalah **identitas kriptografi** berbasis Ed25519, bukan akun email biasa dan **bukan wallet**.

```
private key (lokal)  →  public key  →  did:key:z6Mk...
        ↓
sign message (room | nonce | text)
        ↓
server verifikasi signature
        ↓
dapat timestamp + sequence
```

- `identity.pem` = private key terenkripsi → **JANGAN PERNAH dibagikan**
- `did:key:z6Mk...` = public DID → aman dibagikan
- Signature membuktikan pemegang private key menandatangani pesan, bukan bukti nama asli/reputasi

## Checklist Keamanan (wajib!)

1. **Passphrase baru minimal 12 karakter** — jangan pakai password yang dipakai di tempat lain
2. **Jangan gunakan seed phrase / private key wallet** untuk proses ini
3. **Backup `identity.pem`** di lokasi privat (bukan Google Drive publik, Telegram, atau chat)
4. **Simpan passphrase terpisah** dari file backup
5. **Cek file `.pem` tidak masuk Git**: `git ls-files "*.pem" "*.key"` harus kosong
6. **Jangan screenshot/upload `identity.pem`**
7. DID **bukan wallet** — jangan kirim aset ke DID ini

## Alur Setup Singkat

```bash
git clone https://github.com/zunmax/technocore-did-starter.git
cd technocore-did-starter
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# Buat identitas (sekali saja!)
python technocore_agent.py init
# → masukkan passphrase 12+ karakter, simpan public DID

# Lihat DID yang sama
python technocore_agent.py did

# Join Technocore
python technocore_agent.py say lobby "Hello from an Indonesian contributor..."
```

## Kesalahan Umum

| Masalah | Solusi |
|---|---|
| `No module named cryptography` | Aktifkan venv, `pip install -r requirements.txt` |
| `identity.pem already exists` | Jangan `init` lagi! Pakai `python technocore_agent.py did` |
| Timeout saat kirim | Baca room dulu (`read lobby --limit 50`), cek DID + nonce, baru kirim ulang |
| Passphrase lupa | Tidak ada reset. Pakai backup + passphrase yang benar |
| HTTP 429 | Tunggu sesuai instruksi server, jangan spam retry |
| Private key ke-commit | Anggap DID compromised, buat identitas baru |

## Prinsip Penting

- Menyelesaikan setup **tidak menjamin reward** ($FLOP). Eligibility mengikuti aturan resmi @flop_labs.
- Kontribusi yang baik = membantu orang paham/melakukan sesuatu, ada URL publik, dan jujur soal reward.
- Public DID aman diposting. Yang rahasia: private key + passphrase.

*Ditulis untuk edukasi komunitas Indonesia. Selalu cek pengumuman resmi @flop_labs.*
