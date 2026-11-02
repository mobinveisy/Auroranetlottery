# 🌌 AURORA NET — سایت قرعه‌کشی (نسخه GitHub Pages)

طراحی فارسی/RTL، متحرک، ریسپانسیو، با تصویر اصلی آرورا، فرم ثبت اشتراک، اجازه اختیاری برای ارسال مشخصات دستگاه و موقعیت تقریبی، اعلان در ربات تلگرام، فضای ذخیره‌سازی و صفحه مدیریت با خروجی CSV و انتخاب برنده.

**ساختار:** `site/` سایت استاتیک؛ `cloudflare/worker.js` بک‌اند امن. GitHub Pages به‌تنهایی نمی‌تواند تلگرام را به‌طور امن فراخوانی کند یا داده را ذخیره کند.

## مراحل راه‌اندازی فقط با گوشی

### ۱) ساخت مخزن GitHub و آپلود سایت

1. فایل ZIP را با برنامه Files / ZArchiver / مدیریت فایل گوشی **استخراج** کنید (ZIP را مستقیم به عنوان وب‌سایت آپلود نکنید).
2. در مرورگر گوشی `github.com/new` را باز کنید. مخزنی مانند `aurora-raffle` بسازید. روی GitHub Free، مخزن برای انتشار از GitHub Pages باید **Public** باشد. مخزن Private برای Pages در پلن‌های پشتیبانی‌شده مانند GitHub Pro مجاز است.
3. وارد مخزن شوید؛ با گزینه **Add file → Upload files** محتوای پوشه `site` را در ریشه مخزن آپلود کنید. ساختار در ریشه باید `index.html`، `style.css`، `app.js`، `config.js`، `admin.html`، `admin.js`، `admin.css` و پوشه `assets` باشد. خود پوشه `site` نباید یک لایه اضافه باشد.
4. **Settings → Pages → Build and deployment** را باز کنید؛ **Deploy from a branch**، شاخه `main` و مسیر `/ (root)` را انتخاب کنید؛ Save.
5. لینک سایت به شکل `https://USERNAME.github.io/aurora-raffle/` خواهد بود. مدتی پس از انتشار، لینک در Pages ظاهر می‌شود.

**نکته:** سایت منتشرشده از GitHub Pages معمولاً عمومی است، حتی اگر مخزن Private باشد. برای سایت خصوصی واقعی به کنترل دسترسی جداگانه نیاز است.

### ۲) ربات تلگرام برای دریافت ثبت‌نام‌ها

1. در تلگرام به **@BotFather** بروید، ` /newbot ` را بزنید و ربات بسازید؛ توکن (BOT_TOKEN) را **محرمانه نگه دارید**.
2. ربات را باز کنید و به آن `/start` بفرستید.
3. `CHAT_ID` شخصی خودتان یا گروه مورد نظر را پیدا کنید؛ می‌توانید پس از ارسال `/start`، پاسخ `getUpdates` از **API رسمی Telegram** را در محیط امن خود بررسی کنید. از اشتراک‌گذاری لینک حاوی توکن، عکس توکن یا قرار دادن آن در فایل‌های GitHub خودداری کنید. برای گروه، ابتدا ربات را به گروه اضافه کنید.

### ۳) ساخت بک‌اند رایگان/کم‌هزینه در Cloudflare

1. به `dash.cloudflare.com` بروید، در بخش **Workers & Pages** یک Worker با نام دلخواه مثلاً `aurora-raffle-api` بسازید.
2. با **Edit code** محتوای فایل `cloudflare/worker.js` را به‌طور کامل جایگزین کرده و Deploy کنید.
3. در بخش **KV** یک KV namespace جدید با نام مثلاً `AURORA_ENTRIES` بسازید. در تنظیمات Worker، **Settings → Bindings → Add → KV Namespace** را انتخاب کنید؛ نام متغیر binding را **دقیقاً `REGISTRATIONS`** بگذارید و namespace را به آن متصل کنید.
4. در **Worker → Settings → Variables and Secrets** موارد زیر را اضافه کنید و Deploy را اعمال کنید:

   | نام | نوع | مقدار |
   | --- | --- | --- |
   | `BOT_TOKEN` | **Secret** | توکن ربات از BotFather |
   | `CHAT_ID` | **Secret** | شناسه چت دریافت‌کننده (عدد یا شناسه کانال مجاز) |
   | `ADMIN_KEY` | **Secret** | یک رمز تصادفی طولانی، ترجیحاً بیش از ۳۲ کاراکتر |
   | `ALLOWED_ORIGINS` | **Text** | مثلاً `https://USERNAME.github.io` |

   مقدار `ALLOWED_ORIGINS` فقط **origin** دامنه است؛ بدون اسم ریپو و بدون `/` آخر. برای دامنه اختصاصی می‌توانید چند مبدا را با کاما جدا کنید: `https://USERNAME.github.io,https://giveaway.example.com`.
5. به آدرس `https://YOUR-WORKER.workers.dev/api/health` بروید و ببینید `"configured": true` شده باشد. ممکن است آدرس واقعی Worker با نام/حساب Cloudflare شما متفاوت باشد.
6. در `site/config.js` مقدار `API_URL` را **فقط با آدرس عمومی Worker** پر کنید، مثلاً:

   ```js
   window.AURORA_CONFIG = Object.freeze({
     API_URL: "https://aurora-raffle-api.YOUR-ACCOUNT.workers.dev",
     BRAND: "AURORA NET"
   });
   ```

