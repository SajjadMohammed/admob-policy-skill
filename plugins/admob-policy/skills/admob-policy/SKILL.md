---
name: admob-policy
description: Diagnose and fix any AdMob, Google Publisher, or Google Play ads policy violation in a native Android app — full catalog of Policy Center issues with the concrete code fix for each. Covers Modified ad behavior, accidental clicks, unexpected interstitials, ad density, ads interfering with navigation, inventory value, rewards user choice, invalid traffic, refresh rate limits, incentivized clicks, sub-syndication, app-ads.txt, EU consent, personalized advertising, privacy disclosures, COPPA and child-directed tagging, content policies, Publisher Restrictions (limited ads), and Google Play's separate Ads policy including disruptive and out-of-context ads. Use when an AdMob Policy Center warning arrives, an app is restricted or ads are limited, revenue drops with no warning, a Play listing is flagged, before shipping ad code, or when auditing a banner / interstitial / native / app-open / rewarded implementation. Triggers - AdMob, AdSense policy, Policy Center, Modified ad behavior, policy violation, ads restricted, ad serving limited, accidental clicks, invalid traffic, refresh rate, incentivized clicks, app-ads.txt, AdChoices, NativeAdView, MediaView, AppOpenAd, UMP consent, COPPA, Designed for Families, Google Play ads policy, disruptive ads, out-of-context ads.
---

# AdMob policy — diagnosis and fixes

Companion to `admob-implementation`, which holds the compliant code templates. This skill
is the diagnostic side: identify the violation, quote the rule, apply the fix.

## Scope and limits — read this before trusting the catalog

**Covered.** Native **Android** apps using the Google Mobile Ads SDK, under three enforcement
tracks: AdMob program policies, the Google Publisher Policies and Restrictions, and Google
Play's own Ads policy.

**Not covered.** AdSense for web, Ad Manager / AdX specifics, iOS-only requirements
(App Tracking Transparency, SKAdNetwork), mediation partner policies, and Google's advertiser-
side policies. AdMob and AdSense **share the Google Publisher Policies**, so sections C and D
transfer to web — but every code fix here is Android, and the AdSense-only rules are absent.

**Not exhaustive, and cannot be.** Google's policy surface spans dozens of pages that change
without notice. This catalog is a navigation layer over the official sources in Step 2, not a
replacement for them. Always open the source before acting on an entry.

**Anything not backed by a quote is a judgement call.** Entries carrying a verbatim block quote
are verified against the linked source. Entries without one are the author's reading. Apply the
reporting rule at the bottom of this file and say which kind you are relying on.

## Step 1 — read the warning correctly

AdMob reuses **AdSense web boilerplate** in violation descriptions. The examples inside are
usually irrelevant to a native app. Do not chase them.

"Modified ad behavior" arrives with this text:

> Publishers are not permitted to alter the behavior or targeting of Google ads.
> This includes implementing the AdSense ad code in a "floating box script," or
> altering the ad-targeting using hidden keywords or IFRAMES.

| Phrase | Applies to a native Android app? |
|---|---|
| "floating box script" | No — JavaScript concept |
| "hidden keywords" | No — HTML concept |
| "IFRAMEs" | No — HTML element |
| "alter the **behavior**" (lead-in) | **Yes** — the only operative clause |

Confirm the app has no WebView before dismissing the web clauses:

```
grep -rniE "webview|iframe|loadUrl" app/src/main
```

## Step 2 — official sources, always verify against these

| Topic | URL |
|---|---|
| Google Publisher Policies | https://support.google.com/publisherpolicies/answer/10502938 |
| Google Publisher Restrictions | https://support.google.com/publisherpolicies/answer/10437795 |
| Ad placement / interstitial policies | https://support.google.com/admob/answer/6201362 |
| Native ads policy | https://support.google.com/admob/answer/6329638 |
| App open ad guidance | https://support.google.com/admob/answer/9341964 |
| Rewarded inventory policy | https://support.google.com/admob/answer/7313578 |
| Invalid traffic | https://support.google.com/admob/answer/3342054 |
| EU user consent policy | https://support.google.com/admob/answer/7676680 |
| Child-directed / families | https://support.google.com/admob/answer/6223431 |
| Native Validator | https://developers.google.com/admob/android/native/validator |
| App open implementation guide | https://developers.google.com/admob/android/app-open |
| AdMob behavioral policies | https://support.google.com/admob/answer/2753860 |
| AdMob implementation guidance | https://support.google.com/admob/answer/2936217 |
| Banner refresh rate | https://support.google.com/admob/answer/3245199 |
| **Google Play Ads policy** (separate track) | https://support.google.com/googleplay/android-developer/answer/9857753 |
| app-ads.txt | https://support.google.com/admob/answer/9363762 |

