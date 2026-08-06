# admob-policy â€” a Claude Code skill

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
| A8 | Rewards implementation â€” user choice |
| A9 | Deceptive ad implementation / unclear labeling |
| A10 | Ad-to-content ratio |
| A11 | Inventory value â€” ads on screens without content |
| A12 | Out-of-context / replicated content |

**B. Traffic and account behavior**

| # | Violation |
|---|---|
| B1 | Invalid traffic |
| B2 | Refresh rate outside the permitted range |
| B3 | Incentivized or compensated clicks |
| B4 | Framing, sub-syndication, and account sharing |
| B5 | Authorized inventory â€” app-ads.txt |

**C. Privacy and regulatory** â€” EU user consent (UMP), child-directed apps and Designed for
Families, plus the remaining Publisher Policy privacy items: personalized advertising, privacy
disclosures, identifying users, device and location data, COPPA.

**D. Content policies** (18 categories), plus three sections that sit in the content track but
are really implementation problems:

| # | Topic |
|---|---|
| D2 | Publisher Restrictions â€” the separate mechanism that silently narrows advertiser demand instead of raising a warning |
| D3 | Abusive experiences â€” all eight named conditions (fake messages, unexpected click areas, misleading affordances, back-button trapping, social engineering, auto-redirect, fake pointers, malware) mapped to their Android form |
| D4 | Better Ads Standards â€” the Coalition's four disallowed mobile-app experiences |

**E. Google Play â€” three separate enforcement tracks.** The gap most publishers miss: AdMob and
Play enforce independently, so an app that is perfectly AdMob-compliant can still be removed
from Play with a clean Policy Center.

| # | Track |
|---|---|
| E1 | Play Ads policy â€” disruptive ads, out-of-context ads, 15-second close, device-button interference, "Contains ads" declaration |
| E2 | Families Ads and Monetization â€” the strictest rule set: no app open ads, one placement per screen, 5-second close including rewarded, self-certified SDKs only, neutral age screen |
| E3 | Better Ads Experiences â€” timing, not frequency: an interstitial at the *end* of a section is fine, the same ad at the *start* of the next one is a violation |

E2 includes a table of every number that differs from the general rules, so a kids app is not
audited against the wrong limit.

Plus: Native Validator setup, a 24-point pre-ship audit checklist, and a
**"known false alarms â€” do not fix these"** section covering mistakes that are easy to make when
diagnosing from a screenshot (letterboxing is not distortion; `wrap_content` on a `MediaView`
breaks the 120x120dp video minimum; AdChoices placement cannot be judged from a screenshot;
IFRAME / floating-box clauses are web-only).

Every entry links back to its official Google source. Entries carrying a verbatim block quote
are verified against that source; entries without one are the author's reading, and the skill
tells Claude to say which kind it is relying on when it reports a finding.

## Install

### As a plugin (recommended â€” stays updatable)

In Claude Code:

```
/plugin marketplace add SajjadMohammed/admob-policy-skill
/plugin install admob-policy@admob-skills
```

### Manual

Copy the skill folder into your skills directory:

```bash
# user-level â€” available in every project
git clone https://github.com/SajjadMohammed/admob-policy-skill
cp -r admob-policy-skill/plugins/admob-policy/skills/admob-policy ~/.claude/skills/
```

Project-level instead: copy it to `.claude/skills/` inside the repo.

## Use

The skill activates on its own when you mention an AdMob policy problem:

> I got a "Modified ad behavior" warning from AdMob â€” check my app and fix it

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
web â€” but every code fix here is Android, and the AdSense-only rules are absent.

**Not exhaustive, and it cannot be.** Google's policy surface spans dozens of pages that change
without notice. This is a navigation layer over the official sources, not a replacement for
them. Open the source before acting on any entry.

Diagnosis and remediation only. Compliant code *templates* live in a companion skill,
`admob-implementation`. Not legal advice.

## Contributing

Found a Policy Center warning this catalog does not cover, or a fix that turned out wrong? Open
an issue with the **exact warning text** from the Policy Center and the official documentation
URL that governs it. Rules go in verbatim, with a source â€” no paraphrase, no guesses.

## License

MIT â€” see [LICENSE](LICENSE).

---

## ط¨ط§ظ„ط¹ط±ط¨ظٹط©

ظ…ظ‡ط§ط±ط© ظ„ظ€ Claude Code ظ„طھط´ط®ظٹطµ ظˆط­ظ„ ظ…ط®ط§ظ„ظپط§طھ ط³ظٹط§ط³ط§طھ AdMob ظپظٹ طھط·ط¨ظٹظ‚ط§طھ ط£ظ†ط¯ط±ظˆظٹط¯.

طھط­ط°ظٹط±ط§طھ ظ…ط±ظƒط² ط§ظ„ط³ظٹط§ط³ط§طھ ظپظٹ AdMob ظ…ط¨ظ‡ظ…ط© ظˆطھط³طھط®ط¯ظ… ظ†طµظˆطµط§ظ‹ ظ…ظ†ط³ظˆط®ط© ظ…ظ† AdSense ط§ظ„ط®ط§طµط© ط¨ط§ظ„ظˆظٹط¨طŒ ظˆظ„ط§ طھط´ظٹط±
ط¥ظ„ظ‰ ظ…ظˆط¶ط¹ ط§ظ„ط®ط·ط£ ظپظٹ ط§ظ„ظƒظˆط¯. ظ‡ط°ظ‡ ط§ظ„ظ…ظ‡ط§ط±ط© طھط­ظˆظ‘ظ„ ط§ظ„طھط­ط°ظٹط± ط¥ظ„ظ‰ طھط¹ط¯ظٹظ„ ظ…ظ„ظ…ظˆط³: طھط­ط¯ط¯ ط§ظ„ظ‚ط§ط¹ط¯ط© ط§ظ„ط­ظ‚ظٹظ‚ظٹط©طŒ
طھظ‚طھط¨ط³ظ‡ط§ ط¨ظ†طµظ‡ط§ ظ…ظ† طھظˆط«ظٹظ‚ Google ط§ظ„ط±ط³ظ…ظٹطŒ طھظˆط¶ظ‘ط­ ظ…ط§ ط§ظ„ط°ظٹ ظٹظڈط·ظ„ظ‚ظ‡ط§ ظپظٹ ظƒظˆط¯ ط£ظ†ط¯ط±ظˆظٹط¯طŒ ظˆطھط¹ط·ظٹ ط§ظ„ط­ظ„.

