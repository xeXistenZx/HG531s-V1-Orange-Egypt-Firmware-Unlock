# Publishing To GitHub

## Create The Repository

1. Go to https://github.com/new
2. Repository name: `hg531s-v1-tedata-unlock`
3. Description: `One-instruction signature bypass to cross-flash Huawei HG531s V1 (Orange Egypt) to Tedata HG531 V1 firmware — unlocking Ethernet WAN / FTTH`
4. Public
5. Do NOT initialise with README (you already have one)
6. Click Create repository

## Push The Files

```bash
cd hg531s-v1-tedata-unlock
git init
git add .
git commit -m "Initial commit: HG531s V1 Tedata unlock — one-instruction signature bypass"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/hg531s-v1-tedata-unlock.git
git push -u origin main
```

## GitHub Topics To Add

After pushing, go to the repository page, click the gear icon next to About, and add these topics:

```
huawei hg531 orange-egypt tedata ftth firmware-mod
rtl8676s signature-bypass uart bootloader mips router
```

## What NOT To Include

- Do not upload the `.bin` firmware files (vendor binaries)
- Do not upload your 4 MB full-flash backup
- Do not upload decompressed loader images (they contain vendor code)
- The `docs/` and `scripts/` files are the repository's substance