---

# The violation catalog

Each entry: what it means, what triggers it in Android code, and the fix.

## A. Ad implementation and placement

### A1. Modified ad behavior

**Rule.** Publishers may not alter the behavior of Google ads.

**Triggers in Android code**

- Rewriting an ad view's `LayoutParams`, `dimensionRatio`, width, or height **after** the ad
  object arrives.
- Wrapping an ad view in a container that clips, scales, or transforms it.
- Intercepting or synthesizing touches on an ad view.
- Any `setOnClickListener` on a `NativeAdView`, `AdView`, or an ad's parent container.

**Fix**

```java
// WRONG — reshapes the served creative after load
private void applyMediaAspectRatio(MediaView mediaView, float ratio) {
    ViewGroup.LayoutParams lp = mediaView.getLayoutParams();
    ((ConstraintLayout.LayoutParams) lp).dimensionRatio = String.valueOf(ratio);
    mediaView.setLayoutParams(lp);
}

// RIGHT — request the shape at load time, never touch the view afterwards
new NativeAdOptions.Builder()
        .setMediaAspectRatio(NativeAdOptions.NATIVE_MEDIA_ASPECT_RATIO_LANDSCAPE)
        .build();
```

Keep the ad's size fixed in XML. Never call `setLayoutParams` on an ad view in a load or
bind callback.

---

### A2. Layout encourages accidental clicks

The single most common app violation. Often paired with **Invalid traffic** (B1).

**Triggers**

- A button, FAB, or interactive control adjacent to or overlapping an ad.
- A banner flush against the bottom navigation bar with no separation.
- An ad appearing where a tappable element was a moment earlier.
- An ad card styled to look identical to tappable app content.
- Ads inside a scrolling list with a floating overlay above them.

**Fix**

```xml
<!-- Separate the banner from navigation and from content above it -->
<FrameLayout
    android:id="@+id/adContainerView"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginTop="8dp"
    android:layout_marginBottom="12dp"
    app:layout_constraintBottom_toBottomOf="parent" />
```

Remove any floating element over a list carrying ads. For a scroll-to-top FAB, replace it
with a toolbar tap or a bottom-nav re-tap — both cost zero screen space:

```java
binding.toolbar.setOnClickListener(v -> binding.recyclerView.smoothScrollToPosition(0));

binding.bottomNavigation.setOnItemReselectedListener(
        item -> binding.recyclerView.smoothScrollToPosition(0));
```

Add a global tap debounce so a double tap cannot register two clicks:

```java
public static boolean isValidClick() {
    long now = System.currentTimeMillis();
    if (now - lastClickTime < 600) return false;
    lastClickTime = now;
    return true;
}
```

---

### A3. Unexpected launch interstitials

**Rule, verbatim.**

> "Do not place interstitial ads on app load and when exiting apps as interstitials should
> only be placed in between pages of app content."

> "Interstitial ads should only be implemented at logical breaks in between your app's
> content (e.g. pages, stages, or levels) to ensure that the user is prepared to engage
> with the ad."

**Triggers**

- `showInterstitial()` from a splash screen or an Activity's `onCreate`.
- An interstitial on back-press or app exit.
- An interstitial that appears mid-scroll or mid-interaction because it finished loading
  late.

**Fix**

Use an **app open ad** for launch, never an interstitial. Show interstitials only on a
deliberate content transition. Preload well in advance so carrier latency never makes one
appear at a random moment:

```java
// load early, in onCreate
adManager.loadInterstitial(requireContext(), getString(R.string.interstitial_id));

// show only at a real transition, and never block the user on a missing ad
if (mInterstitialAd == null) {
    listener.onAdClosed();          // proceed immediately
    loadInterstitial(activity, adUnitId);
    return;
}
```

