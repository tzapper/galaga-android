# Galaga Replica — Android Project

A Capacitor-wrapped Android build of the canvas/JS Galaga tribute game.
The game itself lives at `www/index.html` and runs inside a native
WebView shell — same game logic as the browser version, plus:

- Hardware **back button** pauses the game (exits from the start/game-over screen)
- Immersive dark status bar matching the game's palette
- Portrait orientation locked
- **Fully offline** — no internet permission, no network calls, no external
  font/CDN dependency. The app doesn't request any Android permissions at all.

## Build it from your phone (GitHub Actions — no Android Studio)

This project includes `.github/workflows/build-apk.yml`, which builds a
debug APK on GitHub's servers every time you push. You just need to get
this folder into a GitHub repo from your phone, which is the one fiddly
part — mobile browsers don't reliably let you drag-and-drop a whole
folder into GitHub's web uploader. The reliable way is **Termux** (a
terminal app) + `git`.

**1. Install Termux**
Get it from F-Droid (the Play Store version is outdated and unmaintained):
`f-droid.org` → search Termux → install.

**2. Set up Termux**
```bash
pkg update && pkg upgrade -y
pkg install git -y
termux-setup-storage        # grants access to your Downloads folder
```

**3. Get the project into Termux**
Unzip `galaga-android.zip` (download it from this chat first, it'll land
in `~/storage/downloads/` or wherever your browser saves files), then:
```bash
cd ~
cp ~/storage/downloads/galaga-android.zip .
unzip galaga-android.zip
cd galaga-android
```

**4. Create an empty repo on GitHub**
On github.com (mobile browser is fine): **+ → New repository** → name it
`galaga-android` → **don't** initialize with a README → Create.

**5. Create a Personal Access Token** (GitHub no longer accepts your
password for git pushes)
GitHub → your profile picture → **Settings → Developer settings →
Personal access tokens → Tokens (classic) → Generate new token** → tick
`repo` scope → Generate → **copy the token now**, you won't see it again.

**6. Push from Termux**
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<your-username>/galaga-android.git
git push -u origin main
```
When it asks for a username, enter your GitHub username. When it asks for
a password, paste the **token** from step 5, not your account password.

**7. Watch it build**
Go to your repo on github.com → **Actions** tab. You'll see "Build
Android APK" running (takes a few minutes). When it's green, click into
the run → under **Artifacts** at the bottom, tap `galaga-replica-debug-apk`
to download a zip containing `app-debug.apk`.

**8. Install it**
Unzip on your phone, tap the `.apk` file, allow "install from unknown
sources" if prompted. It's an unsigned debug build, which is fine for
your own device but not for distributing to others or the Play Store —
see the note below if you eventually want a signed release build.

If you'd rather trigger a fresh build without pushing new code, use the
**Run workflow** button on the Actions tab (`workflow_dispatch` is
enabled for this).

## Build it on a desktop instead (Android Studio)

**Requirements:** Android Studio (Hedgehog or newer recommended), JDK 17,
Android SDK Platform 34+ — Android Studio will prompt to install anything
missing.

1. Unzip this project anywhere on your machine.
2. Open **Android Studio** → `Open` → point directly at the `android/`
   subfolder — that's the real Gradle project.
3. Let Gradle sync (first sync downloads dependencies — needs internet).
4. Plug in your phone (USB debugging on) or start an emulator.
5. Click **Run ▶** — it'll install and launch on your device.

To build a release APK instead: **Build → Generate Signed Bundle / APK →
APK**, create/select a keystore, and build the release variant. The output
`.apk` will be under `android/app/build/outputs/apk/`.

## If you change the game code

Edit `www/index.html` (or split it up as you like), then from the project
root:

```bash
npm install        # only needed once
npx cap sync android
```

This recopies the web assets into `android/app/src/main/assets/public`
and re-links any native plugins. Re-run the app from Android Studio
afterward.

## Project layout

```
galaga-android/
├── www/index.html          # the actual game (HTML/CSS/JS, canvas + Web Audio)
├── capacitor.config.ts     # app id, name, splash/background color
├── android/                 # the real Android Studio / Gradle project
│   └── app/src/main/assets/public/  # synced copy of www/ — don't hand-edit
└── package.json             # Capacitor CLI + plugin deps (@capacitor/app, @capacitor/status-bar)
```

App ID: `com.tapan.galagareplica` — change it in `capacitor.config.ts` and
re-run `npx cap sync android` before your first build if you want a
different package name.

## Debug vs. release builds

The Actions workflow builds a **debug APK** — installable on your own
device immediately, no signing setup required. If you later want a
**signed release APK** (smaller, optimized, installable without a
"debug" warning, required for Play Store), that needs a keystore file
and a couple of secrets added to the GitHub repo (Settings → Secrets and
variables → Actions). Ask and I can add a `assembleRelease` job and the
signing config for that when you're ready.

## Notes

- This is an original-code tribute inspired by the 1981 arcade game's
  mechanics (formation flight, dive attacks, wave progression) — not
  affiliated with or using assets from Namco/Bandai Namco.
- No native plugins beyond `@capacitor/app` (back button) and
  `@capacitor/status-bar` (immersive theming) are used, so the project
  is easy to extend — add plugins with `npm install @capacitor/<name>`
  then `npx cap sync android`.
