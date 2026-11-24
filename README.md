# דף נחיתה – מרפאת שיניים

דף נחיתה בעמוד אחד (one-pager), RTL בעברית, מותאם למובייל, לקמפיינים בגוגל, בפייסבוק ובאינסטגרם.
קובץ אחד (`index.html`), בלי build ובלי תלויות. אפשר להעלות אותו כמו שהוא ל-GitHub Pages, Netlify או Vercel.

## מה יש בדף
- Hero עם כותרת, הצעה וטופס לידים
- פס נתונים, טיפולים, "למה אנחנו", מקום לביקורות גוגל, שאלות נפוצות, CTA סופי, פוטר
- סרגל דביק במובייל עם כפתורי "התקשרו" ו"וואטסאפ"
- תמיכה במצב כהה (Dark mode)

## הגדרה לפני עלייה לאוויר
כל ההגדרות נמצאות בבלוק `window.SITE_CONFIG` בראש `index.html`:

| שדה | מה לשים | איפה מוצאים |
|---|---|---|
| `META_PIXEL_ID` | מזהה הפיקסל (מספר) | Meta Events Manager → Data Sources |
| `GA4_ID` | `G-XXXXXXX` | GA4 → Admin → Data streams |
| `GOOGLE_ADS_ID` | `AW-XXXXXXXXX` | Google Ads → Goals → Conversions → Tag setup |
| `GOOGLE_ADS_LEAD_LABEL` | תווית ההמרה של הליד | אותו מסך, החלק שאחרי `/` ב-`send_to` |
| `GOOGLE_ADS_CALL_LABEL` | תווית המרה לקליק על טלפון (לא חובה) | המרה נפרדת מסוג "Click to call" באתר |
| `FORM_ENDPOINT` | כתובת Webhook שמקבלת JSON | Make / Zapier / Formspree / Google Apps Script |
| `PHONE` / `WHATSAPP` | טלפון המרפאה / מספר וואטסאפ בפורמט `9725XXXXXXXX` | – |

שדה שנשאר ריק פשוט לא נטען, כך שאפשר לעבוד גם עם חלק מהפיקסלים בלבד.

## אילו אירועים נשלחים
| פעולה | Meta | GA4 | Google Ads |
|---|---|---|---|
| טעינת עמוד | `PageView` | page_view (אוטומטי) | – |
| שליחת טופס תקינה | `Lead` | `generate_lead` | conversion → `LEAD_LABEL` |
| קליק על טלפון | `Contact` | `click_call` | conversion → `CALL_LABEL` |
| קליק על וואטסאפ | `Contact` | `click_whatsapp` | – |

הטופס שומר אוטומטית בשדות נסתרים את `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term`, `gclid` ו-`fbclid`, וגם את כתובת העמוד ושעת השליחה. ככה רואים בגיליון מאיזה קמפיין הגיע כל ליד.

**מומלץ:** ב-GA4 לסמן את `generate_lead` כ-Key event, וב-Meta להגדיר את `Lead` כאירוע האופטימיזציה של הקמפיין.

## חיבור הטופס לגוגל שיטס (בחינם)
1. פותחים Google Sheet חדש, Extensions → Apps Script, ומדביקים:
```js
function doPost(e) {
  const d = JSON.parse(e.postData.contents);
  SpreadsheetApp.getActiveSheet().appendRow([new Date(), d.name, d.phone, d.treatment,
    d.utm_source, d.utm_campaign, d.gclid, d.fbclid, d.page]);
  return ContentService.createTextOutput('ok');
}
```
2. Deploy → New deployment → Web app → Who has access: **Anyone**.
3. מדביקים את הכתובת ב-`FORM_ENDPOINT`.

> Apps Script לא מחזיר כותרות CORS. אם הדפדפן חוסם את הבקשה, עדיף להשתמש ב-Make, Zapier או Formspree, ומשם לשלוח לשיטס, למייל או לוואטסאפ של המרפאה.

## העלאה ל-GitHub Pages
```bash
git init
git add .
git commit -m "Dental landing page"
git branch -M main
git remote add origin https://github.com/<USER>/<REPO>.git
git push -u origin main
```
אחר כך: Settings → Pages → Source: `main` / root. לדומיין של הלקוח מוסיפים קובץ `CNAME` עם הדומיין ומגדירים DNS.

## בדיקה אחרי עלייה
- **Meta Pixel Helper** (תוסף לכרום) – לוודא ש-PageView ו-Lead נשלחים
- **Google Tag Assistant** – לוודא שה-gtag וההמרות נורים
- Meta Events Manager → Test events
- שליחת ליד ניסיון ובדיקה שהוא הגיע ל-Webhook

## לפני שמעלים ללקוח אמיתי
- להחליף את שם המרפאה, הכתובת, הטלפון, השעות והנתונים (כרגע הם לדוגמה)
- להכניס ביקורות גוגל אמיתיות בלבד
- להוסיף קישור למדיניות פרטיות ובאנר עוגיות (תיקון 13 לחוק הגנת הפרטיות)
- לבדוק שהניסוח עומד בכללי הפרסום של Meta לתחום הבריאות ובתקנות הפרסום של רופאי שיניים