---

### A4. Repeated or excessive interstitials

**Rule, verbatim.**

> "Repeated interstitial ads often lead to poor user experiences and accidental clicks."
> Place no more than one interstitial ad after every two user actions within the app.

> Apps cannot display an interstitial ad immediately after another interstitial was shown
> and closed by the user.

**Fix**

```java
public class ClickAdController {

    private static final int CLICKS_BEFORE_AD = 5;   // ≥ 3; 5–7 is comfortable

    private int clickCounter = 0;
    private boolean isFullScreenShowing = false;

    public boolean onItemClicked() {
        if (isFullScreenShowing) return false;      // never stack full-screen ads

        clickCounter++;
        if (clickCounter >= CLICKS_BEFORE_AD) {
            clickCounter = 0;
            return true;
        }
        return false;
    }

    public void onFullScreenAdShown()     { isFullScreenShowing = true; clickCounter = 0; }
    public void onFullScreenAdDismissed() { isFullScreenShowing = false; }
    public void resetForNewScreen()       { clickCounter = 0; }
}
```

The `isFullScreenShowing` guard is what prevents back-to-back full-screen ads. Reset the
counter on every screen entry.

---

### A5. Ads interfering with app functionality

**Rule.** Google ads may not overlay navigation items or severely obstruct content.

The rule runs **both ways**. Google Publisher Restrictions names "Ads interfering" as ads
obscuring content **and content obscuring ads**. App UI covering a banner restricts that
inventory even though nothing appears in the Policy Center.

**Triggers**

- A banner covering a toolbar, tab bar, or bottom navigation.
- App UI covering the banner: a sticky footer, snackbar, bottom sheet, or expanded FAB menu
  drawn over the ad container.
- An ad pushing content off-screen or blocking a required control.
- An ad container using `padding` instead of `margin`, shrinking the ad frame.
- Content that scrolls underneath an anchored ad without bottom inset.

**Fix**

Anchor the ad **outside** the content region, never on top of it:

```xml
<androidx.recyclerview.widget.RecyclerView
    android:id="@+id/recyclerView"
    android:layout_width="match_parent"
    android:layout_height="0dp"
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintBottom_toTopOf="@id/adContainerView" />

<FrameLayout
    android:id="@+id/adContainerView"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginTop="8dp"
    app:layout_constraintBottom_toBottomOf="parent" />
```

Use `layout_margin`, never `android:padding`, on ad containers — padding restricts the ad
frame and is reported as "resizing ad frames".

---

### A6. App open ad misuse

**Rules, verbatim.**

> "the app open ad is placed **on top of the app's loading screen**. The loading screen is
> seen underneath the ad."

> "**Don't display app open ads on top of other ads** (for example, content with banner ads)."

> "Don't display ads immediately before or after app open ads."

> "App open ad units should only appear when the user **opens or switches back** to an app."

Apps in Google Play's **Designed for Families** program cannot use this format at all.

**Triggers**

- Showing it from a content Activity's `onCreate` or `onWindowFocusChanged`, so it lands on
  a half-drawn screen and app content shows through.
- Showing it on foreground return while a banner is on screen.
- Two `AppOpenManager` instances both registering lifecycle callbacks and both showing.

**Fix**

Cold start — show it from the splash, over the loading screen:

```java
new Handler(Looper.getMainLooper()).postDelayed(this::showAdThenNavigate, 2500);

private void showAdThenNavigate() {
    if (isFinishing() || isDestroyed()) return;
    AppOpenManager manager = ((MyApplication) getApplication()).getAppOpenManager();
    if (manager == null) { navigateToMain(); return; }
    manager.showAdIfAvailable(this, this::navigateToMain);
}
```

Warm start — gate on whether a banner is on screen, not on the Activity class (useless in a
single-Activity app):

```java
@Override
public void onStart(@NonNull LifecycleOwner owner) {
    if (System.currentTimeMillis() - lastDismissTime < 1000) return;

    Activity activity = currentActivity.get();
    if (activity == null || activity instanceof SplashActivity || isShowingAd) return;

    if (BannerTracker.hasBannerOnScreen()) return;   // policy: not on top of other ads

    showAdIfAvailable(activity, null);
}
```

