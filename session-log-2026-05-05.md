# Session Log — 2026-05-05

## סיכום Session זה

### נושאים שטופלו
1. תיקוני HMI (hmi2.html) — שינויים רבים
2. עיצוב לוגו — קבצי SVG מרובים
3. הגדרת גודל מסך ל-7 אינץ' 1024×600

---

## HMI Changes (hmi2.html)

### 1. הסרת `jamAttempts` מ-gSet
- הוסר לגמרי מ-Global Settings
- לא מופיע יותר בהגדרות

### 2. תיקון simTick — מונה חיתוכי להב
```javascript
// נוסף בתוך simTick():
fBladeCuts++;  // כל חיתוך של יחידה קדמית
rBladeCuts++;  // כל חיתוך של יחידה אחורית

// כל 25 ticks:
simTickCount++;
if (simTickCount % 25 === 0) renderBladeStatus();
```

### 3. מסך Maintenance — Sensor Health (עיצוב מחדש)
**לפני:** כל חיישן בתא נפרד, תפס הרבה מקום
**אחרי:** שורה אחת לכל יחידה עם מפריד `|`:
```
Front:  Home ● OK  |  End ● OK
Rear:   Home ● OK  |  End ● OK
```

### 4. מסך Maintenance — Filter Change Log
- נוסף card חדש: "Filter Change Log"
- מוגבל ל-2 entries
- כפתור "Replace Filter" מוסיף entry עם timestamp

### 5. End Work Modal — שם שדה
- "Rejects Weight" → **"Cut Pieces Weight"**
- כל הפונטים במודאל הוגדלו ~40%

