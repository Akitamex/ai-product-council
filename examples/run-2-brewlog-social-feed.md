# Council run: social feed and SocialKit SDK for Brewlog

A complete, lightly edited transcript of a council run. The product is fictional: **Brewlog**, an indie iOS coffee-brewing journal (one developer, ~12k MAU, $2.99/mo subscription, all data stored on-device with no accounts).

**Invocation:**

```
/product-council I want to add a social feed so users can share brews and
follow friends. found this free SDK called SocialKit that handles the feed,
CDN for photos, and contact matching for finding friends. could ship in a
week. good idea?
```

---

## Mediator: scope + selection

**Scope: Large.** The plan introduces user accounts (Brewlog has none today), uploads brew photos to a third-party CDN, and matches device contacts against an external service. Each of these independently crosses the product's core declared boundary: "your data stays on your device." The dependency is on the critical path, not optional, and the change is effectively irreversible once users accumulate social graphs and uploaded photos on SocialKit's infrastructure.

**Spawning:** council-product (strategy pivot disguised as a feature, worth-it judgment), council-engineering (on-device invariant, real SDK cost, week estimate), council-risk (contact-matching and photo upload break the product's stated privacy boundary: veto armed), council-design (feed colonizes a minimal single-purpose app structure).

**Skipping:** council-voice. No user-facing copy exists yet to review, and the plan does not survive to the naming stage: the risk and architecture questions must resolve first. If the council approves a trimmed version, voice reviews the share-card text at that point.

---

## Persona reviews

### Product & Scope

**Verdict (worth building?):** no-as-proposed. A social feed is a strategy pivot from journal to network. Those are different products with different growth mechanics, different churn profiles, and different failure modes. Brewlog has 12k MAU paying $2.99 for a calm private journaling experience. The evidence base for "they want a feed" is absent from the invocation.

**Smallest honest version:** share-as-image export. A user taps Share on a brew entry, the app renders a watermarked card (beans, ratio, notes, date) as a PNG, and standard iOS share sheet handles distribution. No accounts, no feed, no external dependency, no new data boundary. Users who want to share to Instagram or a group chat can do so today if the feature existed. This version is buildable in two days and reversible.

**Concerns:** the graveyard of indie social feeds is long. Building a feed on 12k MAU means moderating it, hosting it, and watching it atrophy if engagement is thin. Brewlog's retention comes from the journaling habit, not from social proof. Adding social-proof UI (follower counts, likes) to a habit-tracking app typically pressures the calmer users who contribute the most subscription revenue. Indie apps that have tried this split path rarely report that the social layer paid for itself.

**Asks (must-have to ship the reduced version):** define what success for share-export looks like before building the feed (shares-per-active-user rate, inbound conversion from share cards, 60-day measurement window before any social-layer decision).

**Non-blocking suggestions:** add a share counter in the app's local stats screen so the developer can see how often users tap share before committing to the social layer.

---

### Engineering

**Feasibility:** unsound as proposed.

**Real cost:** "on-device, no accounts" is not a configuration; it is the load-bearing architecture of the app. The entire data model (CoreData local store, no sync, no user identity) assumes no network writes. Adding a social feed requires: a user identity and auth layer (Apple Sign In or equivalent), a server-side social graph, a moderation surface, media upload and processing pipeline, feed ranking or chronological pagination, push notifications for follows and likes, and a migration path for the 12k existing users who currently have no accounts. SocialKit bundles some of this, which does not reduce the cost: it moves it off-device to a dependency the developer does not control. The real cost is two to four months of calendar time for a single developer, not one week.

**Invariants at risk:** the on-device promise is the product's core invariant. Every brew entry (beans sourced, dose, ratio, tasting notes, timestamps) currently never leaves the device. SocialKit's CDN upload inherently moves photo data to external servers. Contact-matching inherently uploads the device address book to SocialKit's infrastructure during the friend-discovery call. Both are permanent: data sent to an external service cannot be recalled.

**"Free SDK" cost structure:** SocialKit's CDN and social graph are free at low volume. At 12k MAU with photo uploads, the developer crosses into paid tiers within weeks of launch. Pricing flips are a well-documented pattern for third-party social infrastructure: the developer takes the adoption dependency, then the vendor reprices. The switching cost after users have uploaded photos is high.

**Asks (must-have to ship the reduced version):** share-as-image export requires no new dependency. Render a UIGraphicsImageRenderer card from existing local data, pass to UIActivityViewController. Two source files touched, no migration, no server.

**Non-blocking suggestions:** if a social layer is revisited, build on Apple's CloudKit (developer already has the entitlement, data stays under the user's iCloud account, no third-party data-sharing) rather than a third-party SDK.