Create exactly one `AppOpenManager`, and call `fetchAd()` only after `MobileAds.initialize`
completes.

---

### A7. Native ad violations

**Rules, verbatim.**

> "app content **must not overlap** the native ad" — causes invalid clicks

> "camouflaging the Ad attribution or AdChoices overlay" is prohibited

> "Scaling an image or video element can be done by modifying the aspect ratio. However,
> **distorting (stretching/squeezing)** the image or video by changing its aspect ratio is
> not allowed."

> Non-video: the primary image may be cropped **symmetrically by up to 10% in width**;
> height cropping is not allowed.

> "for native video ads, the main asset `MediaView` must be at least **120x120dp**. Video
> ads won't serve to implementations with main asset MediaView smaller than 120dp in any
> dimension."

**Triggers**

- A FAB or sticky element crossing an ad card during scroll.
- The "Ad" badge hidden, tiny, or low-contrast.
- A card styled to be indistinguishable from tappable app content.
- Mutating `MediaView` size after load.

**Fix**

```java
// preserve the creative's ratio; letterboxing is allowed, distortion is not
mediaView.setImageScaleType(ImageView.ScaleType.FIT_CENTER);
mediaView.setMediaContent(mediaContent);
adView.setNativeAd(nativeAd);   // always last
```

```xml
<!-- Fixed ratio is allowed scaling. Never wrap_content: video needs ≥ 120dp both ways. -->
<com.google.android.gms.ads.nativead.MediaView
    android:id="@+id/ad_media"
    android:layout_width="0dp"
    android:layout_height="0dp"
    app:layout_constraintDimensionRatio="16:9" />
```

Keep the ad badge visible and legible. Leave AdChoices placement to the SDK unless a
real-device test proves it wrong.

---

### A8. Rewards implementation — user choice

**Rules.**

- The user must **affirmatively and unambiguously opt in** — a button meaning "yes" or
  "accept". No auto-play.
- The user opts in **per ad**. There is no blanket "always show rewarded ads" consent.
- The ad must not oblige interaction — it must be skippable or dismissable.
- Rewarded **interstitial** is the exception: no opt-in, but an intro screen explaining the
  reward with an **opt-out** is required before it shows.

**Triggers**

- A rewarded ad shown on screen entry, timer, or level completion without a tap.
- A settings toggle that auto-plays rewarded ads.
- Granting the reward in `onAdDismissedFullScreenContent`, which pays users who skipped.

**Fix**

```java
// the button label must state the reward, e.g. "Watch an ad to unlock X"
binding.btnWatchAd.setOnClickListener(v -> {
    if (rewardedAd == null) { loadRewarded(); return; }

    rewardedAd.show(this, rewardItem ->
            grantReward(rewardItem.getType(), rewardItem.getAmount()));
});
```

Grant the reward **only** inside `onUserEarnedReward`.

---

### A9. Deceptive ad implementation / unclear labeling

**Rule.** Ads must be clearly distinguishable from app content and must not be labeled in a
way that misleads.

**Triggers**

- Native ad card visually identical to content cards.
- Headings like "Related", "You may also like", or "Recommended" above ads.
- Ads dressed as system dialogs, notifications, download buttons, or close buttons.
- A fake "X" that is actually the ad.

**Fix**

Label ads with a neutral word — "Ad", "Advertisement", "Sponsored", or the local
equivalent — and give the ad card a visual boundary distinct from content cards. Never
imply the ad is app content or a system element.

---

### A10. Ad-to-content ratio

**Rule.** Excessive ads that overwhelm publisher content are prohibited.

**Triggers**

- More ads than content items in a list.
- Multiple banners on one screen.
- A banner plus a native ad plus an interstitial in the same short flow.

**Fix**

Keep list ad density sane — first slot around index 2, then every 6 items, capped near 10:

```java
if (adCount < MAX_ADS && i >= AD_FIRST_INDEX
        && (i - AD_FIRST_INDEX) % AD_INTERVAL == 0) {
    mixed.add(new AdItem());
    adCount++;
}
```

