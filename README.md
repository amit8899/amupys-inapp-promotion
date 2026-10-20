# Amupys In-App Promotion System

A lightweight, server-driven cross-promotion system for Android applications. It enables real-time promotion of your newly published Google Play Store apps across existing apps without requiring app updates.

---

## 🚀 How It Works

1. **Remote Control via GitHub**: 
   The client app asynchronously fetches `promotion.json` from this repository.
2. **Instant Kill-Switch (`toshow`)**: 
   Set `"toshow": false` to instantly hide the promotion globally across all active users.
3. **Smart & Non-Intrusive Frequency Capping**:
   - **Daily Limit ("1 Promotion per Day Only")**: Enforces a strict maximum of 1 promotion popup per day using local date tracking (`yyyyMMdd`).
   - **Total Duration Cap (`maxDays`)**: Limits promotion visibility to a maximum number of days (e.g., `maxDays: 10`). Once the user has been shown the promotion on 10 distinct days, the popup is permanently stopped.
   - **Cooldown Timer (`cooldown_hours`)**: Ensures at least 24 hours between popup impressions.
   - **Randomized Display (`show_chance_percentage`)**: Even on eligible days, a random roll (e.g., 50% probability) prevents showing the dialog on every single session, keeping the user experience clean.
   - **Install Conversion Tracking**: Once a user taps "Install", the app records the package ID and never displays that promotion again.
4. **Dedicated "More Apps" View in Settings / About**:
   - In addition to dialog popups, a native "MORE APPS" card is displayed in the **About** (`MoreFragment`) tab whenever `"toshow": true`.
   - Displays the promoted app's logo, title, description, badge, and a direct "GET" action button.
5. **Offline Resilience & Fast Performance**:
   - The config is cached in Android `SharedPreferences`.
   - Network calls run on background threads using OkHttp and never block the main UI thread.

---

## 📄 JSON Configuration Schema (`promotion.json`)

```json
{
  "toshow": true,
  "app_id": "com.amupys.socialbuddy",
  "badge": "FEATURED APP",
  "title": "Social Media Downloader",
  "subtitle": "Save Reels, Shorts & HD Videos",
  "rating": "4.8 ★  •  Free",
  "short_description": "Easily download HD videos, reels, and stories from Instagram, Facebook, TikTok, X (Twitter), and WhatsApp status with one tap. Fast, free, and watermark-free!",
  "icon_url": "https://amupys.amupys-web.workers.dev/assets/apps/social/social_logo_512.jpg",
  "app_icon_url": "https://amupys.amupys-web.workers.dev/assets/apps/social/social_logo_512.jpg",
  "features": [
    "⚡ 3x Faster Video Downloads",
    "🎬 Save in Full HD & 4K with Audio",
    "✨ 100% Free & No Watermark"
  ],
  "button_text": "Install Free on Play Store",
  "playstore_link": "https://play.google.com/store/apps/details?id=com.amupys.socialdownloader",
  "cancel_text": "Maybe Later",
  "maxDays": 10,
  "cooldown_hours": 24,
  "show_chance_percentage": 50
}
```

### Field Definitions

| Field | Type | Description |
|---|---|---|
| `toshow` | `boolean` | **Master Toggle**. `true` to enable dialog and More Apps view, `false` to disable. |
| `app_id` | `string` | Google Play package name (e.g. `com.amupys.socialbuddy`). Used to launch the Play Store and track user conversion. |
| `maxDays` | `int` | Maximum number of days the promotion dialog can be shown to a user (e.g. `10`). Defaults to `10`. |
| `badge` | `string` | Top highlight badge (e.g. `"FEATURED APP"`, `"NEW RELEASE"`). |
| `title` | `string` | App title displayed in the dialog header and More Apps section. |
| `subtitle` | `string` | Catchy slogan or short subtitle. |
| `rating` | `string` | Social proof indicator (e.g. `"4.8 ★  •  Free"`). |
| `short_description` | `string` | 1–2 sentence summary highlighting app benefits. |
| `icon_url` / `app_icon_url` | `string` | Direct image URL for the app icon (dynamically loaded via Glide with fallback). |
| `features` | `array` | Up to 3 feature bullet points. |
| `button_text` | `string` | Call-To-Action (CTA) button label. |
| `playstore_link` | `string` | Web URL fallback if Play Store app cannot be opened directly. |
| `cancel_text` | `string` | Dismiss button label (e.g. `"Maybe Later"`). |
| `cooldown_hours` | `int` | Minimum hours to wait between impressions (default `24`). |
| `show_chance_percentage` | `int` | Random chance percentage (`1–100`) to show when eligible (default `50`). |

---

## 🛠️ Android Client Integration

### 1. Raw Endpoint
```java
public static final String PROMOTION_JSON_URL = 
    "https://raw.githubusercontent.com/amit8899/amupys-inapp-promotion/main/promotion.json";
```

### 2. Trigger Points

#### On App Launch (`MainActivity.java`)
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // ...
    PromotionManager.maybeShowOnAppOpen(this);
}
```

#### On User Action / Download (`BottomSheetLO.java`)
```java
// Called when user enqueues a download
PromotionManager.maybeShowOnDownload(hostActivity);
```

#### More Apps Section (`MoreFragment.java`)
```java
// Dynamically binds and displays the promoted app row in the About tab
PromotionManager.setupMoreAppsView(view, getActivity());
```

---

## 🔄 Updating Promotions

To promote a new application or update the content:
1. Edit `promotion.json` in this repository.
2. Commit the changes to the `main` branch.
3. Active user devices will fetch the updated configuration on their next app session.