---

### Risk

**Veto status:** VETO: the plan inherently uploads two categories of personal data to a third-party service without informed consent, breaking the product's stated privacy boundary.

**Data flows created or changed:**

1. **Contact-matching:** SocialKit's friend-discovery call uploads the user's device address book (names, phone numbers, email addresses of people who have not consented to anything) to SocialKit's servers. This is not a sloppy-implementation risk: it is how contact-matching works. The user has not been told this happens; the contacts certainly have not.

2. **Photo CDN upload:** brew photos leave the device and are stored on SocialKit's CDN. Photos from cafes or home kitchens contain EXIF metadata including GPS coordinates, camera model, and timestamp. The App Store page for Brewlog states "your data stays on your device." Uploading photos to an external CDN makes that statement false the moment the feature ships.

**Irreversible steps:** once a user's address book has been sent to SocialKit, that upload cannot be undone. Once photos with EXIF are on SocialKit's CDN, their metadata is accessible to SocialKit under SocialKit's own terms of service, which the Brewlog developer cannot revoke. Social graphs, once formed, produce expectations of continuity: removing the feature after adoption causes user-facing harm.

**Platform-policy exposure:** App Store privacy labels require disclosure of data linked to the user and data linked to the device. Adding contact-matching and CDN photo upload without updating the privacy label triggers App Store Review policy violations. Apple's guidelines additionally require explicit user permission before accessing contacts (NSContactsUsageDescription) and explicit disclosure before uploading photos off-device. Consent UI can be added, but it cannot make the contact-upload acceptable for users whose contacts did not consent.

**What a non-vetoed version requires:** the plan must remove both offending data flows entirely. No contact-matching against an external service. No automatic photo upload to a third-party CDN. A non-vetoed social feature either (a) keeps all data on-device and uses local QR-code or AirDrop sharing between known contacts, or (b) stores data in the user's own iCloud via CloudKit, where Apple's privacy framework governs the boundary and the App Store page claim remains true. SocialKit in its current role cannot satisfy these conditions.

**Asks (must-have to unblock):** remove contact-matching entirely from the plan. Remove SocialKit CDN upload from the plan. Update the App Store privacy label to match whatever data flows remain. Any photo sharing must be export-only from the device, not upload-to-CDN.

**Non-blocking suggestions:** publish a one-paragraph privacy statement on the App Store page and in-app settings before any social feature ships; the existing "on-device" language is a commitment that now needs active defense.

---

### Design & Craft

**Coherence verdict:** the social feed breaks the product's established minimal character, regardless of implementation.

**Concerns:** Brewlog's current structure is a single-purpose quiet tool: a list of brews, a detail view, a log form. There are no tabs, no notification badges, no follower counts. A social feed requires a tab bar (at minimum: Journal, Feed, Profile), a notification layer, and social-proof elements (likes, follower counts, comment counts). Each of these is individually reasonable in a social app. Together they transform the app's identity from "private journal" to "social network for coffee people." The users who pay $2.99/mo for calm habit tracking are not the same users who want to compete on brew aesthetics in a feed. Introducing social-proof UI typically pressures the quieter users toward performative behavior, which degrades the journaling habit.

**Share-as-image keeps the craft:** a well-designed share card (bean name, ratio, brew method, tasting note excerpt, a minimal Brewlog wordmark) is a design artifact the user is proud to post elsewhere. It extends the product's aesthetic into social contexts without bringing social mechanics into the product. The card surface is also the one place where typography, color palette, and layout reinforce the brand without requiring the developer to build and maintain a full design system for a social layer.