One banner per screen. Never a banner and a native ad in the same viewport.

---

### A11. Inventory value — ads on screens without content

**Rule.** Ads are prohibited on low-content, under-construction, or utility-only screens.

**Triggers**

- Ads on a splash, loading, login, error, "no results", or "coming soon" screen.
- Ads on a settings or About screen with almost no content.
- Ads showing before content has loaded.

**Fix**

Load the ad container only once real content is on screen, and hide it on empty states:

```java
if (results.isEmpty()) {
    binding.adContainerView.setVisibility(View.GONE);
} else {
    binding.adContainerView.setVisibility(View.VISIBLE);
}
```

The splash exception is the app open ad, which is *required* to be there — a banner or
interstitial on a splash is not.

---

### A12. Out-of-context ads and replicated content

**Rules.** Ads must clearly associate with relevant publisher content. Ads are disallowed on
screens whose material is copied without attribution.

**Triggers**

- Ads floating over a screen with no owned content around them.
- Content scraped from another source with no attribution.

**Fix**

Place ads within or beside genuine owned content. Attribute any third-party material.

---

## B. Traffic

### B1. Invalid traffic

**Rule.** Any clicks or impressions that artificially inflate advertiser cost or publisher
earnings. Officially listed examples:

1. Publishers clicking their own live ads
2. Repeated ad clicks or impressions by one or more users
3. Publishers encouraging clicks, including implementations causing accidental high-volume clicks
4. Automated clicking tools, robots, or deceptive software

> "ultimately it is your responsibility as the publisher to ensure that the traffic on your
> ads is valid."

**Triggers in code**

- Testing against **live production ad unit IDs**.
- Reloading a banner on a short timer on top of the SDK's own refresh.
- Requesting ads for off-screen or never-displayed views.
- Any text encouraging taps: "support us by clicking the ad", "tap the ad to continue".

**Fix**

Always use test IDs and a registered test device in debug builds:

```java
MobileAds.setRequestConfiguration(
        new RequestConfiguration.Builder()
                .setTestDeviceIds(Arrays.asList("YOUR_DEVICE_ID"))
                .build());
```

Never call `loadAd()` on a timer. Forward `onPause`/`onResume`/`onDestroy` so banners stop
while invisible. Destroy native ads when their screen dies. Never write copy that asks for
clicks.

---

### B2. Refresh rate outside the permitted range

**Rule, verbatim.**

> "The refresh rate may not be set to a value outside of the range specified in the SDK."

Custom banner refresh is **30–150 seconds**, or disabled. Collapsible banners: no less than
30 seconds. Google's implementation guidance adds: when a user navigates back and forth
between screens carrying ads, a new ad request should not be made sooner than **60 seconds**.

**Triggers**

- A `Handler`/`Timer`/coroutine calling `adView.loadAd()` on top of the SDK's own refresh.
- Recreating the `AdView` on every `onResume`, so tab switching fires a request per switch.
- Reloading a banner in a `ViewPager` page-change listener.

**Fix**

Set refresh in the AdMob console on the ad unit, never in code. Create the `AdView` once per
screen and reuse it; `onResume()` resumes the existing view, it does not re-request:

```java
// WRONG — a request per resume
@Override public void onResume() { super.onResume(); loadBannerAd(container, id); }

// RIGHT — load once, then only resume
@Override public void onResume() { super.onResume(); if (adView != null) adView.resume(); }
```

---

### B3. Incentivized or compensated clicks

**Rule, verbatim.**

> "Google ads may not be placed on apps that promise payment or incentives to users who
> click on or view ads."

This is **not** the rewarded ad format. Rewarded ads are a sanctioned product with their own
rules (A8). This rule bans paying users for *ordinary* ad views or clicks anywhere in the app.

**Triggers**

- Coins, points, or in-app currency granted for viewing a banner or native ad.
- "Watch ads to remove ads" loops that pay per impression on non-rewarded formats.
- Any copy pointing at an ad: "support us", "tap here to help".
- The app's purpose is driving traffic to a paid-to-click service.

**Fix**

Rewards flow **only** through `RewardedAd` / `RewardedInterstitialAd`, granted only inside
`onUserEarnedReward`. No reward is ever tied to a banner, native, interstitial, or app open
impression.