### 6. Log Basket Modal — מה מוצג
- **לפני:** הציג `basketCuts` (סה"כ שתי יחידות)
- **אחרי:** מציג רק `fCuts` (יחידה קדמית בלבד)
- כותרת משנה: "Front unit · cut pieces weight"
- קוד: `document.getElementById('mb-cuts').textContent = fCuts;`

### 7. שינוי שם — "Cut Rate" → "Cut Cycle"
- בכל מקום שמופיע "Cut Rate" שונה ל-**"Cut Cycle"**
- קיצור: `c/min` (לא `cyc/min`)

### 8. Basket Weigh Reminder
**דרישה:** אנימציה על כפתור ה-Log Basket לאחר X דקות.

**יישום:**
- נוסף `basketRemind` ל-gSet: `{ val: 60, min: 5, max: 480, step: 5, unit: ' min' }`
- טריגר: `basketSec >= gSet.basketRemind.val * 60 && state === 'running'`
- הכפתור נושם (scale + deep-blue peak) — CSS animation `basket-breathe`
- פעמון 🔔 מופיע בתוך הכפתור ומצלצל בשיא ה-breathe — CSS animation `bell-ring`

```css
@keyframes basket-breathe {
  0%,100% { transform: scale(1); background: var(--cyan-dim); ... }
  50%      { transform: scale(1.028); background: rgba(10,70,180,0.52); ... }
}
@keyframes bell-ring {
  0%,38%,100% { transform: rotate(0deg) scale(1); }
  44%  { transform: rotate(-20deg) scale(1.15); }
  ...
}
.btn-basket-main.weigh-remind { animation: basket-breathe 2.6s ease-in-out infinite; }
.basket-bell { display: none; }
.btn-basket-main.weigh-remind .basket-bell { display: inline-block; animation: bell-ring 2.6s ...; }
```

- בסימולטור: כפתור "⚖ Weigh Remind" שמגדיר `basketSec = gSet.basketRemind.val * 60`

---

## Logo Files

### logo.svg (Primary)
- Badge עגול, r=56, center (76,80), fill כהה #07101C
- **גליל:** x=31–121 (90px = 80% רוחב badge), y=20–78
- **להב:** נכנס מימין, polygon: `134,71  62,71  55,76  63,82  134,82`
- **כדור:** cx=76, cy=110, r=24
- Wordmark: SNAP·CUT, Arial Black 900, 56px, dot בצבע #006FA0
- Tagline: "SMART CUT. CLEAN DROP."

### logo-dark.svg
- זהה ל-logo.svg אבל wordmark בלבן (#E4EEF8) על רקע כהה

### logo-ltr.svg
- מראה של logo.svg — להב נכנס משמאל
- Blade polygon מורה: `18,71  90,71  96,76  88,82  18,82`

### logo-b.svg (Sphere as Hero)
- גליל צר בלבד בחלק העליון (x=48–104, y=20–62)
- כדור גדול: cx=76, cy=99, r=34 — הכדור הוא הגיבור

### logo-green.svg (Recycling)
- כל מה שב-logo.svg אבל בגוון ירוק כהה
- **תיקון מרכזי:** 3 קשתות מרובות שלא היו מיושרות → קשת אחת חלקה:
```svg
<!-- קשת 270° בכיוון השעון מ-60° עד 330°, r=68 -->
<path d="M 110.0,138.9 A 68,68 0 1,1 134.9,46.0"
      stroke="#28A048" stroke-width="6.5" fill="none"/>
<!-- חץ בנקודת הסיום (330°) -->
<polygon points="134.9,46.0 136.0,33.8 123.8,40.8" fill="#28A048"/>
```
- Math: Start 60°=(76+68·cos60°, 80+68·sin60°)=(110,138.9), End 330°=(134.9,46)
- כיוון התנועה בנקודת הסיום: (0.5, 0.866), חץ מחושב בהתאם

### logo-pro.svg (Premium/World-Class)
- **Badge dome highlight:** radialGradient שקוף מאוד — ה-badge נראה כמו ספירה מלוטשת
- **גליל:** x=46–106 (60px = 53% רוחב badge)
- **כדור:** cx=76, cy=112, r=26 — התחתית נחתכת ב-2px ע"י ה-badge (תחושת "נופל")
- **Seam line:** קו ציאן מ-x=52 עד x=106 ב-y=74 — מראה היכן הלהב כבר עבר
- **Inner depth ring:** `<circle r="54.4" stroke="rgba(255,255,255,0.045)"/>` — עומק כמו זכוכית שעון
- ספרייה gradient: 5 stops לכדור, צבעים עמוקים יותר

---

## Screen Sizing — 7" 1024×600

### האתגר
דפדפנים לא חושפים DPI פיזי אמיתי. `matchMedia(min-resolution)` מחזיר 96×DPR (CSS reference), לא DPI פיזי. CSS `cm` units ≠ סמ פיזיים על מסכי desktop.

### הפתרון הסופי
```css
html, body { width: 100%; overflow-x: hidden; }

.hmi-wrapper {
  width:  calc(1024px * var(--hmi-scale, 1));
  height: calc(600px  * var(--hmi-scale, 1));
  overflow: hidden;
}

.hmi-frame {
  width: 1024px; height: 600px;
  transform-origin: top left;
  transform: scale(var(--hmi-scale, 1));
}

.demo-panel {
  width: calc(1024px * var(--hmi-scale, 1));
  margin-top: 10px;
}
```

```javascript
(function fitHMI() {
  const LS_KEY = 'hmi-dev-scale';

  function savedScale() {
    const s = parseFloat(localStorage.getItem(LS_KEY));
    if (!s) return Math.min(1.0, window.innerWidth/1024, window.innerHeight/600);
    return s;
  }

  window.hmiScaleStep = function(delta) {
    const next = clamp(currentScale + delta);
    localStorage.setItem(LS_KEY, next);
    applyScale(next);
  };

  applyScale(savedScale());
})();
```

**על המכשיר הפיזי (1024×600):** scale=1.0 → בדיוק 15.5 סמ
**בפיתוח:** localStorage שומר הגדרה ידנית. כפתורי `−` / `%` / `+` בסימולטור (±5%)

### מבנה HTML
```html
<body>
  <div class="hmi-wrapper">       ← מתכווץ לגודל הוויזואלי האמיתי
    <div class="hmi-frame">       ← תמיד 1024×600, transform scale
      ...
    </div>
  </div>
  <div class="demo-panel">        ← מחוץ ל-frame, תמיד גלוי
    ...
  </div>
</body>
```

---

## Git Commits (Session זה)
```
b9ee162  Fix physical size via DPI detection; restore simulator bar
b70db26  Cap HMI scale at 1.0 — never scale up beyond native resolution
fbc06cc  Scale HMI to fit any viewport; pixel-perfect on 1024×600
72e99e3  Fix viewport for 7" 1024×600 touchscreen
63565a9  Add pro mark logo variant; fix green recycling ring
886a6a7  (earlier commits...)
```

---

## דברים שנדונו אבל לא יושמו
- **CSS `cm` כגודל container:** נוסה, לא עובד על מסכי desktop (CSS cm ≠ physical cm)
- **matchMedia DPI detection:** לא עובד כי מחזיר 96×DPR, לא DPI פיזי
- **Auto-calibration ruler widget:** נדון, לא יושם — הוחלט על localStorage + כפתורים

---

## TODO / שינויים עתידיים (מה שהמשתמש ציין)
- שינויים נוספים של ממשק — פרטים לא פורטו, יגיעו ב-session הבא

---

## איך לפתוח session חדש
1. פתח Claude Code
2. Claude יקרא `CLAUDE.md` אוטומטית
3. כל ההקשר הבסיסי זמין
4. אם צריך פרטים ספציפיים מה-session הזה — הפנה ל-`session-log-2026-05-05.md`
