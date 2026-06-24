# gpt-image-gen — Skill ליצירת תמונות עם OpenAI

סקיל זה שולח prompt ל-OpenAI Images API ומחזיר תמונה PNG שמורה בנתיב שצוין.

## מודל

`gpt-image-2` — מודל יצירת תמונות של OpenAI (שוחרר אפריל 2026).
**אל תשנה את שם המודל.** אם יש שגיאה, הבעיה היא ב-API key או בפרמטרים — לא בשם המודל.

## דרישות סביבה

`OPENAI_API_KEY` — מוגדר בקובץ `.env` בשורש הפרויקט.

```bash
source .env 2>/dev/null || true
export $(grep -v '^#' .env | xargs) 2>/dev/null || true
```

## קריאת ה-API

### שיטה ראשית — curl + jq

```bash
curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<THE_PROMPT>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' | jq -r '.data[0].b64_json' | base64 --decode > <OUTPUT_PATH>.png
```

### שיטה חלופית — curl + Python (כאשר jq לא מותקן)

```bash
RESPONSE=$(curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<THE_PROMPT>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }')

echo "$RESPONSE" | python3 -c "
import sys, json, base64
data = json.load(sys.stdin)
b64 = data['data'][0]['b64_json']
with open('<OUTPUT_PATH>.png', 'wb') as f:
    f.write(base64.b64decode(b64))
"
```

## זרימת עבודה מלאה לסוכן

```bash
# 1. טעינת משתני סביבה
export $(grep -v '^#' .env | grep -v '^$' | xargs)

# 2. הגדרת משתנים
PROMPT="<THE_PROMPT>"
OUTPUT_PATH="yuval/outputs/<YYYY-MM-DD>-<slug>.png"

# 3. קריאה ראשית עם jq
if command -v jq &> /dev/null; then
  curl -s -X POST "https://api.openai.com/v1/images/generations" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -H "Content-Type: application/json" \
    -d "{\"model\": \"gpt-image-2\", \"prompt\": \"$PROMPT\", \"size\": \"1024x1024\", \"quality\": \"medium\", \"output_format\": \"png\"}" \
    | jq -r '.data[0].b64_json' | base64 --decode > "$OUTPUT_PATH"
else
  # fallback עם Python
  RESPONSE=$(curl -s -X POST "https://api.openai.com/v1/images/generations" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -H "Content-Type: application/json" \
    -d "{\"model\": \"gpt-image-2\", \"prompt\": \"$PROMPT\", \"size\": \"1024x1024\", \"quality\": \"medium\", \"output_format\": \"png\"}")
  echo "$RESPONSE" | python3 -c "
import sys, json, base64
data = json.load(sys.stdin)
b64 = data['data'][0]['b64_json']
with open('$OUTPUT_PATH', 'wb') as f:
    f.write(base64.b64decode(b64))
"
fi

# 4. אימות
if [ -s "$OUTPUT_PATH" ]; then
  echo "SUCCESS: Image saved to $OUTPUT_PATH"
else
  echo "ERROR: Image file is empty or missing"
  exit 1
fi
```

## טיפול בשגיאות

אם ה-API מחזיר שגיאה, בדוק:
1. `OPENAI_API_KEY` מוגדר ותקין
2. ה-prompt לא מפר את מדיניות התוכן של OpenAI
3. חיבור אינטרנט תקין

**אל תשנה את שם המודל** — `gpt-image-2` הוא המודל הנכון.