---

### B4. Framing, sub-syndication, and account sharing

**Rules, verbatim.**

> "Publishers may not enter into sub-syndication relationships." Google requires a direct
> publisher relationship.

Framing third-party content without permission and monetizing it is not allowed. If the
content is already monetized elsewhere, additional AdMob code must not be layered on the app
version of it. Sharing AdMob console access or Google SDK code with third parties without
written consent is also prohibited.

**Triggers**

- One AdMob account serving ads in apps published by other developers.
- A white-label app template shipped to clients with your ad unit IDs baked in.
- A `WebView` framing someone else's site with a banner over it.

**Fix**

One AdMob account per publisher, per app they actually own. Never distribute your ad unit IDs
inside a template or SDK handed to another party.

---

### B5. Authorized inventory — app-ads.txt

**Rule.** "Authorized inventory" is a named requirement in the Google Publisher Policies. An
`app-ads.txt` file declares who is authorized to sell your app's inventory. Missing or wrong,
programmatic demand drops and unauthorized resale becomes possible.

Not a violation that gets an app banned, but an enforcement-adjacent revenue problem that
belongs in the same audit.

**Fix**

1. Set a developer website in the Play Console listing.
2. Host `https://yourdomain.com/app-ads.txt`.
3. Paste the exact line AdMob generates under **Apps → app-ads.txt**.
4. Verify in the AdMob dashboard — it crawls and reports the status.

---

## C. Privacy and regulatory

### C1. EU user consent

**Rule.** Consent is required for cookies and mobile ad identifiers where legally mandated —
the EEA, UK, and Switzerland — per the ePrivacy Directive.

**Fix**

Integrate the UMP SDK, request consent before the first ad request, and gate ad loading on
the consent result. In this codebase pattern, that gate belongs at the top of every load
path:

```java
// if (!ConsentHelper.canRequestAds(context)) return;
```

Keep this check in one place so no format bypasses it.

### C2. Child-directed apps and families

**Rule.** Apps directed at children must tag ad requests. AdMob blocks non-self-certified
ad sources for such apps. **Designed for Families apps cannot use app open ads.**

**Fix**

```java
RequestConfiguration config = new RequestConfiguration.Builder()
        .setTagForChildDirectedTreatment(RequestConfiguration.TAG_FOR_CHILD_DIRECTED_TREATMENT_TRUE)
        .setMaxAdContentRating(RequestConfiguration.MAX_AD_CONTENT_RATING_G)
        .build();
MobileAds.setRequestConfiguration(config);
```

Set this **before** the first ad request, in the Application class.

### C3. The rest of the privacy-related policies

The Google Publisher Policies name six privacy items. C1 and C2 above cover two. The others
are frequently missed:

| Policy | What it requires |
|---|---|
| **Personalized advertising** | Do not build personalized ad audiences from sensitive categories (health, religion, sexual orientation, financial hardship, etc.). Do not personalize ads to children. |
| **Privacy disclosures** | The app's privacy policy must disclose third-party ad serving, cookies/device identifiers, and how to opt out. Must match the Play Data safety form. |
| **Identifying users** | Never pass personally identifiable information to Google — no email, name, phone, or precise location in an ad request, ad unit name, custom parameter, or URL. |
| **Use of device and location data** | Collect and transmit location or device data only with clear disclosure and consent. |
| **Cookies on Google domains** | Do not modify or interfere with cookies set on Google domains. |
| **COPPA** | US-specific. Overlaps C2 but is a separate named policy. |

**Triggers in Android code**

- A user ID, email, or account name used as an ad unit name, custom targeting key, or in an
  `AdRequest` extra.
- Requesting `ACCESS_FINE_LOCATION` purely to improve ad targeting, without disclosure.
- A privacy policy that never mentions Google AdMob.

**Fix**

Keep ad requests free of any user-derived value. If the app has a privacy policy screen, it
must name Google as an ad partner and link
`https://policies.google.com/technologies/partner-sites`.

---

## D. Content policies

These concern what the app itself contains, not how ads are coded. A violation here limits
or blocks ad serving regardless of a perfect implementation.

