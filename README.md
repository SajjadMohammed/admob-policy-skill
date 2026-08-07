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

**D. Content policies** (18 categories), plus three sections that sit in the content track but
are really implementation problems:

| # | Topic |
|---|---|
| D2 | Publisher Restrictions — the separate mechanism that silently narrows advertiser demand instead of raising a warning |
| D3 | Abusive experiences — all eight named conditions (fake messages, unexpected click areas, misleading affordances, back-button trapping, social engineering, auto-redirect, fake pointers, malware) mapped to their Android form |
| D4 | Better Ads Standards — the Coalition's four disallowed mobile-app experiences |

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

**F. After the fix — the re-review.** The step where most remediations actually fail. A review
runs against the build **live on the store**, not the one sitting in Play Console, so a review
requested the moment an update is uploaded is a review of the old binary. The skill explains how
to tell that apart from a genuine second failure: a repeat rejection carrying the *same*
screenshots and a generic policy link means the reviewer found nothing new to say — check the
rollout status before touching the code again.

Plus: Native Validator setup, a 30-point pre-ship audit checklist, and a
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

مهارة لـ Claude Code تشخّص مخالفات إعلانات AdMob في تطبيقات أندرويد وتصلحها.

تحذيرات AdMob غامضة. تصلك رسالة منسوخة أصلاً من AdSense تتكلم عن الويب و iframes، وتطبيقك
أندرويد ولا علاقة له بشيء من ذلك. ولا تقول لك أين الخطأ في الكود.

هذه المهارة تترجم التحذير إلى تعديل تعرف كيف تنفّذه: أي قاعدة خالفتَ فعلاً، نصّها كما كتبته
Google، أي سطر في مشروعك يسبّبها، والكود البديل.

### ماذا تغطي

**أ — تنفيذ الإعلانات وأماكنها (12 مخالفة).** تعديل سلوك الإعلان، تصميم يوقع المستخدم في نقرات
غير مقصودة، إعلان بيني يظهر عند فتح التطبيق أو الخروج منه، إعلانات بينية متكررة، إعلان يعطّل
استخدام التطبيق، سوء استعمال إعلان فتح الشاشة، مخالفات الإعلان المدمج، إعلانات المكافأة
وموافقة المستخدم، إعلان لا يُميَّز عن المحتوى، كثرة الإعلانات مقابل المحتوى، إعلانات على شاشات
فارغة، وإعلانات خارج سياق المحتوى.

**ب — النقرات والحساب (5).** النقرات والمشاهدات المزيّفة، تحديث البانر أسرع من المسموح، مكافأة
المستخدم على النقر، تأجير حساب AdMob أو تضمينه في قوالب تُباع، وملف app-ads.txt.

**ج — الخصوصية (3).** موافقة مستخدمي أوروبا، التطبيقات التي يستخدمها أطفال، ومنع تسريب بيانات
المستخدم إلى طلبات الإعلانات.

**د — المحتوى وقيود العرض (4).** أنواع المحتوى التي تمنع الإعلانات، والقيود التي لا تمنع تطبيقك
بل تُضيّق المعلنين فتنخفض أرباحك دون أي تحذير. مع التجارب الخادعة بأنواعها الثمانية، ومعايير
Better Ads.

**هـ — Google Play، ثلاثة مسارات منفصلة عن AdMob تماماً.** هذه أكثر ما يُغفَل: تطبيق سليم
تماماً عند AdMob يمكن أن يُحذف من Play دون أن يظهر شيء في مركز سياسات AdMob. تشمل سياسة
الإعلانات، وقواعد تطبيقات الأطفال (الأشد صرامة على الإطلاق)، وقاعدة التوقيت في
Better Ads Experiences.

### وأيضاً

**و — بعد الإصلاح، إعادة المراجعة.** هنا يسقط أكثر الناس. المراجعة تفحص النسخة **المنشورة فعلاً
على المتجر**، لا التي رفعتَها إلى Play Console. فإن طلبتَ المراجعة فور الرفع، فحصوا النسخة
القديمة. والعلامة على ذلك: يردّون عليك **بنفس الصور** ورابط سياسات عام — أي لم يجدوا جديداً
يقولونه. راجع حالة النشر قبل أن تعدّل سطراً واحداً.

أداة Native Validator من Google، قائمة من 30 نقطة تفحصها قبل رفع أي نسخة، وقسم **"إنذارات كاذبة
— لا تصلحها"**: أخطاء شائعة يظنّها المطوّر مخالفة فيكسر كوداً سليماً بلا سبب.

### حدود المهارة

أندرويد فقط. لا تغطي AdSense للمواقع، ولا Ad Manager، ولا iOS، ولا سياسات شبكات الوساطة.

وليست مرجعاً شاملاً. سياسات Google موزّعة على عشرات الصفحات وتتغير دون إشعار. المهارة تدلّك على
المصدر الرسمي وتختصر عليك الطريق إليه، لا تحلّ محلّه. كل قاعدة معها اقتباس حرفي فهي محقَّقة من
مصدرها؛ وما دون ذلك اجتهاد قابل للنقاش.

### التثبيت

```
/plugin marketplace add SajjadMohammed/admob-policy-skill
/plugin install admob-policy@admob-skills
```

### الاستخدام

تعمل تلقائياً حين تذكر مشكلة إعلانات:

> وصلني تحذير Modified ad behavior من أدموب، افحص التطبيق وحلّها

أو استدعها مباشرة بـ `/admob-policy`.