**Asks (must-have to ship the share card):** the card must use the app's existing type scale and color tokens, not introduce new ones. Export at 2x resolution minimum. The watermark should read "brewed with Brewlog" in the app's body typeface, set small, placed consistently (bottom-right corner works).

**Non-blocking suggestions:** offer two card layouts (minimal text-only, photo-with-notes) so users whose photos are good can lead with the image and users who prefer privacy can share text only without EXIF exposure.

---

## Mediator: synthesis

**Alignment:** all four personas converged on the same reduction before seeing each other's reviews. Share-as-image export serves the genuine user desire (showing off a good brew) without accounts, without a feed, without an external SDK, and without violating the product's stated privacy boundary. No persona defended the SocialKit plan or the one-week timeline as stated.

**Conflicts:**

1. **Product growth wish vs the on-device identity.** Product flags that 12k MAU is thin for a network and the graveyard of indie social apps is evidence, but it acknowledges that the developer may have growth ambitions the data does not yet reflect. If the developer wants to build a social product, that is a legitimate goal: it requires a deliberate pivot decision, a new privacy story, and a timeline measured in months, not a week. This is a strategy ruling, and it goes to the human.

**Veto status: VETO stands.** Risk returned a hard veto on two concrete data flows the plan inherently creates: contact-matching uploads the device address book to SocialKit without consent from the contacts, and photo CDN upload contradicts the App Store listing's "your data stays on your device" statement. These are not implementation risks: they are what the plan does by design. The veto blocks approval of the SocialKit-based plan.

**The two legitimate outs for the veto:**

- **Revise the plan:** remove contact-matching and CDN photo upload entirely. A version that keeps all data on-device (share-card export, local QR pairing, or CloudKit-backed sync under the user's own Apple ID) satisfies the veto's conditions. Risk's asks become checkable guards for that revised plan.
- **Explicit on-the-record override:** the developer may override the veto by stating, on the record here, that they intend to change the product's privacy posture, update the App Store privacy label, add consent flows, and accept the SocialKit dependency pricing risk. The override does not make the risks disappear: it records that the developer made the call knowingly.

**Open questions for the human:**

1. (Veto, first.) Do you accept the veto, revise the plan to remove the data leaks, and ship share-as-image export? Or do you intend to override, accept the privacy-posture change, and pursue the full social layer with proper consent and updated App Store disclosure?
2. If you accept the reduction: is the 60-day share-rate measurement window acceptable before revisiting the social-layer question?

---

## Human rulings (recorded verbatim)

> 1. veto accepted. SocialKit is out, contact-matching is out. share card only for now,
>    keep the on-device promise intact
> 2. 60 days is fine. if share-card usage is strong, i'll do the CloudKit route properly
> 3. can you design the watermarked share card next? want to see what it looks like

---

## Final plan (excerpt from the decision record)

1. Implement share-as-image export using `UIGraphicsImageRenderer` from existing local brew data. No new dependency, no network calls.
2. Card layout: bean name, brew method, dose/ratio, one-line tasting note excerpt, date. Two variants: text-only and photo-with-notes (photo sourced from local camera roll, not uploaded anywhere).
3. Watermark: "brewed with Brewlog" in existing body typeface, small, bottom-right. Export at 2x resolution minimum.
4. SocialKit: not integrated. Remove from consideration until a deliberate social-pivot decision is made and documented separately.
5. Metric: shares-per-active-user rate logged locally via an in-app counter. 60-day measurement window starting at release. Checkpoint by 2026-08-18: expand toward CloudKit-backed social or hold at export.
6. App Store privacy label: no update required (no new data flows off-device). Review label accuracy before the checkpoint if CloudKit path is chosen.

Persisted to `_decisions/2026-06-18-share-not-feed.md`.

---

*What to notice: the veto did not block the council, it blocked the specific plan. Risk named two concrete data flows (contact-matching, CDN photo upload) that the proposal inherently creates, not hypothetical failures of sloppy code. The human's override path stayed open and was documented explicitly, but unused. The council ran to a complete resolution with a persisted decision record in under eight minutes.*