| Category | Short meaning |
|---|---|
| Illegal content | Illegal material or promotion of illegal activity |
| Intellectual property abuse | Copyright infringement, counterfeit goods |
| Dangerous or derogatory | Hatred, harassment, threats, exploitation |
| Animal cruelty | Gratuitous violence to animals, endangered species trade |
| Misrepresentative content | Misleading representation, unreliable and harmful claims, deceptive practices, manipulated media |
| Sexually explicit content | Graphic sexual material, non-consensual themes |
| Compensated sexual acts | Prostitution, escort services, sugar dating |
| Mail order brides | Arranged-marriage brokering |
| Adult themes in family content | Adult material disguised as family-appropriate |
| Child sexual abuse | Zero tolerance |
| Malware / unwanted software | Distribution or hosting |
| Enabling dishonest behavior | Tools for deception, hacking, academic cheating |
| Unsupported languages | Content must be in a supported language |
| Dishonest declarations | Publisher account information must be accurate |
| Abusive experiences | Misleading UI: fake system warnings, fake close buttons, auto-redirects |
| Better Ads Standards | Formats the Coalition for Better Ads found intrusive |
| Spam policies for web search | Applies to any web content tied to the account |
| Sanctions compliance | Publisher must not be in a sanctioned jurisdiction |

**Fix.** Remove or gate the offending content, then request review. No code change helps.

---

## D2. Publisher Restrictions — limited ads, not a violation

A different mechanism, easy to mistake for enforcement. Restricted content does **not** get the
app banned; it **narrows the pool of advertisers**, so revenue falls with no warning in the
Policy Center.

> "If your content is labeled with an inventory restriction, fewer advertising sources will be
> eligible to bid on it."

**Content restrictions** — sexual content · shocking content · explosives · guns, gun parts and
related products · other weapons · tobacco · recreational drugs · alcohol sale or misuse ·
online gambling · prescription drugs · unapproved pharmaceuticals and supplements ·
**app removed from the Google Play Store**.

That last one matters for Android: a Play removal restricts ad demand on the AdMob side too,
even after the AdMob Policy Center clears. Play and AdMob problems compound.

**Behavioral restriction** — "Ads interfering": Google-served ads obscuring content, **and
content obscuring ads**. Note the second direction — it is not in the A-section rules. A
sticky header, snackbar, or bottom sheet that covers a banner restricts that inventory.

**Video inventory restrictions** — accurate inventory description, supported implementations,
advertiser value protection, user experience requirements, and Connected TV / digital
out-of-home requirements. Applies only if the app serves video ad inventory.

**How this shows up.** eCPM collapses on a specific app or ad unit while the Policy Center
stays clean.

**Fix.** There is no code fix. Either the app's content sits in a restricted category — accept
the lower demand — or the app is being *miscategorized*, in which case correct the app's
content rating and category in AdMob and the Play Console.

---

## E. Google Play's Ads policy — a separate enforcement track

**This is the gap most publishers miss.** AdMob and Google Play enforce independently. A
perfectly AdMob-compliant app can still be **removed from Play** under Play's own Ads policy,
and the Policy Center will show nothing.

**Rules, verbatim.**

> Ads must not "force a user to click an ad or submit personal information," and must not
> "interfere with other apps, ads, or the operation of the device." They must be "easily
> dismissible without penalty."

> Ads must not "simulate or mimic the user interface of any app feature, such as notifications
> or warning elements."

> Full-screen interstitials must be "closeable after 15 seconds" — opted-in rewarded ads may
> exceed this.

> Ads must not interfere "with system or device buttons and ports."

> "It must be clear to the user which app is serving each ad."

Ads must not appear **outside** the app serving them, and must not be triggered by an exit
button.

**Triggers**

- An ad shown from a foreground service, notification, or on the lock screen.
- An interstitial on back-press — this violates Play's exit-button rule *and* AdMob's A3.
- An ad drawn over the navigation bar, status bar, or gesture area.
- A custom "close" affordance on a full-screen ad that is smaller or slower than the SDK's.
- Ads shown after the user has left the app.

**Fix**

Never show any ad from a Service, `BroadcastReceiver`, or notification. Show full-screen ads
only from a foreground Activity the user is currently in. Never replace or overlay the SDK's
own close button. Apply window insets so no ad container extends under system bars:

