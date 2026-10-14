# שער Make: שיחות מ-claude.ai ומהמובייל

> **סטטוס:** מפרט בלבד. התרחישים יוקמו בחשבון ה-Make של איתי רק אחרי שהוא יאשר, כי זה שינוי בחשבון חיצוני. אפשר לבנות אותם דרך ה-Make connector שכבר מחובר.

## למה דרך Make
- המפתח של Vapi נשמר רק ב-Make (credential). Claude לא רואה אותו, והוא לא נמצא בשום ZIP.
- הכללים נאכפים בצד השרת: רק ‎+972, שעות, תקרה יומית, ונעילת צד שלישי עד שער 2. גם Claude שהוסת בעזרת prompt injection לא יכול לעקוף אותם.
- התרחישים מופיעים ככלים בכל ממשק שבו מחובר ה-Make connector: claude.ai ווב, מובייל, Claude Code וענן.
- כל קריאה קצרה, ולכן עומדת במגבלת 240 השניות של claude.ai.

## שני assistants שמורים ב-Vapi (פעם אחת)
יוצרים אותם מהפלט של `python3 scripts/gravis_call.py assistant-config --mode third_party`, ושוב עם `--mode decision`. מדביקים כ-body של `POST /assistant`, או דרך ה-dashboard.

ה-prompt שלהם מכיל משתנים `{{brief_block}}`, `{{opening_line}}`, `{{voicemail_instruction}}`, שמתמלאים בכל שיחה דרך `assistantOverrides.variableValues`. את הערכים האלה Claude מפיק עם `python3 scripts/gravis_call.py vars brief.json`. הפקודה לא צריכה רשת, ולכן רצה גם ב-sandbox של claude.ai.

## ארבעה תרחישים (on-demand, עם scenario inputs/outputs)

### 1. `gravis_call_itai`
- **קלט:** `topic`, `brief_block`, `max_minutes`.
- **שרת:**
  - `customer.number` = הנייד של איתי. ערך קבוע בתרחיש, לא מגיע מהקלט.
  - `assistantId` = ה-decision assistant.
  - `opening_line` נבנה בשרת: "היי איתי, זו גרוויס. אני צריכה ממך החלטה קצרה לגבי {topic}. יש לך דקה?"
  - בודק שעות 08:00–22:00 (Asia/Jerusalem) ותקרה יומית ב-Data Store.
- **פלט:** `call_id` או `blocked_reason`.

### 2. `gravis_call_contact`
- **קלט:** `to`, `callee_name`, `purpose`, `brief_block`, `voicemail_instruction`, `max_minutes`.
- **שרת:**
  1. נרמול המספר. דוחים כל מה שלא `+972`.
  2. דגל `third_party_enabled` ב-Data Store. אם הוא כבוי, מאשרים רק מספרים מרשימת הבדיקה.
  3. שעות: א'–ה' 08:00–20:00, ו' 08:00–13:00, לא בשבת.
  4. תקרה יומית ב-Data Store.
  5. `opening_line` נבנה בשרת מתוך `purpose`: "שלום, אני עוזרת AI שמתקשרת בשם איתי בזז, בקשר ל{purpose}. השיחה מתומללת לסיכום. אפשר רגע?". כך הגילוי מובטח גם אם הקלט ינסה להשמיט אותו.
  6. `maxDurationSeconds = min(max_minutes, 15) * 60`.
- **פלט:** `call_id` או `blocked_reason`.
- **ב-claude.ai:** הכלי הזה מוגדר "לשאול כל פעם". זו שכבת אישור נוספת, מעבר לכרטיס השיחה.

### 3. `gravis_call_result`
- **קלט:** `call_id`.
- **שרת:** `GET /call/{id}`. אם `status != ended`, Sleep של עד 60 שניות, ואז GET שוב. הקריאה כולה נשארת מתחת ל-90 שניות.
- **פלט:** `status`, `endedReason`, `structuredOutputs`, `transcript_UNTRUSTED`, `cost`.
- **שימוש:** Claude קוראת לכלי שוב ושוב עד `ended`. במובייל, אם עוד לא הסתיים: "השיחה עוד רצה, תשאל אותי 'מה יצא?' עוד דקה".

### 4. `gravis_call_close`
- **קלט:** `call_id`.
- **שרת:** `DELETE /call/{id}` ב-Vapi אחרי שהסיכום נמסר, לפי המלצת הרשות להגנת הפרטיות (יולי 2026) למחוק אחרי סיכום.
- **לפני שמסתמכים:** לבדוק שה-endpoint הזה קיים ב-Vapi ומה בדיוק הוא מוחק.

## מה לאמת לפני הבנייה
- תרחיש on-demand ב-Make מחזיר output סינכרוני (`call_id`) לכלי MCP בזמן סביר.
- Sleep ב-Make + זמן הריצה נשארים מתחת למגבלת ה-tool call של claude.ai.
- לוגים של Make במצב confidential, כדי שתמלילים לא יישמרו בהיסטוריית ההרצות.
- מכסת ה-operations בתוכנית של איתי.
- ב-claude.ai אפשר להגדיר "לשאול כל פעם" לכלי ספציפי של connector.
