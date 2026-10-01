# Vows & Vibe

**See it. Style it. Love it.**

[📱 Download the Android APK](https://github.com/areebaaqeel-dot/vows-vibes/releases/latest/download/vowsvibe-android-debug.apk) · [▶️ Demo video](https://youtube.com/shorts/DTSpsm3_4W4)


## The Problem

Coordinating bridal-party outfits is fragmented across group chats, reference photos and separate shopping decisions. Brides struggle to communicate a consistent color direction, and "sage" or "emerald" can mean very different shades from one store to the next.

Bridesmaids often want the freedom to choose a dress they love, but they can't easily tell whether it fits the bride's palette, how it will look on them, or how it will sit alongside the rest of the party before committing. The result is guesswork, returns and a lot of back-and-forth.

## The Solution

Vows & Vibe brings that process into one Android app. The bride defines the wedding style and color palette, invites each bridesmaid, and reviews everyone together in a shared lineup.

Bridesmaids are free to explore dresses beyond the bride's exact picks. For any dress, the app identifies its color family, suggests related shades that stay within the palette, and uses the CIEDE2000 color-difference formula (which measures color distance the way the human eye perceives it) to show how close the dress is to the bride's chosen colors. Each participant can virtually try on dresses, confirm a look and exchange private suggestions with the bride.

The result is a bridal party that looks coordinated without everyone wearing the same dress, with fewer uncertain purchases and less back-and-forth.

## Try It in 2 Minutes

1. Install the APK above. It's a debug build, so Android may ask you to allow installs from your browser or file manager.
2. Sign in with Google as the **bride** and create a wedding event (date, dress direction, palette).
3. Copy the private **invitation link** the app generates for the event.
4. Open that link on the same or another Android device. It opens in the installed app as a **bridesmaid**, with no separate account needed.
5. Upload a full-body photo, pick a dress and generate a virtual try-on (VTO).

## Bride Workflow

The bride signs in with Google and can:

1. Create and manage one wedding event.
2. Set the date, dress direction, fabric, length and color palette.
3. Add example dresses and share the private invitation link.
4. Upload a full-body photo and capture one guided skin-tone selfie.
5. Generate and save virtual try-on previews.
6. Confirm her final look.
7. Arrange confirmed party members in the lineup canvas and filter by palette relationship.
8. Open a bridesmaid list and enter an individual suggestion conversation.
9. Optionally generate a venue group preview, and save or export the arranged lineup as a PNG.

The bride can also delete the event and all associated participant photos, dresses, previews, messages and lineup data.

## Bridesmaid Workflow

A bridesmaid opens the invitation link, enters her name and receives a private fitting session. She can:

1. Review the bride's event summary and palette.
2. Upload a full-body photo.
3. Capture a guided selfie for personal color guidance based on her undertone.
4. Choose an example dress or upload her own.
5. Generate virtual try-ons and revisit saved previews.
6. Confirm one look for the shared lineup.
7. View the lineup and the bride's saved group preview, if she created one.
8. Exchange suggestions directly with the bride.

Bridesmaids cannot message each other. The bride has a separate conversation with each bridesmaid, and each bridesmaid opens directly into her conversation with the bride.

## Color Guidance

Color guidance is simple and advisory.

- A guided selfie produces a saved skin-tone value and a warm, cool or neutral undertone classification.
- The color guide shows shades that generally complement that undertone and shades that may look less balanced.
- The event summary can show an undertone-aware **Suggested match** beside suitable colors from the bride's palette.

Badges on bridesmaid dress cards:

| Badge | Meaning |
|---|---|
| **Family Match** | The shade coordinates closely with the wedding palette |
| **Related shade** | Same color family, but noticeably lighter, darker or different |
| *(none)* | Neither a Family Match nor a Related shade |

The bride's lineup canvas can filter by **Palette Match**, **Family Match**, **Related shade** and **Different family**.

Skin-tone classification uses CIE Lab conversion, and palette relationships use CIEDE2000 distance where perceptual comparison is useful. Results are styling suggestions, not scientific or mandatory rules: lighting, cameras, fabric and personal preference all still matter.

## Virtual Try-On and Privacy

| Data | Where it goes | Retention |
|---|---|---|
| Full-body photo | Private Supabase Storage bucket, protected by Row Level Security. Short-lived signed URLs are used for repeated try-ons. | Deleted when the event is deleted |
| Guided selfie | Sent directly to Perfect Corp YouCam for skin-color extraction | **Never stored.** Only the derived skin hex, hair and undertone values are kept |
| Try-on previews | `vto_attempts` table | Lets each participant revisit successful previews and confirm one final look |
| Optional venue image and group preview | Sent to Alibaba Cloud Qwen Image, only if the bride generates a group preview | Saved preview is visible to the bridal party |

Confirmed looks receive background-removed cutouts for the shared lineup.

## Suggestions

Suggestions are stored in Supabase and delivered through Supabase Realtime.

- Bride → bridesmaid and bridesmaid → bride are allowed.
- Messages are organized into individual conversations.
- Suggestions stay tied to the recipient's current confirmed look, so stale look-specific messages disappear when that look changes.

## RevenueCat Wedding Pass

The Android app uses RevenueCat for a one-time **Vows & Vibe Pro Wedding Pass**.

- The bride buys the pass once for her wedding. Invited bridesmaids inherit her verified entitlement and never purchase separately.
- RevenueCat customer identity uses the signed-in bride's Supabase user UUID.
- The entitlement ID is `vows_vibes_pro`.
- Entitlements are verified server-side by the backend, using RevenueCat webhooks and verification.

| | Free (bride) | Pro Wedding Pass |
|---|---|---|
| Weddings | 1 | 1 |
| Dress uploads | 2 | Unlimited |
| Successful try-ons | 2 (bride) | 8 per participant |
| Saved skin-tone analyses | 1 (bride) | 1 per participant |
| Suggestions | – | ✅ |
| Lineup saving and export | – | ✅ |
| Group preview | – | ✅ |

Try-on allowances are totals for the wedding and do not reset.

## Shared Lineup and Group Preview

Confirmed participant cutouts appear in a Fabric.js canvas controlled by the bride. She can reposition people, adjust layering, hide participants, filter by palette relationship, save the arrangement and export a PNG.

The **group preview is optional**. The bride can add a venue image and generate a composition with an image-generation model. A saved preview becomes visible to the bridal party.

## Architecture

| Layer | Technology |
|---|---|
| Android app | Capacitor 8 with a bundled Vite/React frontend |
| Supporting backend | Next.js 16 API routes, auth callbacks and invitation endpoints on Vercel |
| Auth, database, storage, realtime | Supabase |
| Purchases and entitlements | RevenueCat Capacitor SDK, plus server-side verification and webhooks |
| Virtual try-on and skin analysis | Perfect Corp YouCam APIs |
| Lineup canvas | Fabric.js |
| Background removal | `@imgly/background-removal-node` |
| Group preview | Alibaba Cloud Model Studio / Qwen Image |

Other app features: Android App Links open invitations directly in the installed app, and lineup state is saved between sessions.

## Repository Map

```text
src/                          # Supporting Next.js backend and shared application code
├── app/api/                 # Deployed events, VTO, billing and webhook endpoints
├── components/              # Shared bride, bridesmaid, lineup and conversation UI
└── lib/
    ├── billing/             # RevenueCat verification and cached entitlement state
    ├── color/               # Undertone guide and palette relationships
    ├── cutout/              # Confirmed-look background removal
    ├── suggestions/         # Bride/bridesmaid conversation permissions
    ├── supabase/            # Server and browser Supabase clients
    └── youcam/              # Perfect Corp integration

mobile/                       # Android application
├── android/                 # Native Android project
├── src/                     # Bundled mobile application and native integrations
└── capacitor.config.ts      # Android app configuration

supabase/
├── migrations/              # RevenueCat and VTO usage-tracking migrations
└── schema.sql               # Complete database schema
```