طھط؛ط·ظٹ: ط§ظ„طھظ†ظپظٹط° ظˆط§ظ„ظ…ظˆط¶ط¹ (A1â€“A12)طŒ ط§ظ„ط­ط±ظƒط© ظˆط§ظ„ط³ظ„ظˆظƒ â€” ط§ظ„ط­ط±ظƒط© ط؛ظٹط± ط§ظ„طµط§ظ„ط­ط© ظˆظ…ط¹ط¯ظ‘ظ„ ط§ظ„طھط­ط¯ظٹط« ظˆط§ظ„ظ†ظ‚ط±ط§طھ
ط§ظ„ظ…ط­ظپظژظ‘ط²ط© ظˆط§ظ„طھظˆط²ظٹط¹ ط§ظ„ظپط±ط¹ظٹ ظˆ app-ads.txt (B1â€“B5)طŒ ط§ظ„ط®طµظˆطµظٹط© ظˆط§ظ„طھظ†ط¸ظٹظ… â€” ظ…ظˆط§ظپظ‚ط© ط§ظ„ط§طھط­ط§ط¯ ط§ظ„ط£ظˆط±ظˆط¨ظٹ
ظˆط§ظ„طھط·ط¨ظٹظ‚ط§طھ ط§ظ„ظ…ظˆط¬ظ‘ظ‡ط© ظ„ظ„ط£ط·ظپط§ظ„ ظˆط¨ظ‚ظٹط© ط¨ظ†ظˆط¯ ط§ظ„ط®طµظˆطµظٹط© (C1â€“C3)طŒ ط³ظٹط§ط³ط§طھ ط§ظ„ظ…ط­طھظˆظ‰ ظˆظ‚ظٹظˆط¯ ط§ظ„ظ†ط§ط´ط±ظٹظ† (DطŒ D2)طŒ
ظˆظ…ط³ط§ط±ط§طھ Google Play ط§ظ„ط«ظ„ط§ط«ط© ط§ظ„ظ…ظ†ظپطµظ„ط© طھظ…ط§ظ…ط§ظ‹ ط¹ظ† AdMob â€” ط³ظٹط§ط³ط© ط§ظ„ط¥ط¹ظ„ط§ظ†ط§طھطŒ ظˆظ…طھط·ظ„ط¨ط§طھ Families Ads
and Monetization ط§ظ„ط£ط´ط¯ طµط±ط§ظ…ط©طŒ ظˆ Better Ads Experiences (E1â€“E3).

ط¥ط¶ط§ظپط© ط¥ظ„ظ‰ Native ValidatorطŒ ظˆظ‚ط§ط¦ظ…ط© ظپط­طµ ظ…ظ† 21 ظ†ظ‚ط·ط© ظ‚ط¨ظ„ ط§ظ„ظ†ط´ط±طŒ ظˆظ‚ط³ظ… "ط¥ظ†ط°ط§ط±ط§طھ ظƒط§ط°ط¨ط© â€” ظ„ط§ طھظڈطµظ„ط­ظ‡ط§".

**ط§ظ„ظ†ط·ط§ظ‚ ظˆط­ط¯ظˆط¯ظ‡.** ط£ظ†ط¯ط±ظˆظٹط¯ ظپظ‚ط·. ظ„ط§ طھط؛ط·ظٹ AdSense ظ„ظ„ظˆظٹط¨ ظˆظ„ط§ Ad Manager ظˆظ„ط§ iOS ظˆظ„ط§ ط³ظٹط§ط³ط§طھ
ط§ظ„ظˆط³ط·ط§ط،. ظˆظ„ظٹط³طھ ط´ط§ظ…ظ„ط© â€” ط³ط·ط­ ط³ظٹط§ط³ط§طھ Google ط¹ط´ط±ط§طھ ط§ظ„طµظپط­ط§طھ طھطھط؛ظٹط± ط¯ظˆظ† ط¥ط´ط¹ط§ط±طŒ ظˆظ‡ط°ظ‡ ط·ط¨ظ‚ط© طھظ†ظ‚ظ‘ظ„ ظپظˆظ‚
ط§ظ„ظ…طµط§ط¯ط± ط§ظ„ط±ط³ظ…ظٹط© ظ„ط§ ط¨ط¯ظٹظ„ ط¹ظ†ظ‡ط§. ظ…ط§ ط¹ظ„ظٹظ‡ ط§ظ‚طھط¨ط§ط³ ط­ط±ظپظٹ ظ…ط­ظ‚ظژظ‘ظ‚ ظ…ظ† ظ…طµط¯ط±ظ‡طŒ ظˆظ…ط§ ط¯ظˆظ†ظ‡ ط§ط¬طھظ‡ط§ط¯.

**ط§ظ„طھط«ط¨ظٹطھ:**

```
/plugin marketplace add SajjadMohammed/admob-policy-skill
/plugin install admob-policy@admob-skills
```