```java
ViewCompat.setOnApplyWindowInsetsListener(binding.adContainerView, (v, insets) -> {
    Insets bars = insets.getInsets(WindowInsetsCompat.Type.systemBars());
    v.setPadding(0, 0, 0, bars.bottom);   // padding on the CONTAINER, never on the AdView
    return insets;
});
```

Declare **"Contains ads"** in the Play Console listing. An undeclared ad-supported app is a
Play policy violation on its own.

---

# Verify with Google's own tool, not guesswork

Native Validator ships in the SDK (**19.2.0+**), is **enabled by default**, and shows an
overlay listing real violations next to each native ad.

Requirements: the device registered as a test device, and test ad unit IDs
(native: `ca-app-pub-3940256099942544/2247696110`).

Disable with:

```xml
<meta-data
    android:name="com.google.android.gms.ads.flag.NATIVE_AD_DEBUGGER_ENABLED"
    android:value="false" />
```

# Audit checklist

Run these before concluding anything.

1. **Overlap** — any FAB, sticky header, snackbar, or overlay above a list carrying ads.
   The most common real cause.
2. **App open trigger point** — splash, or a content screen mid-draw?
3. **App open over banners** — is the resumed screen carrying a banner?
4. **Duplicate ad managers** — more than one registering lifecycle callbacks.
5. **Ad view mutation** — any `setLayoutParams` on an ad view after load.
6. **Container padding** — convert to margin.
7. **Lifecycle forwarding** — every banner host forwards pause/resume/destroy.
8. **Native ad destruction** — `nativeAd.destroy()` on screen death.
9. **Interstitial cadence and trigger** — not on launch/exit, not back-to-back, ≥ 1 per 2 actions.
10. **Rewarded opt-in** — explicit tap, reward only in `onUserEarnedReward`.
11. **Empty and loading states** — ad containers hidden when there is no content.
12. **Test IDs** — no live ad unit reachable from a debug build.
13. **Manual refresh** — no `loadAd()` on a timer, no re-request on every `onResume`.
14. **Reward source** — no in-app currency granted for a non-rewarded impression.
15. **No PII in ad requests** — no user ID, email, or account name in any targeting value.
16. **Ads only from a foreground Activity** — never a Service, receiver, or notification.
17. **System insets** — no ad container under the status, navigation, or gesture bar.
18. **Play listing** — "Contains ads" declared; privacy policy names Google as an ad partner.
19. **app-ads.txt** — published and verified in the AdMob dashboard.

# Known false alarms — do not "fix" these

- **Black bars around a portrait creative.** `FIT_CENTER` inside a fixed-ratio container is
  letterboxing, which **preserves** the aspect ratio. Policy forbids distortion, not
  letterboxing. Ugly, not a violation. Improve it by requesting
  `NATIVE_MEDIA_ASPECT_RATIO_LANDSCAPE` at load time.
- **`MediaView` set to `wrap_content`** to "let the ad size itself" — it can collapse below
  the 120dp minimum and stop video ads from serving.
- **AdChoices position.** Verify on a real device before touching `setAdChoicesPlacement`.
  A low-resolution policy-center screenshot is not evidence.
- **Interstitial on a list-item tap** is a natural transition point and not a violation by
  itself; only frequency and unexpectedness are the risk.
- **IFRAME / floating box / hidden keywords** in the warning text — web boilerplate, never
  applicable to a native app with no WebView.
- **A revenue drop with a clean Policy Center** is usually Publisher Restrictions (D2) or
  seasonal demand, not a hidden violation. Do not refactor working ad code chasing it.
- **Rewarded ads are not "incentivized clicks."** B3 bans paying for ordinary impressions;
  `RewardedAd` is a sanctioned format with its own rules (A8).

# Reporting rule

Rank findings by whether an official quote supports them. Separate:

- **Violation** — backed by a verbatim policy line
- **Risk** — plausible but unquoted
- **Cleanup** — correctness or revenue, not policy

Never present a guess as a confirmed violation. When a screenshot is the only evidence and
it is ambiguous, say so and reach for Native Validator instead.
