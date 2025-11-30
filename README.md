````markdown
# 🌍 ipmyp API – سرویس GeoIP ساده، سریع و خوش‌دست

`ipmyp API` یک سرویس **تشخیص موقعیت و اطلاعات شبکه بر اساس IP یا دامنه** است که به‌صورت RESTful در دسترس است:

```text
https://api.ipmyp.ir
````

با چند درخواست ساده می‌تونی از روی IP اطلاعاتی مثل کشور، شهر، ISP، سازمان، موقعیت جغرافیایی و… رو بگیری و توی پروژه‌هات استفاده کنی 🚀

---

## ✨ ویژگی‌ها

* 🌐 پشتیبانی از جست‌وجو براساس:

  * IP (IPv4 و IPv6)
  * دامنه (Domain)
  * IP خود کلاینت (اگر query ندی)
* 📍 اطلاعات مکانی:

  * قاره، کشور، استان/منطقه، شهر، کدپستی، مختصات (lat/lon)، timezone
* 🛰 اطلاعات شبکه:

  * `as`, `asname`, `isp`, `org` بر اساس دیتابیس ASN
* 🎚 فیلتر کردن خروجی با `fields=`
* 🔁 درخواست‌های گروهی با `/batch`
* 📡 پشتیبانی از JSONP با `callback=`
* ⚙️ مناسب برای:

  * وب‌سایت‌ها و SPAها
  * بک‌اند APIها
  * سیستم‌های مانیتورینگ، آنالیتیکس و امنیت

---

## 📌 Endpoint ها

### 1️⃣ `GET /json/:query?`

**Base URL:**

```text
https://api.ipmyp.ir/json/
```

**مثال‌ها:**

```text
https://api.ipmyp.ir/json/8.8.8.8
https://api.ipmyp.ir/json/google.com
https://api.ipmyp.ir/json/           (بدون query → IP خود کلاینت)
https://api.ipmyp.ir/json/1.1.1.1?fields=status,country,city,isp,org
https://api.ipmyp.ir/json/8.8.8.8?callback=myFunc
```

* `query` می‌تواند:

  * IP (مثلاً `8.8.8.8`)
  * دامنه (مثلاً `google.com`)
  * خالی (در این صورت IP درخواست‌کننده استفاده می‌شود)

---

### 2️⃣ `POST /batch`

برای وقتی که می‌خوای **چند IP/دامنه را در یک درخواست** بررسی کنی.

**URL:**

```text
https://api.ipmyp.ir/batch
```

**Body (JSON Array):**

```json
[
  "8.8.8.8",
  "1.1.1.1",
  { "query": "google.com" },
  "",
  null
]
```

* هر آیتم می‌تواند:

  * یک `string` (IP/دامنه)
  * یا یک آبجکت `{ "query": "..." }`
  * اگر خالی / null باشد، IP خود کلاینت استفاده می‌شود.

**مثال با curl:**

```bash
curl -X POST https://api.ipmyp.ir/batch \
  -H "Content-Type: application/json" \
  -d '["8.8.8.8","1.1.1.1","google.com"]'
```

---

## 🧾 ساختار پاسخ (Response)

### ✅ موفق (`status: "success"`)

نمونه پاسخ:

```json
{
  "status": "success",
  "continent": "North America",
  "continentCode": "NA",
  "country": "United States",
  "countryCode": "US",
  "region": "CA",
  "regionName": "California",
  "city": "Mountain View",
  "district": null,
  "zip": "94043",
  "lat": 37.4056,
  "lon": -122.0775,
  "timezone": "America/Los_Angeles",
  "offset": null,
  "currency": null,
  "isp": "GOOGLE",
  "org": "GOOGLE",
  "as": "AS15169 GOOGLE",
  "asname": "GOOGLE",
  "reverse": null,
  "mobile": false,
  "proxy": false,
  "hosting": false,
  "query": "8.8.8.8"
}
```

> بسته به IP و دیتابیس، بعضی فیلدها ممکن است `null` باشند (مثلاً city یا zip برای بعضی IPها).

---

### ❌ خطا (`status: "fail"`)

مثلاً وقتی query نامعتبر باشد:

```json
{
  "status": "fail",
  "message": "invalid query",
  "query": "not-an-ip"
}
```

---

## 🎚 پارامترهای مفید

### `fields` – انتخاب فیلدهای خروجی

اگر نیاز به همه‌ی فیلدها نداری، می‌تونی فقط چند تا رو درخواست بدی:

```text
https://api.ipmyp.ir/json/8.8.8.8?fields=status,country,city,isp,org,as
```

پاسخ:

```json
{
  "status": "success",
  "country": "United States",
  "city": "Mountain View",
  "isp": "GOOGLE",
  "org": "GOOGLE",
  "as": "AS15169 GOOGLE"
}
```

### `callback` – JSONP

برای سناریوهایی که نیاز به JSONP داری (مثلاً استفاده با `<script>`):

```text
https://api.ipmyp.ir/json/8.8.8.8?callback=myFunc
```

خروجی:

```js
myFunc({
  "status": "success",
  "country": "United States",
  ...
});
```

---

## 🧪 مثال‌های کاربردی (Use Cases)

### 1️⃣ فرانت‌اند – تشخیص لوکیشن کاربر

```html
<script>
  fetch('https://api.ipmyp.ir/json/')
    .then(res => res.json())
    .then(data => {
      console.log('Country:', data.country);
      console.log('City:', data.city);
      // مثال: تغییر زبان/تم بر اساس کشور
      if (data.countryCode === 'IR') {
        document.documentElement.lang = 'fa';
        document.body.classList.add('rtl');
      }
    })
    .catch(console.error);
