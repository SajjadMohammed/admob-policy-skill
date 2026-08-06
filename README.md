# admob-policy — a Claude Code skill

Diagnose and fix **AdMob, Google Publisher, and Google Play ad policy violations** in a native
Android app.

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

**B. Traffic and account behavior**

| # | Violation |
|---|---|
| B1 | Invalid traffic |
| B2 | Refresh rate outside the permitted range |
| B3 | Incentivized or compensated clicks |
| B4 | Framing, sub-syndication, and account sharing |
| B5 | Authorized inventory — app-ads.txt |

**C. Privacy and regulatory** — EU user consent (UMP), child-directed apps and Designed for
Families, plus the remaining Publisher Policy privacy items: personalized advertising, privacy
disclosures, identifying users, device and location data, COPPA.

**D. Content policies** (18 categories) and **D2. Publisher Restrictions** — the separate
mechanism that silently narrows advertiser demand instead of raising a warning.

**E. Google Play — three separate enforcement tracks.** The gap most publishers miss: AdMob and
Play enforce independently, so an app that is perfectly AdMob-compliant can still be removed
from Play with a clean Policy Center.

| # | Track |
|---|---|
| E1 | Play Ads policy — disruptive ads, out-of-context ads, 15-second close, device-button interference, "Contains ads" declaration |
| E2 | Families Ads and Monetization — the strictest rule set: no app open ads, one placement per screen, 5-second close including rewarded, self-certified SDKs only, neutral age screen |
| E3 | Better Ads Experiences — timing, not frequency: an interstitial at the *end* of a section is fine, the same ad at the *start* of the next one is a violation |

E2 includes a table of every number that differs from the general rules, so a kids app is not
audited against the wrong limit.

Plus: Native Validator setup, a 21-point pre-ship audit checklist, and a
**"known false alarms — do not fix these"** section covering mistakes that are easy to make when
diagnosing from a screenshot (letterboxing is not distortion; `wrap_content` on a `MediaView`
breaks the 120x120dp video minimum; AdChoices placement cannot be judged from a screenshot;
IFRAME / floating-box clauses are web-only).

Every entry links back to its official Google source. Entries carrying a verbatim block quote
are verified against that source; entries without one are the author's reading, and the skill
tells Claude to say which kind it is relying on when it reports a finding.

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

## Scope and limits

**Covered.** Native **Android** apps (Java / Kotlin) on the Google Mobile Ads SDK, across three
enforcement tracks: AdMob program policies, the Google Publisher Policies and Restrictions, and
Google Play's own Ads policy.

**Not covered.** AdSense for web, Ad Manager / AdX specifics, iOS requirements (App Tracking
Transparency, SKAdNetwork), mediation partner policies, and advertiser-side policies. AdMob and
AdSense share the Google Publisher Policies, so the content and privacy sections do transfer to
web — but every code fix here is Android, and the AdSense-only rules are absent.

**Not exhaustive, and it cannot be.** Google's policy surface spans dozens of pages that change
without notice. This is a navigation layer over the official sources, not a replacement for
them. Open the source before acting on any entry.

Diagnosis and remediation only. Compliant code *templates* live in a companion skill,
`admob-implementation`. Not legal advice.

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

تغطي: التنفيذ والموضع (A1–A12)، الحركة والسلوك — الحركة غير الصالحة ومعدّل التحديث والنقرات
المحفَّزة والتوزيع الفرعي و app-ads.txt (B1–B5)، الخصوصية والتنظيم — موافقة الاتحاد الأوروبي
والتطبيقات الموجّهة للأطفال وبقية بنود الخصوصية (C1–C3)، سياسات المحتوى وقيود الناشرين (D، D2)،
ومسارات Google Play الثلاثة المنفصلة تماماً عن AdMob — سياسة الإعلانات، ومتطلبات Families Ads
and Monetization الأشد صرامة، و Better Ads Experiences (E1–E3).

إضافة إلى Native Validator، وقائمة فحص من 21 نقطة قبل النشر، وقسم "إنذارات كاذبة — لا تُصلحها".

**النطاق وحدوده.** أندرويد فقط. لا تغطي AdSense للويب ولا Ad Manager ولا iOS ولا سياسات
الوسطاء. وليست شاملة — سطح سياسات Google عشرات الصفحات تتغير دون إشعار، وهذه طبقة تنقّل فوق
المصادر الرسمية لا بديل عنها. ما عليه اقتباس حرفي محقَّق من مصدره، وما دونه اجتهاد.

**التثبيت:**

```
/plugin marketplace add SajjadMohammed/admob-policy-skill
/plugin install admob-policy@admob-skills
```
