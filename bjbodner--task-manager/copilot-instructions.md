## task-manager

> אפליקציית Kanban board אישית בעברית (RTL) עם גיימיפיקציה.

# TaskFlow - מערכת ניהול משימות אישית

## סקירה
אפליקציית Kanban board אישית בעברית (RTL) עם גיימיפיקציה.
משתמש יחיד, ללא auth, רצה מקומית בדפדפן.

## סטאק טכנולוגי
- **Frontend:** React 18 + Vite, Tailwind CSS
- **Backend:** Supabase (PostgreSQL)
- **ספריות:** @hello-pangea/dnd (drag & drop), canvas-confetti, Recharts
- **הרצה:** `npm run dev`

## מבנה פרויקט
```
src/
  components/
    Board/          # לוח קנבן ראשי
    TaskCard/       # כרטיס משימה
    Header/         # XP bar, רמה, סטריק
    Stats/          # דשבורד סטטיסטיקות
    AddTask/        # טופס הוספה מהירה
    Celebration/    # קונפטי ואנימציות
  hooks/            # Custom React hooks
  lib/              # Supabase client, utils
  styles/           # גלובלי, תמה
```

## מודל נתונים (Supabase)

### tasks
| שדה | סוג | ברירת מחדל |
|-----|------|------------|
| id | UUID | auto |
| title | TEXT | - |
| status | ENUM: todo/in_progress/needs_approval/done | todo |
| category | ENUM: work/home/relationships/learning | work |
| priority | ENUM: high/medium/low | medium |
| created_at | TIMESTAMPTZ | now() |
| updated_at | TIMESTAMPTZ | now() |
| completed_at | TIMESTAMPTZ | NULL |
| xp_earned | INTEGER | 0 |

### user_stats (שורה יחידה)
| שדה | סוג | ברירת מחדל |
|-----|------|------------|
| total_xp | INTEGER | 0 |
| level | INTEGER | 1 |
| current_streak | INTEGER | 0 |
| longest_streak | INTEGER | 0 |
| last_completed_date | DATE | NULL |
| tasks_completed_total | INTEGER | 0 |

## 4 עמודות קנבן
1. **עדיין לא התחיל** (todo) - אפור, ברירת מחדל למשימות חדשות
2. **בביצוע** (in_progress) - כחול
3. **צריך אישור** (needs_approval) - כתום
4. **הושלם** (done) - ירוק, מפעיל XP + אנימציה

## קטגוריות
- עבודה (work) - כחול
- בית (home) - ירוק
- מערכות יחסים (relationships) - ורוד
- לימודים (learning) - סגול

## גיימיפיקציה
- **XP:** נמוכה +10, בינונית +20, גבוהה +30, בונוס סטריק +5
- **רמות:** כל רמה דורשת רמה × 100 XP (רמה 2 = 200, רמה 3 = 300...)
- **סטריקים:** ספירת ימים רצופים עם השלמת משימה. נשבר ביום ללא השלמה
- **אבני דרך:** 7, 30, 100 ימים
- **אנימציות:** קונפטי בהשלמה, אנימציה מיוחדת בעליית רמה

## עיצוב
- צבעוני ושובב, פינות מעוגלות
- עברית בלבד, RTL מלא
- Desktop-first, רספונסיבי
- Header: שם אפליקציה, XP bar, רמה, סטריק (אייקון אש), כפתור סטטיסטיקות

## פעולות משתמש
- הוספה מהירה (כותרת + קטגוריה + עדיפות)
- גרירה בין עמודות
- עריכת משימה (כותרת/קטגוריה/עדיפות)
- מחיקה עם אישור
- דשבורד: גרפים שבועי/חודשי, עוגה לפי קטגוריות, היסטוריית סטריקים

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BjBodner) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
