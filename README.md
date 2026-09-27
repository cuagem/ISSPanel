<div align="center">

# ⚡ IsatisStackTeam — ISSPanel

### پنل حرفه‌ای ساخت و مدیریت کانفیگ VLESS روی Railway

پنل رایگان ساخت و مدیریت کانفیگ‌های V2Ray/Xray با استفاده از Railway — با پینگ پایین و سرورهای آمریکا، هلند و سنگاپور.

**پروتکل‌های پشتیبانی‌شده:** `ws` · `gRPC` · `XHTTP` · **`Reality`** ⭐

[![Telegram](https://img.shields.io/badge/Telegram-IsatisStackTeam-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)](https://telegram.me/IsatisStackTeam)
[![YouTube](https://img.shields.io/badge/YouTube-IsatisStackTeam-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://www.youtube.com/@IsatisStack)

### 🌐 [مشاهده آموزش کار با پنل](https://youtu.be/jaYCnAm5Al8?si=qlheDmXF20dOf5oj)

</div>

---

## ✨ معرفی پروژه

ISSPanel یک پنل Master/Agent برای ساخت و توزیع کانفیگ‌های VLESS روی سرویس Railway است:

- **مدیریت چند پنل (Master/Remote):** پنل مرکزی می‌تواند چند سرور Agent را از راه دور مدیریت کند.
- **اینباندها:** ساخت اینباند با پروتکل‌های `ws`، `grpc`، `xhttp` و جدیداً **`Reality`**.
- **کاربران و اشتراک (Subscription):** هر کاربر یک UUID و لینک اشتراک اختصاصی (`/sub/:token` و `/sub64/:token`) با QR می‌گیرد.
- **تولید خودکار کانفیگ Xray:** پنل به‌صورت خودکار فایل کانفیگ Xray را می‌سازد، Xray را ری‌استارت می‌کند و ترافیک را روی مسیر درست پروکسی می‌کند (HTTP + WebSocket Upgrade).
- **پریست ضد فیلتر:** با یک کلیک، ۳ اینباند ws/grpc/xhttp + یک اینباند Reality ساخته می‌شود.

## 🚀 قابلیت Reality

پروتکل **Reality** امن‌ترین و سریع‌ترین حالت VLESS است: TLS واقعی سایت هدف را «قرض می‌گیرد»، نیازی به دامنه و گواهی ندارد و در برابر SNI-based detection مقاوم است.

هنگام انتخاب `reality` در فرم اینباند:

- کلید X25519 سراسری پنل به‌صورت خودکار تولید و در `data/reality-key.json` ذخیره می‌شود (یا از `REALITY_PRIVATE_KEY` در env).
- `Short ID` و `SpiderX` برای هر اینباند خودکار ساخته می‌شوند.
- `Dest` و `SNI` قابل تنظیم‌اند (پیش‌فرض `www.microsoft.com:443`).
- لینک VLESS خروجی به‌صورت خودکار شامل `security=reality`، `pbk`، `sid` و `spx` خواهد بود.

> نکته: Reality را با شبکه `tcp` (و ترجیحاً پورت `443`) استفاده کنید؛ این ترکیب سریع‌ترین حالت ممکن است.

## ⚡ بهینه‌سازی سرعت

کانفیگ‌های تولیدی پنل برای حداکثر سرعت بهینه شده‌اند:

| تنظیم | اثر |
|---|---|
| `sockopt.tcpFastOpen` | کاهش زمان برقراری اتصال (TFO) |
| `sockopt.tcpNoDelay` | حذف تأخیر Nagle برای ترافیک تعاملی |
| `sockopt.tcpKeepAliveIdle` | نگه‌داشتن اتصالات طولانی‌مدت بدون قطعی |
| `sockopt.tcpMaxSeg: 1440` | جلوگیری از fragmentation روی شبکه‌های مخابراتی |
| `sniffing.destOverride` با `quic` | مسیریابی دقیق‌تر ترافیک، جلوگیری از نشت DNS/QUIC |
| `dns` با `UseIPv4` و DoH محلی | رزولوشن سریع DNS بدون رفت‌وبرگشت IPv6 |
| `domainStrategy: UseIP` | اتصال مستقیم به IP — یک hops کمتر |
| `grpc.multiMode` | چندپلکسی gRPC برای توان عملیاتی بالاتر |

## 📦 نصب

### روش ۱: استقرار روی Railway (پیشنهادی)

1. این ریپو را Fork کنید و یک ریپوی جدید بسازید.
2. در [Railway](https://railway.app) یک پروژه جدید از ریپوی خودتان بسازید. Railway به‌صورت خودکار `Dockerfile` را می‌سازد (شامل آخرین نسخه Xray-Core).
3. متغیرهای محیطی (Variables) را تنظیم کنید (بخش متغیرها را ببینید).
4. یک دامنه (Domain) به سرویس وصل کنید؛ آدرس دامنه همان `Address` پنل شماست.
5. با باز کردن `https://<دامنه>/dash` وارد پنل شوید.

### روش ۲: اجرای محلی / سرور شخصی

```bash
# پیش‌نیاز: Node.js 18+ و Xray-Core در PATH
git clone <این-ریپو>
cd <پروژه>
npm install
cp .env.example .env    # مقادیر را ویرایش کنید
npm start               # پنل روی پورت 3000 بالا می‌آید
```

سپس مرورگر را روی `http://localhost:3000/dash` باز کنید.

## 🔧 متغیرهای محیطی

| متغیر | پیش‌فرض | توضیح |
|---|---|---|
| `NODE_ROLE` | `hybrid` | نقش نود: `master`، `agent` یا `hybrid` |
| `PORT` | `3000` | پورت وب‌پنل |
| `ADMIN_USER` | `admin` | نام کاربری مدیر |
| `ADMIN_PASS` | `admin` | رمز عبور مدیر — **حتماً تغییر دهید!** |
| `AGENT_KEY` | `change-me-agent-key` | کلید ارتباط Master ↔ Agent |
| `XRAY_BASE_PORT` | `10086` | پورت پایه داخلی Xray (پورت هر اینباند = پایه + id) |
| `DB_PATH` | `./data/configs.db` | مسیر دیتابیس SQLite |
| `PUBLIC_BASE_URL` | — | آدرس عمومی پنل (برای لینک‌های اشتراک) |
| `PUBLIC_HOST` | `localhost` | آدرس عمومی Agent |
| `REALITY_PRIVATE_KEY` | خودکار | کلید خصوصی X25519 برای Reality (اگر خالی باشد تولید و ذخیره می‌شود) |
| `REALITY_DEST` | `www.microsoft.com:443` | سایت هدف برای camoflage در Reality |
| `XRAY_BIN` | `xray` | مسیر باینری Xray |

## 📝 استفاده

1. **افزودن پنل:** نام و Address (دامنه) سرور را وارد کنید. برای سرورهای دیگر تیک Remote را بزنید و `API Base` + `API Key` (همان `AGENT_KEY` آن سرور) را بدهید.
2. **افزودن اینباند:** پروتکل (`ws`/`grpc`/`xhttp`/`tcp`)، Host و Path را وارد کنید. برای Reality: گزینه `reality` را در Security انتخاب کنید — pbk/sid/spx خودکار ساخته می‌شوند.
3. **کاربر سریع:** نام کاربری و UUID (با دکمه «UUID» تولید کنید) بدهید؛ کاربر ساخته و به همه اینباندهای پنل متصل می‌شود.
4. **پریست ضد فیلتر:** با یک کلیک اینباندهای ws + grpc + xhttp + Reality با مسیرهای تصادفی ساخته می‌شوند.
5. **لینک و QR:** برای هر کاربر دکمه «کپی همه لینک‌ها»، «کپی Subscription» و «QR Subscription» در جدول موجود است.

## 🔗 لینک اشتراک

- متن ساده: `https://<دامنه>/sub/<token>`
- Base64 (سازگار با اکثر کلاینت‌ها): `https://<دامنه>/sub64/<token>`

## 🗄 ساختار پروژه

```
├── server.js            # سرور Express + API + تولید کانفیگ Xray + پروکسی
├── public/
│   ├── index.html       # صفحه اصلی
│   ├── dash.html        # صفحه ورود
│   └── dash-view.html   # داشبورد مدیریت
├── Dockerfile           # ایمیج Node 18 + Xray-Core
├── data/                # دیتابیس SQLite + کلید Reality (در اجرا ساخته می‌شود)
└── .env.example         # نمونه متغیرهای محیطی
```

## 🔒 نکات امنیتی

- حتماً `ADMIN_PASS` و `AGENT_KEY` را تغییر دهید.
- `data/reality-key.json` را گم نکنید؛ اگر پاک شود، کلاینت‌های Reality قبلی با pbk قدیمی قطع می‌شوند. برای حفظ سازگاری، مقدار `REALITY_PRIVATE_KEY` را در env ثابت نگه دارید.

## پیشنهاد برای `XRAY_BASE_PORT`

`XRAY_BASE_PORT` پورت داخلی است که Xray از آن برای ساخت inboundها استفاده می‌کند.

| پورت | توضیح |
|---|---|
| `2086` | شبیه پورت‌های cPanel، مناسب |
| `2087` | گزینه مناسب |
| `2095` | گزینه مناسب |
| `2096` | گزینه مناسب |
| `4433` | شبیه HTTPS |
| `8443` | رایج برای HTTPS جایگزین |
| `10086` | مقدار پیش‌فرض رایج |
| `49152` تا `65535` | بازه dynamic/private |

---

<div align="center">

ساخته‌شده با ❤️ توسط [IsatisStackTeam](https://telegram.me/IsatisStackTeam)

</div>
