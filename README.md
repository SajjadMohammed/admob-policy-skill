# admob-policy — a Claude Code skill

Diagnose and fix **any AdMob / Google Publisher policy violation** in a native Android app.

AdMob Policy Center warnings are vague, reuse AdSense *web* boilerplate, and never point at a
line of code. This skill turns a warning into an actionable diff: it names the real rule, quotes
it verbatim from Google's official documentation, shows what triggers it in Android code, and
gives the fix.

Built from a real remediation of a "Modified ad behavior" enforcement on a published app.

## What it covers

**A. Ad implementation and placement**

| # | Violation |
|---|---|
| A1 | Modified ad behavior |
| A2 | Layout encouraging accidental clicks |
| A3 | Unexpected interstitials on app load / exit |
| A4 | Repeated or excessive interstitials |
| A5 | Ads interfering with app functionality |
| A6 | App open ad misuse |
| A7 | Native ad violations |
| A8 | Rewards implementation — user choice |
| A9 | Deceptive ad implementation / unclear labeling |
| A10 | Ad-to-content ratio |
| A11 | Inventory value — ads on screens without content |
| A12 | Out-of-context / replicated content |

**B.** Invalid traffic — **C.** EU user consent (UMP), child-directed apps and Designed for
Families — **D.** Content policies (12 categories)

Plus: Native Validator setup, a 12-point pre-ship audit checklist, and a
**"known false alarms — do not fix these"** section covering mistakes that are easy to make when
diagnosing from a screenshot (letterboxing is not distortion; `wrap_content` on a `MediaView`
breaks the 120x120dp video minimum; AdChoices placement cannot be judged from a screenshot;
IFRAME / floating-box clauses are web-only).

Every rule links back to its official Google source. Nothing in the catalog is inferred.

## Install

### As a plugin (recommended — stays updatable)

In Claude Code:

```
/plugin marketplace add SajjadMohammed/admob-policy-skill
/plugin install admob-policy@admob-skills
```

### Manual

Copy the skill folder into your skills directory:

```bash
# user-level — available in every project
git clone https://github.com/SajjadMohammed/admob-policy-skill
cp -r admob-policy-skill/plugins/admob-policy/skills/admob-policy ~/.claude/skills/
```

Project-level instead: copy it to `.claude/skills/` inside the repo.

## Use

The skill activates on its own when you mention an AdMob policy problem:

> I got a "Modified ad behavior" warning from AdMob — check my app and fix it

Or invoke it explicitly:

```
/admob-policy
```

## Scope

- Native **Android** apps (Java / Kotlin), Google Mobile Ads SDK.
- Diagnosis and remediation. Compliant code *templates* live in a companion skill,
  `admob-implementation`.
- Not legal advice. Google's documentation is the authority; the linked sources are canonical
  and this skill is a navigation layer over them. Policies change — verify before you ship.

## Contributing

Found a Policy Center warning this catalog does not cover, or a fix that turned out wrong? Open
an issue with the **exact warning text** from the Policy Center and the official documentation
URL that governs it. Rules go in verbatim, with a source — no paraphrase, no guesses.

## License

MIT — see [LICENSE](LICENSE).

---

## بالعربية

مهارة لـ Claude Code لتشخيص وحل مخالفات سياسات AdMob في تطبيقات أندرويد.

تحذيرات مركز السياسات في AdMob مبهمة وتستخدم نصوصاً منسوخة من AdSense الخاصة بالويب، ولا تشير
إلى موضع الخطأ في الكود. هذه المهارة تحوّل التحذير إلى تعديل ملموس: تحدد القاعدة الحقيقية،
تقتبسها بنصها من توثيق Google الرسمي، توضّح ما الذي يُطلقها في كود أندرويد، وتعطي الحل.

تغطي 18 مخالفة: التنفيذ والموضع (A1–A12)، الحركة غير الصالحة (B)، موافقة الاتحاد الأوروبي
والتطبيقات الموجّهة للأطفال (C)، وسياسات المحتوى (D). إضافة إلى Native Validator، وقائمة فحص
من 12 نقطة قبل النشر، وقسم "إنذارات كاذبة — لا تُصلحها".

**التثبيت:**

```
/plugin marketplace add SajjadMohammed/admob-policy-skill
/plugin install admob-policy@admob-skills
```
