# clean-code-skill

[English](README.md) | **العربية**

مهارة (Skill) للـ AI Agent بتخليه يراجع الكود ويعمله Refactor على أساس **11 قاعدة Clean Code**، بخطوات آمنة ومتدرجة و**من غير ما يغيّر سلوك الكود**.

بتشتغل مع [OpenCode](https://opencode.ai) و Claude Code، ومع أي Agent بيقرأ صيغة `SKILL.md`.

## ليه الـ Skill دي؟

لو ادّيت الـ Agent قائمة قواعد بس، هيطلع تعديلات عامة وغير متسقة. الـ Skill دي بتديله:

- **Workflow واضح:** تحديد النطاق، قياس الوضع الحالي، فحص، تقرير، تنفيذ بترتيب آمن، تحقق، ملخص.
- **وضعين:** مراجعة فقط (تقرير)، أو Refactor (تنفيذ التعديلات).
- **حدود أمان:** مش هيغيّر أسماء الـ routes أو أعمدة الداتابيز أو الـ APIs العامة، ومش هيلمس نصوص الواجهة أو الترجمات، ومش هيضيف مكتبات من غير ما يسألك.
- **وعي بتغيّر السلوك:** أي تعديل بيغيّر السلوك (زي إضافة معالجة أخطاء) بيتكتب في قسم منفصل عشان توافق عليه.
- **أنماط عملية:** لكل قاعدة: العَرَض (Smell)، والحل، ومثال قبل/بعد، مع مثال Flask كامل.
- **أمانة:** لو الكود نضيف فعلاً، هيقولك كده بدل ما يخترع ملاحظات.

## القواعد الـ 11

| # | القاعدة | باختصار |
|---|---------|---------|
| 1 | Naming (التسمية) | أسماء واضحة ومباشرة بتقول الحاجة دي إيه أو بتعمل إيه |
| 2 | Comment (التعليقات) | اشرح **ليه**، مش اللي الكود بيقوله أصلاً |
| 3 | Consistency (الاتساق) | أسلوب واحد ثابت في المشروع كله |
| 4 | Early Return (الخروج المبكر) | اخرج بدري بدل الشروط المتداخلة |
| 5 | Function (الدوال) | كل دالة مسؤولة عن مهمة واحدة |
| 6 | Single Responsibility (المسؤولية الواحدة) | كل جزء في الكود له مسؤولية واحدة واضحة |
| 7 | Abstraction (التجريد) | خبّي التفاصيل المشوشة من غير مبالغة |
| 8 | Separation of Concerns (فصل المسؤوليات) | افصل الواجهة عن الـ API عن الـ Business Logic |
| 9 | DRY (عدم تكرار الكود) | الكود المكرر يتجمع في مكان واحد |
| 10 | Don't Repeat Logic (عدم تكرار المنطق) | كل قاعدة عمل (Business Rule) تتكتب في مكان واحد |
| 11 | Error Handling (التعامل مع الأخطاء) | متفترضش إن كل حاجة هتنجح، وعالج الفشل صراحةً |

القواعد متقسمة على أربع مجموعات (قراءة الكود، البنية، التكرار، معالجة الأخطاء) عشان الـ Agent يحمّل الإرشادات اللي محتاجها بس.

## التثبيت

الـ Skill هي فولدر `clean-code/`. انسخه لمكان الـ skills بتاع الـ Agent.

**OpenCode، عام (لكل المشاريع)**
```bash
git clone https://github.com/<your-username>/clean-code-skill.git
mkdir -p ~/.config/opencode/skills
cp -r clean-code-skill/clean-code ~/.config/opencode/skills/
```

**ويندوز (PowerShell)**
```powershell
git clone https://github.com/<your-username>/clean-code-skill.git
New-Item -ItemType Directory -Force "$HOME\.config\opencode\skills"
Copy-Item -Recurse clean-code-skill\clean-code "$HOME\.config\opencode\skills\"
```

**لمشروع واحد:** انسخ `clean-code/` جوه `.opencode/skills/` في المشروع.

**Claude Code:** انسخه في `~/.claude/skills/` (عام) أو `.claude/skills/` (للمشروع).

أعد تشغيل الـ Agent بعد التثبيت.

## الاستخدام

أضمن طريقة إنك تذكر اسم الـ Skill صراحةً:

```
Use the clean-code skill to review src/ (report only)
Use the clean-code skill to refactor routes.py
Apply only Early Return and Naming from the clean-code skill on services/
```

وبتتفعّل لوحدها كمان لما تطلب تنضيف أو Refactor أو مراجعة أو تحسين جودة الكود.

وبتشتغل بالعربي:

```
راجع الكود ده بـ clean-code skill وطلعلي تقرير بس
نضّف ملف routes.py وطبّق Early Return وفصل المسؤوليات
```

## الهيكل

```
clean-code-skill/
├── README.md / README.ar.md
├── LICENSE
├── clean-code/                  <- الـ Skill (انسخ الفولدر ده)
│   ├── SKILL.md                 الـ workflow وجدول القواعد وحدود الأمان وشكل التقرير
│   └── references/
│       ├── readability.md       Naming, Comment, Consistency, Early Return
│       ├── structure.md         Function, Single Responsibility, Abstraction, Separation of Concerns
│       ├── duplication.md       DRY, Don't Repeat Logic
│       └── error-handling.md    Error Handling
└── examples/
    └── flask-refactor.md        مثال قبل/بعد كامل
```

ملف `SKILL.md` قصير ويتحمّل كل ما الـ Skill تتفعّل. ملفات `references/` بتتقرأ وقت الحاجة بس، وده بيحافظ على حجم الـ context صغير.

## نصايح

- جرّبها على مشروع صغير الأول، وبعدين على مشروع أكبر.
- المشاريع اللي فيها Tests بتديك أحسن نتيجة، لأن الـ Agent بيتحقق بعد كل خطوة.
- اعمل commit قبل ما تبدأ، عشان تقدر ترجع عن أي خطوة.
- قول للـ Agent أنهي قواعد أهم عندك، أو أنهي فولدرات يتجاهلها.

## المساهمة

الـ Issues والـ Pull Requests مرحّب بيها: أمثلة قبل/بعد للغات وفريمورك تانية (JavaScript/TypeScript، C#، Java، Go...)، تحسين اكتشاف المشاكل، والترجمات.

## الرخصة

MIT. شوف [LICENSE](LICENSE).
