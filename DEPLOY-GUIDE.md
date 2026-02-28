# NEXUS — PWA & Android Deployment Guide

## What's In This Package

```
nexus-pwa/
├── index.html          ← The game (production-ready PWA)
├── manifest.json       ← PWA manifest (app name, icons, theme)
├── sw.js               ← Service worker (offline support)
├── favicon.ico         ← Browser favicon
├── icons/              ← All app icons (72px to 512px + maskable)
│   ├── icon-72x72.png
│   ├── icon-96x96.png
│   ├── icon-128x128.png
│   ├── icon-144x144.png
│   ├── icon-152x152.png
│   ├── icon-192x192.png
│   ├── icon-384x384.png
│   ├── icon-512x512.png
│   └── maskable-icon-512x512.png
└── .well-known/
    └── assetlinks.json ← Android TWA verification (edit before deploy)
```

---

## STEP 1: Deploy the PWA (GitHub Pages)

### Option A: Use your existing GitHub account
1. Create a new repo called `nexus`
2. Upload ALL files from this package (maintaining folder structure)
3. Go to Settings → Pages → Deploy from `main` branch
4. Your game is live at: `https://yourusername.github.io/nexus/`

### Option B: Custom domain (when you get one)
1. In your repo Settings → Pages → Custom domain, enter your domain
2. Add a CNAME record with your domain registrar pointing to `yourusername.github.io`
3. GitHub will auto-provision HTTPS

### Verify PWA works:
1. Open the URL on your phone's Chrome
2. You should see "Add to Home Screen" prompt
3. The game should work offline after first visit
4. Check Chrome DevTools → Application → Manifest (should show all icons)

---

## STEP 2: Wrap as Android App (TWA)

TWA (Trusted Web Activity) wraps your PWA as a native Android app with
full-screen experience, no browser UI. This is how you get on Google Play.

### Option A: Bubblewrap (Recommended — easiest)

Bubblewrap is Google's official CLI tool for creating TWAs.

```bash
# Install Node.js if you don't have it
# Then install Bubblewrap:
npm install -g @nicedoc/bubblewrap

# Initialize your TWA project:
mkdir nexus-android && cd nexus-android
bubblewrap init --manifest https://YOUR-DOMAIN/manifest.json

# It will ask you:
# - Package name: io.nexus.game
# - App name: NEXUS
# - Launcher name: NEXUS
# - Theme color: #0a0a12
# - Background color: #0a0a12
# - Start URL: https://YOUR-DOMAIN/index.html
# - Icon: (it pulls from manifest)
# - Signing key: Create new (save the password!)

# Build the APK:
bubblewrap build

# This creates:
# - app-release-bundle.aab (upload to Google Play)
# - app-release-signed.apk (for testing)
```

### Option B: PWABuilder (No-code — browser-based)

1. Go to https://www.pwabuilder.com/
2. Enter your PWA URL
3. Click "Package for stores" → Android
4. Download the generated Android package
5. It gives you a .aab file ready for Google Play

### Option C: Capacitor (More control, if you want native features later)

```bash
npm init -y
npm install @capacitor/core @capacitor/cli
npx cap init NEXUS io.nexus.game --web-dir .

# Copy your PWA files into the project root
npx cap add android
npx cap sync
npx cap open android  # Opens Android Studio
# Build → Generate Signed Bundle/APK
```

---

## STEP 3: Upload to Google Play

### Prerequisites
- Google Play Developer account (you already have this)
- Signed .aab file from Step 2

### In Google Play Console:

1. **Create App**
   - App name: `NEXUS — Discover. Adapt. Evolve.`
   - Default language: English (US)
   - App or game: Game
   - Free or paid: Free

2. **Store Listing**
   - Short description (80 chars max):
     `Discover hidden rules. Place seeds. Watch them evolve. Outscore evolution.`
   - Full description:
     ```
     NEXUS is a pattern-discovery puzzle game where you place seeds on a
     grid and watch them evolve according to hidden rules you must discover
     through observation.

     🧬 DISCOVER — Each round has a hidden mutation rule. Watch how cells
     live, die, and spawn to figure out the pattern.

     ⚡ ADAPT — Rules change every round. Memorization won't save you.
     Only adaptive thinking wins.

     🔥 EVOLVE — Build combos, earn power-ups, and climb the difficulty
     ladder from Novice to Nexus.

     Features:
     • 12 unique mutation rules with escalating difficulty
     • Daily challenges — same puzzle for everyone, compare scores
     • Combo system — consecutive survivals multiply your score
     • Power-ups: Peek (reveal rules), Shield (protect cells), Extra Seeds
     • Streak tracking — how many days in a row can you play?
     • Share your score grid to social media
     • Works offline — play anywhere
     • No ads in core gameplay

     Easy to learn. Impossible to master. One more round...
     ```

3. **Categorization**
   - Category: Puzzle
   - Tags: puzzle, strategy, brain, cellular automata, pattern

4. **Graphics Assets**
   - Icon: Use `icon-512x512.png` from icons folder
   - Feature graphic: 1024x500px (I can generate this next)
   - Screenshots: At least 2 phone screenshots (I can generate these next)

5. **Content Rating**
   - Fill out the IARC questionnaire
   - This game should get: Everyone / PEGI 3

6. **App Release**
   - Go to Production → Create new release
   - Upload the .aab file
   - Add release notes: "Initial release"
   - Review and roll out

---

## STEP 4: Digital Asset Links (Required for TWA)

For the TWA to show fullscreen (no browser bar), you need to verify
domain ownership:

1. Get your signing key SHA-256 fingerprint:
   ```bash
   keytool -list -v -keystore your-keystore.jks -alias your-alias
   ```
   Copy the SHA-256 fingerprint.

2. Edit `.well-known/assetlinks.json` in your web repo:
   Replace `REPLACE_WITH_YOUR_SHA256_FINGERPRINT` with your actual fingerprint.

3. Push to GitHub. The file must be accessible at:
   `https://YOUR-DOMAIN/.well-known/assetlinks.json`

---

## STEP 5: Update manifest.json paths (if using custom domain)

If you deploy to a subdirectory (like GitHub Pages), update `start_url` in
manifest.json:
- GitHub Pages: `"start_url": "/nexus/index.html"`
- Custom domain: `"start_url": "/index.html"`

---

## Timeline

| Task | Time | Status |
|------|------|--------|
| PWA files ready | — | ✅ Done |
| Deploy to GitHub Pages | 10 min | ⬜ You |
| Test PWA on phone | 5 min | ⬜ You |
| Wrap with PWABuilder or Bubblewrap | 20 min | ⬜ You |
| Create Google Play listing | 30 min | ⬜ You |
| Upload .aab and submit | 10 min | ⬜ You |
| Google review (typically) | 1-3 days | ⬜ Google |

---

## Quick Start (Fastest Path)

1. Push this entire `nexus-pwa` folder to a GitHub repo
2. Enable GitHub Pages
3. Go to https://www.pwabuilder.com → enter your URL → download Android package
4. Upload to Google Play Console
5. Done. You're on the Play Store.