</script>
```

🎯 کاربردها:

* تنظیم زبان و جهت سایت (RTL/LTR)
* نمایش پیام‌ها/تخفیف‌ها بر اساس کشور
* ریدایرکت به ساب‌دامین منطقه‌ای

---

### 2️⃣ بک‌اند Node.js – لاگ‌گیری با GeoIP

```js
const axios = require('axios');

async function logRequest(req) {
  const ip =
    req.headers['x-forwarded-for']?.split(',')[0].trim() ||
    req.socket.remoteAddress;

  const { data } = await axios.get(
    `https://api.ipmyp.ir/json/${ip}?fields=status,country,city,isp,org,as,query`
  );

  console.log({
    path: req.url,
    ip: data.query,
    country: data.country,
    city: data.city,
    isp: data.isp,
    as: data.as,
  });
}
```

🎯 کاربردها:

* اضافه کردن GeoIP به لاگ‌های HTTP
* تحلیل کاربران براساس کشور/ISP
* ارسال داده‌ی غنی‌شده به ELK / Loki / Grafana

---

### 3️⃣ امنیت – محدود کردن دسترسی براساس کشور / ISP

```js
const axios = require('axios');

async function securityMiddleware(req, res, next) {
  const ip =
    req.headers['x-forwarded-for']?.split(',')[0].trim() ||
    req.socket.remoteAddress;

  const { data } = await axios.get(
    `https://api.ipmyp.ir/json/${ip}?fields=status,countryCode,isp,org,query`
  );

  if (data.status !== 'success') {
    return res.status(403).send('Forbidden');
  }

  const blockedCountries = ['CN', 'RU'];
  const blockedISPs = ['Some Data Center'];

  if (blockedCountries.includes(data.countryCode)) {
    return res.status(403).send('Access from your region is restricted');
  }

  if (blockedISPs.includes(data.isp)) {
    return res.status(403).send('Access blocked');
  }

  next();
}
```

🎯 کاربردها:

* محدود کردن پنل‌های ادمین به چند کشور/ISP خاص
* سخت‌تر کردن دسترسی Botها و دیتاسنترها
* فعال کردن captcha یا احراز هویت اضافه برای مناطق خاص

---

### 4️⃣ Batch پردازش – غنی‌سازی لیست IPها

```js
const axios = require('axios');

async function enrichIPs(ipList) {
  const { data } = await axios.post(
    'https://api.ipmyp.ir/batch?fields=status,country,city,isp,org,query',
    ipList,
    { headers: { 'Content-Type': 'application/json' } }
  );

  data.forEach(item => {
    console.log(
      `${item.query} => ${item.country} / ${item.city} / ${item.isp}`
    );
  });
}

enrichIPs(['8.8.8.8', '1.1.1.1', 'google.com']);
```

🎯 کاربردها:

* غنی‌سازی لاگ‌های قبلی (offline processing)
* تحلیل reportهای امنیتی
* ساخت داشبوردهای جغرافیایی از ترافیک

---

## ⏱ Rate Limit

(مقادیر واقعی ممکن است بسته به تنظیمات سرویس تغییر کند.)

* محدودیت تقریبی برای هر IP در یک بازه‌ی زمانی (مثلاً در هر دقیقه).
* دو هدر مهم در همه‌ی پاسخ‌ها:

```http
X-Rl: <remaining_requests>
X-Ttl: <seconds_until_reset>
```

مثال:

```http
X-Rl: 12
X-Ttl: 24
```

اگر محدودیت رد شود، پاسخ نمونه:

```json
{
  "status": "fail",
  "message": "rate limit exceeded",
  "query": "1.2.3.4"
}
```

---

## 🧑‍💻 نکات توسعه و مشارکت

اگر:

* ایده‌ای برای فیلدهای جدید داری (مثل `currency`, `offset`, `reverse`)
* پیشنهاد بهبود performance / امنیت
* یا دوست داری نسخه‌ی self-host شده‌ی همین API رو روی سرور خودت ران کنی

می‌تونی:

* Issue ثبت کنی
* پیشنهاد طراحی / PR بدی (اگر این ریپو پابلیک باشد)
* یا از این README به‌عنوان داکیومنت شروع برای پروژه‌ی خودت استفاده کنی.

---

## 📞 پشتیبانی / ارتباط

(این قسمت را با اطلاعات واقعی خودت جایگزین کن)

* وب‌سایت: `https://api.ipmyp.ir`
* ایمیل: `support@ipmyp.ir`
* کانال‌های دیگر (در صورت وجود): Telegram / Discord / …

---

با عشق به دنیای شبکه، Dev و زیرساخت 💚
`api.ipmyp.ir`

```
::contentReference[oaicite:0]{index=0}
```
