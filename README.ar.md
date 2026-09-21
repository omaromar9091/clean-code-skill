# clean-code-skill

[English](README.md) | العربية

Skill لـ OpenCode و Claude Code بتراجع الكود وتعمله Refactor على أساس 11 قاعدة Clean Code. بتشتغل بخطوات صغيرة، وسلوك الكود بيفضل زي ما هو.

## بتعمل إيه

وجّه الـ Agent لملف أو فولدر. بيقرأ الكود، ويكتب تقرير بالمشاكل اللي لقاها مرتبة حسب الخطورة، ولو طلبت منه ينفّذ، بيطبّق الحلول قاعدة قاعدة، وبيشغّل التيستات أو التطبيق بعد كل خطوة.

فيه وضعين: مراجعة بس (تقرير من غير تعديل)، أو Refactor (بيعدّل). في الحالتين بيسيب في حاله الحاجات اللي كود تاني معتمد عليها: روابط الـ routes وأعمدة الداتابيز والـ APIs العامة ونصوص الواجهة والترجمات. ومبيضيفش مكتبات من غير ما يسألك. أي تعديل بيغيّر السلوك، زي إضافة معالجة أخطاء جديدة، بيتكتب في قسم لوحده عشان توافق عليه الأول.

## القواعد الـ 11

| # | القاعدة | باختصار |
|---|---------|---------|
| 1 | Naming (التسمية) | الاسم يقول الحاجة دي إيه أو بتعمل إيه |
| 2 | Comment (التعليقات) | اشرح ليه، مش اللي الكود بيقوله أصلاً |
| 3 | Consistency (الاتساق) | أسلوب واحد في المشروع كله |
| 4 | Early Return (الخروج المبكر) | اخرج بدري بدل الشروط المتداخلة |
| 5 | Function (الدوال) | كل دالة تعمل مهمة واحدة |
| 6 | Single Responsibility (المسؤولية الواحدة) | كل موديول أو كلاس له سبب واحد للتغيير |
| 7 | Abstraction (التجريد) | خبّي التفاصيل المشوشة من غير مبالغة |
| 8 | Separation of Concerns (فصل المسؤوليات) | افصل الواجهة عن الـ API عن الـ Business Logic |
| 9 | DRY (عدم تكرار الكود) | الكود المكرر يتجمع في مكان واحد |
| 10 | Don't Repeat Logic (عدم تكرار المنطق) | كل قاعدة عمل تتكتب في مكان واحد |
| 11 | Error Handling (التعامل مع الأخطاء) | عالج الفشل صراحةً بدل ما تفترض إن كل حاجة هتنجح |

القواعد متقسمة على أربع مجموعات: قراءة الكود، والبنية، والتكرار، ومعالجة الأخطاء. الـ Agent بيقرأ تفاصيل المجموعة بس لما يلاقي فيها مشكلة.

## التثبيت

الـ Skill هي فولدر `clean-code/`. انسخه في فولدر الـ skills بتاع الـ Agent. نفّذ الأوامر من فولدر مفيهوش فولدر اسمه `clean-code-skill` بالفعل.

OpenCode لكل المشاريع (Linux و macOS):

```bash
git clone https://github.com/omaromar9091/clean-code-skill.git
mkdir -p ~/.config/opencode/skills
cp -r clean-code-skill/clean-code ~/.config/opencode/skills/
```

OpenCode على ويندوز (PowerShell):

```powershell
git clone https://github.com/omaromar9091/clean-code-skill.git
New-Item -ItemType Directory -Force "$HOME\.config\opencode\skills"
Copy-Item -Recurse clean-code-skill\clean-code "$HOME\.config\opencode\skills\"
```

لمشروع واحد، انسخ `clean-code/` جوه `.opencode/skills/` في المشروع. لـ Claude Code، انسخه في `~/.claude/skills/` (لكل المشاريع) أو `.claude/skills/` (لمشروع واحد).

أعد تشغيل الـ Agent بعد النسخ.

## الاستخدام

أضمن طريقة إنك تذكر اسم الـ Skill في الطلب:

```
Use the clean-code skill to review src/ (report only)
Use the clean-code skill to refactor routes.py
Apply only Early Return and Naming from the clean-code skill on services/
```

وبتشتغل لوحدها كمان لما تطلب تنضيف أو Refactor أو مراجعة للكود. والطلبات بالعربي شغالة:

```
راجع الكود ده بـ clean-code skill وطلعلي تقرير بس
نضّف ملف routes.py وطبّق Early Return وفصل المسؤوليات
```

اعمل commit لشغلك قبل ما تبدأ، عشان تقدر ترجع عن أي خطوة. المشاريع اللي فيها تيستات بتطلع نتايج أحسن، لأن الـ Agent بيشغّلها بعد كل تعديل.

## هيكل الـ repo

```
clean-code-skill/
├── README.md
├── README.ar.md
├── LICENSE
├── clean-code/                  الـ Skill؛ انسخ الفولدر ده
│   ├── SKILL.md                 الـ workflow وجدول القواعد والحدود وشكل التقرير
│   └── references/
│       ├── readability.md       Naming, Comment, Consistency, Early Return
│       ├── structure.md         Function, Single Responsibility, Abstraction, Separation of Concerns
│       ├── duplication.md       DRY, Don't Repeat Logic
│       └── error-handling.md    Error Handling
└── examples/
    └── flask-refactor.md        مثال كامل قبل وبعد
```

ملف `SKILL.md` بيتحمّل كل ما الـ Skill تشتغل. ملفات `references/` بتتقرأ وقت الحاجة بس، وده بيخلي الـ context صغير.

## ملاحظات وتواصل

لو لقيت مشكلة أو عندك اقتراح، افتح Issue: https://github.com/omaromar9091/clean-code-skill/issues

## الرخصة

MIT. شوف [LICENSE](LICENSE).
