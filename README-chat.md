# Live chat – how it works (no account, no server)

The chat connects every visitor through a free public MQTT broker (HiveMQ, backup: EMQX).
Nothing to set up: upload `index.html` + `data.json` and the chat is live for everyone.

* Everyone online sees the online list, people joining ("👋 Say hi"), the typing indicator and new messages instantly.
* Private messages to the admin are end-to-end encrypted (AES-GCM / ECDH). Only the user and the admin can read them.
* Admin actions (correct, pin, delete, mute, 👑 badge) are digitally signed. Nobody else can fake them.

## Admin login
1. Keep `admin-key.json` SECRET. **Never upload it to GitHub.** Store it on your phone/PC.
2. Open the site → 🔒 → choose `admin-key.json` (or paste its content) → Login. The key is remembered on that device.
3. Lost key = no admin access. Keep a backup copy.

## Honest limits
* The group chat is public. Messages are kept for 3 days.
* It uses a shared public broker: if it is down the chat shows "connecting…". No guarantee of service.
* Spam: 1 message per 1.2 s per client; the admin can mute users.

# Optional: Firebase instead (stricter, real accounts)
Paste your `firebaseConfig` into `FIREBASE_CONFIG` in index.html. Then Firebase is used instead of the public broker. Steps and rules:

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

---

## Konten, Admin-Seite, Moderator (neu)

**Ablauf für Benutzer:** Die Seite zeigt zuerst nur Anmelden/Registrieren. Benutzername + eigenes Passwort (mind. 10 Zeichen, Buchstaben und Zahl) → danach Zugang. Optionale Angaben (E-Mail, Land, Lernziel …) werden **nur für den Admin verschlüsselt** gesendet; nur du kannst sie in der Admin-Konsole lesen.

**Admin-Konto einrichten (einmalig):** Auf dem Anmeldebildschirm „🛠 Admin-Konto einrichten“ → Inhalt von `admin-key.json` einfügen + Benutzername und Passwort wählen. Danach meldest du dich auf jedem Gerät mit Name + Passwort an. `admin-key.json` NIEMALS auf GitHub hochladen – nur `index.html` (und data.json) gehören ins Repo.

**Admin-Konsole (🛠 oben):** Dashboard, Benutzer (Info, sperren, freigeben, Moderator ernennen), Registrierungsanfragen, Chat/Meldungen, Konferenzen, Seiteneinstellungen (Registrierung offen / mit Freigabe / geschlossen, Chat, Privatchat, Konferenzen, Anrufe, Links, Nachrichten-Abstand, Teilnehmerlimits, Hinweis-Banner), Inhalte (Wörter/Sätze), Mein Konto.

**Moderator:** Wenn du nicht da bist, ernennst du einen Benutzer zum Moderator. Er kann im Chat/in Konferenzen moderieren (löschen, stummschalten, bannen), sieht aber weder Registrierungsdaten noch Admin-Nachrichten und kann keine Einstellungen ändern.

**Ehrliche Grenzen (statische Seite ohne eigenen Server):**
- Alles läuft über öffentliche MQTT-Broker; es gibt keine Garantie für Verfügbarkeit.
- Die Zugangsdaten liegen als mit dem Passwort verschlüsselte Schlüssel im Broker. Schwache Passwörter können offline erraten werden – darum Passwortregeln.
- Wer einen freien Benutzernamen zuerst registriert, behält ihn (Admin-Name ist geschützt). Empfehlung: Registrierung auf „mit Freigabe“ stellen.
- Gefälschte Einträge (Rollen, Sperren, Einstellungen, Nachrichten) werden durch Signaturen abgewiesen; gelöschte/überschriebene Einträge stellen die Clients automatisch wieder her.
- Lernfortschritt bleibt pro Gerät. Übersetzungen sind nicht von Muttersprachlern geprüft.
