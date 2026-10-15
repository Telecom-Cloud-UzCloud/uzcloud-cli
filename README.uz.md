# UzCloud CLI

[🇬🇧 English](README.md) | [🇷🇺 Русский](README.ru.md) | [🇺🇿 O'zbekcha](README.uz.md)

`uzc` bu [UzCloud](https://uzcloud.uz/) uchun buyruqlar qatori utilitasi. U virtual mashinalar, disklar, tarmoqlar, Kubernetes klasterlari, yuk balansirovkachilari, DNS va billingni boshqaradi. [Terraform provayderi](https://registry.terraform.io/providers/telecom-cloud-uzcloud/uzcloud/latest) bilan bir xil API va bir xil API-mijoz orqali ishlaydi.

## O'rnatish

[Releases](https://github.com/Telecom-Cloud-UzCloud/uzcloud-cli/releases) sahifasidan platformangiz uchun arxivni yuklab oling va `uzc` ni `PATH` dagi katalogga joylashtiring.

macOS va Linux:

```bash
VERSION=1.0.3
OS=darwin      # yoki linux
ARCH=arm64     # yoki amd64
curl -LO https://github.com/Telecom-Cloud-UzCloud/uzcloud-cli/releases/download/v$VERSION/uzcloud_cli_${VERSION}_${OS}_${ARCH}.tar.gz
tar -xzf uzcloud_cli_${VERSION}_${OS}_${ARCH}.tar.gz uzc
sudo mv uzc /usr/local/bin/
uzc version
```

Windows: `uzcloud_cli_<versiya>_windows_amd64.zip` ni yuklab oling, arxivdan chiqaring va `uzc.exe` joylashgan papkani `PATH` ga qo'shing.

Har bir relizga Debian/Ubuntu, RHEL/Fedora va Alpine paketlari (`.deb`, `.rpm`, `.apk`) ham biriktirilgan.

Yuklangan faylni o'sha relizdagi `uzcloud_cli_<versiya>_SHA256SUMS` orqali tekshiring:

```bash
shasum -a 256 -c uzcloud_cli_${VERSION}_SHA256SUMS --ignore-missing
```

macOS da binar fayl hali notarizatsiyadan o'tmagan. Agar Gatekeeper uni bloklasa, `xattr -d com.apple.quarantine /usr/local/bin/uzc` ni bajaring.

## Tez boshlash

UzCloud portalida API-token chiqaring (**Profile -> API**), so'ng:

```bash
uzc init                       # tokenni va standart loyiha/region/provayder/billing siklini saqlaydi
uzc region list
uzc plan list                  # standart regiondagi VM tariflari
uzc template list
uzc ssh-key create laptop      # ~/.ssh/id_ed25519.pub ni yuklaydi
uzc vm create web-1 --template ubuntu-22-04 --plan box2cm2 --ssh-key laptop --wait
uzc vm list
```

## Sozlamalar

Ustuvorlik tartibi: **flag -> muhit o'zgaruvchisi -> profil**.

| Sozlama          | Flag               | Muhit o'zgaruvchisi  |
|------------------|--------------------|----------------------|
| API-token        | `--token`          | `UZC_API_TOKEN`      |
| API manzili      | `--endpoint`       | `UZC_API_ENDPOINT`   |
| Loyiha           | `--project`        | `UZC_PROJECT`        |
| Region           | `--region`         | `UZC_REGION`         |
| Bulut provayderi | `--cloud-provider` | `UZC_CLOUD_PROVIDER` |
| Billing sikli    | `--billing-cycle`  | `UZC_BILLING_CYCLE`  |
| Profil           | `--profile`, `-p`  | `UZC_PROFILE`        |
| Konfiguratsiya fayli | `--config`     | `UZC_CONFIG`         |

O'zgaruvchilar nomlari Terraform provayderi bilan bir xil, shuning uchun ikkala vosita bitta muhitdan foydalana oladi. Profillar `<user config dir>/uzcloud/config.yaml` faylida `0600` huquqlari bilan saqlanadi.

```bash
uzc config set region tashkent
uzc config view
uzc init --profile staging
uzc config use staging
uzc -p prod vm list
```

## Chiqish formati

Barcha buyruqlar `-o table` (standart), `-o json` va `-o yaml` formatlarini qo'llab-quvvatlaydi. Holat xabarlari stderr ga yoziladi, shuning uchun stdout ni `jq` ga uzatish mumkin:

```bash
uzc vm list -o json | jq -r '.[] | select(.state=="Running") | .slug'
uzc vm list -q | xargs uzc vm stop   # -q: faqat sluglar
```

## Buyruqlar

To'liq ro'yxat: `uzc --help`, har bir buyruq bo'yicha batafsil: `uzc <buyruq> --help`. Asinxron amallar `--wait` flagini qabul qiladi. O'chiruvchi amallar `--yes` berilmasa, tasdiqlashni so'raydi.

Alohida buyrug'i bo'lmagan endpointlar uchun `uzc api` mavjud. U xuddi shu avtorizatsiya, qayta urinishlar va xatolarni qayta ishlash bilan ishlaydi:

```bash
uzc api GET /account/balance
uzc api POST /account/coupons -F coupon_code=FREE100
uzc api POST /autoscale -d @autoscale.json
```

Avtoto'ldirish: `uzc completion zsh|bash|fish|powershell`. Resurs sluglari ham to'ldiriladi: `uzc vm stop <TAB>`.
