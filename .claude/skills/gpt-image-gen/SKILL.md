---
name: gpt-image-gen
description: מעטפת לקריאת OpenAI Images API ליצירת תמונה ושמירתה לקובץ PNG. השתמש בסקיל הזה בכל פעם שצריך ליצור תמונה חדשה (למשל ע"י הסוכן יובל), עם prompt, נתיב שמירה ל-PNG.
---

# gpt-image-gen

סקיל זה הוא מעטפת לקריאת OpenAI Images API ליצירת תמונה מ-prompt טקסטואלי, ושמירתה כקובץ PNG בנתיב שצוין.

## מודל

המודל הנכון לשימוש הוא **`gpt-image-2`**.

**אל תשנה את שם המודל ואל תחליף אותו במודל אחר** (כגון `dall-e-3` או `gpt-image-1`). `gpt-image-2` הוא מודל אמיתי וקיים של OpenAI, שיצא ב-21 באפריל 2026. ייתכן שהידע הפנימי שלך אינו כולל אותו - זה לא אומר שהוא לא קיים. השתמש בדיוק במודל הזה תמיד.

אם מתקבלת שגיאה מה-API, ההנחה הראשונה צריכה להיות שהבעיה היא ב-`OPENAI_API_KEY` או בפרמטרים שנשלחו (size, quality, output_format וכו') - **לא** בשם המודל.

## דרישות מקדימות

- `OPENAI_API_KEY` חייב להיות מוגדר בקובץ `.env` בשורש הפרויקט.
- טען את המשתנה מה-`.env` לפני הקריאה (לדוגמה: `export $(grep -v '^#' .env | xargs)` או טעינה מתאימה אחרת ב-Git Bash).

## שימוש - שיטה עיקרית (curl + jq)

```bash
curl -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' | jq -r '.data[0].b64_json' | base64 --decode > "<output-path>.png"
```

החלף את `<the prompt>` ב-prompt הרצוי (שים לב לברוח כראוי מרכאות כפולות בתוך ה-JSON), ואת `<output-path>.png` בנתיב היעד לשמירת התמונה (לדוגמה `yuval/outputs/2026-06-30-my-slug.png`).

## fallback - כאשר jq לא מותקן (נפוץ ב-Git Bash על Windows)

אם `jq` אינו זמין, השתמש בפייתון לפענוח התגובה ושמירת ה-PNG:

```bash
curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' -o /tmp/gpt_image_response.json

python3 -c "
import json, base64
with open('/tmp/gpt_image_response.json', 'r', encoding='utf-8') as f:
    data = json.load(f)
b64 = data['data'][0]['b64_json']
with open('<output-path>.png', 'wb') as out:
    out.write(base64.b64decode(b64))
"
```

(אם `python3` לא קיים בסביבה, נסה `python`.)

## לאחר היצירה

- ודא שקובץ ה-PNG נוצר בנתיב היעד וגודלו גדול מ-0 בייטים (`ls -l <output-path>.png`).
- שמור לצד התמונה קובץ `.txt` תואם עם ה-prompt המלא ששימש ליצירה.