7. `config.js` را با GitHub Edit و Commit تغییر دهید. حالا فرم به Worker وصل است.
8. با یک شناسه آزمایشی ثبت کنید، رسید دریافت کنید و پیام ربات تلگرام را بررسی کنید. اگر خطای اتصال دیدید، مقدار `ALLOWED_ORIGINS` و API_URL را چک کنید. فرم تا وقتی Worker تنظیم نشود **ثبت واقعی انجام نمی‌دهد**.

### ۴) پنل مدیریت و قرعه‌کشی

- آدرس: `https://USERNAME.github.io/aurora-raffle/admin.html`
- با همان مقدار `ADMIN_KEY` (Secret) وارد شوید. کلید در مرورگر ذخیره نمی‌شود و به‌صورت هدر HTTPS به Worker ارسال می‌شود.
- فهرست را جستجو کنید، با **دریافت CSV** خروجی بگیرید و با **انتخاب برنده** قرعه‌کشی کنید.
- انتخاب برنده با مولد تصادفی رمزنگاری‌شده انجام می‌شود و نتیجه به عنوان `draw:main` ذخیره می‌شود؛ پس از انتخاب، از خود پنل دیگر امکان قرعه مجدد وجود ندارد. پیام برنده هم برای ربات ارسال می‌شود.
- **مهم:** انتخاب تصادفی روی ثبت‌نام‌های دریافت‌شده است، نه الزاماً کاربران احراز هویت‌شده. ثبت با نام قابل اثبات نیست. برای قرعه‌کشی‌های دارای ارزش بالا لازم است شناسه‌ها با پایگاه اشتراک تطبیق داده شوند؛ همچنین Workers KV از سازگاری نهایی استفاده می‌کند و برای ثبت همزمان زیاد جایگزین پایگاه داده تراکنشی نیست.

## حریم خصوصی و ضدسوءاستفاده

- مشخصات دستگاه فقط با تیک اختیاری کاربر جمع‌آوری و ارسال می‌شود.
- موقعیت مکانی فقط با کلیک روی دکمه و تأیید اجازه مرورگر به دست می‌آید؛ مختصات قبل از ارسال به ۳ رقم اعشار (تقریباً ۱۰۰ متر در عرض‌های میانی جغرافیایی) گرد می‌شود.
- نشانی IP خام در داده ثبت‌نام یا پیام تلگرام ارسال/ذخیره نمی‌شود؛ فقط یک هش موقت برای محدود کردن درخواست‌های تکراری، با انقضای ۱۰ دقیقه، استفاده می‌شود.
- ورودی‌های KV پس از **۹۰ روز** به‌صورت خودکار منقضی می‌شوند؛ اما پیام‌های ارسال‌شده به تلگرام به‌صورت خودکار پاک نمی‌شوند. مدیریت حذف این پیام‌ها با برگزارکننده است.
- شناسه اشتراک به‌طور عادی از ثبت مجدد جلوگیری می‌کند؛ ثبت با نام امکان تکرار دارد. برای تبلیغات گسترده، Cloudflare Turnstile، احراز هویت اشتراک و پایگاه داده D1 پیشنهاد می‌شوند.
- صفحه `admin.html` از نظر URL عمومی است، ولی اطلاعات و اقدامات آن در Worker با کلید `ADMIN_KEY` محافظت می‌شوند. صفحه را به کاربران تبلیغ نکنید.
- برای ثبت‌نام عمومی واقعی، قوانین شرکت، زمان قرعه‌کشی و جایزه را قبل از انتشار نهایی اعلام کنید. نمونه حاضر هیچ جایزه/زمان ساختگی ذکر نکرده است.

## فایل‌ها

```text
aurora-net-raffle/
├── site/                  ← محتوای قابل آپلود در ریشه GitHub Pages
│   ├── index.html         ← لندینگ‌پیج فارسی
│   ├── style.css          ← افکت‌ها، انیمیشن‌ها، ریسپانسیو
│   ├── app.js             ← فرم و رضایت کاربر
│   ├── config.js          ← فقط لینک Worker؛ هیچ توکنی اینجا نگذارید
│   ├── admin.html         ← پنل مدیر
│   ├── admin.css
│   ├── admin.js
│   ├── .nojekyll
│   └── assets/aurora-original.webp / aurora-thumb.webp
├── cloudflare/
│   └── worker.js          ← API و تلگرام و ذخیره‌سازی امن
└── README-FA.md
```

## لینک‌های رسمی استفاده‌شده

- خرید و تمدید: https://t.me/Auroraserverbot
- پشتیبانی: https://t.me/Auroranetsupport
- کانال: https://t.me/auroranetvpn
- وضعیت سرویس‌ها: https://abraservice.xyz/status/aurora

## منابع رسمی فنی

- GitHub Pages: https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site
- Cloudflare KV Binding: https://developers.cloudflare.com/kv/concepts/kv-namespaces/
- Cloudflare Secrets: https://developers.cloudflare.com/workers/configuration/secrets/
- Telegram Bot API: https://core.telegram.org/bots/api#sendmessage
