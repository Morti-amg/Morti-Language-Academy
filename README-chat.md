# راه‌اندازی گفتگوی زنده (Firebase، رایگان)

سایت بدون این مرحله هم کار می‌کند، ولی گفتگو فقط بین تب‌های همان مرورگر است (حالت آزمایشی).
برای گفتگوی واقعی بین همهٔ کاربران:

1. به https://console.firebase.google.com بروید و با حساب گوگل وارد شوید، **Add project** را بزنید.
2. **Build ← Authentication ← Get started**: روش **Anonymous** را روشن کنید (برای کاربران).
   سپس روش **Email/Password** را هم روشن کنید (برای ورود شما به‌عنوان مدیر).
3. **Authentication ← Users ← Add user**: ایمیل و رمز خودتان را بسازید. **User UID** را کپی کنید.
4. **Build ← Realtime Database ← Create database** (حالت locked).
5. در تب **Rules** این متن را بچسبانید و **Publish** کنید.
6. در تب **Data** روی علامت + کنار نام پایگاه‌داده بزنید: نام `admins`، و داخلش نام = UID مدیر، مقدار = `true` (boolean).
7. **Project settings ← General ← Your apps ← Web (</>)** را بزنید، برنامه را ثبت کنید و `firebaseConfig` را کپی کنید.
8. در `index.html` خط `const FIREBASE_CONFIG=null;` را پیدا کنید و `null` را با آن متن (به شکل `{apiKey:"...", ...}`) عوض کنید.
9. **Authentication ← Settings ← Authorized domains**: آدرس `morti-amg.github.io` را اضافه کنید.
10. فایل را در GitHub دوباره آپلود کنید. در تب «مدیر» با همان ایمیل و رمز وارد شوید.

## Rules
```json
{
  "rules": {
    "rooms": { "main": {
      ".read": "auth != null",
      "$m": {
        ".write": "auth != null && (root.child('admins/'+auth.uid).val() === true || (!data.exists() && newData.child('uid').val() === auth.uid && !root.child('muted/'+auth.uid).exists()))",
        ".validate": "newData.hasChildren(['uid','name','text','ts'])"
      } } },
    "online": { ".read": "auth != null", "$u": { ".write": "auth != null && auth.uid === $u" } },
    "muted": { ".read": "auth != null", ".write": "auth != null && root.child('admins/'+auth.uid).val() === true" },
    "admins": { "$u": { ".read": "auth != null && auth.uid === $u" } },
    "dm": { "$u": {
      ".read": "auth != null && (auth.uid === $u || root.child('admins/'+auth.uid).val() === true)",
      ".write": "auth != null && (auth.uid === $u || root.child('admins/'+auth.uid).val() === true)" } },
    "dmidx": { ".read": "auth != null && root.child('admins/'+auth.uid).val() === true",
      "$u": { ".write": "auth != null && (auth.uid === $u || root.child('admins/'+auth.uid).val() === true)" } },
    "reports": { ".read": "auth != null && root.child('admins/'+auth.uid).val() === true",
      "$r": { ".write": "auth != null && (!data.exists() || root.child('admins/'+auth.uid).val() === true)" } }
  }
}
```

## قوانین جامعه (در سایت به چهار زبان نمایش داده می‌شود)
1. با همه مؤدب و مهربان باش؛ هر کسی در حال یادگیری است.
2. اشتباه کردن طبیعی است؛ اشتباه دیگران را با مهربانی اصلاح کن.
3. توهین، تمسخر، نفرت‌پراکنی و فحش ممنوع است.
4. اطلاعات شخصی و لینک نفرست.
5. اسپم و تبلیغ ممنوع است.
6. مدیر می‌تواند پیام‌ها را اصلاح یا حذف کند و کاربر خاطی را بی‌صدا کند.
