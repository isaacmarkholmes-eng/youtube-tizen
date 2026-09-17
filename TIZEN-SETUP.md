# Video Lounge — Samsung TV (Tizen) install guide

This folder is a **thin launcher app** that adds a **Video Lounge** icon to your Samsung TV home screen. The app opens your hosted Video Lounge site (Netlify URL) full-screen.

**Cost:** $0 for personal use on your own TV (free Samsung account + free certificates).

---

## What you need

| Item | Notes |
|------|--------|
| Windows PC | Tizen Studio is Windows/macOS/Linux; this guide uses Windows paths |
| Samsung Smart TV | Same Wi‑Fi as your PC |
| Deployed Video Lounge URL | e.g. `https://your-site.netlify.app` |
| Samsung account | Free at [developer.samsung.com](https://developer.samsung.com) |

---

## Step 1 — Deploy Video Lounge (if not already)

1. Push `custom-youtube-player` to GitHub (or zip the folder).
2. Connect the repo to [Netlify](https://www.netlify.com/) (drag-and-drop works too).
3. Note your live URL, e.g. `https://video-lounge-abc123.netlify.app`.

---

## Step 2 — Set the URL in this project

Edit `tizen/config.js`:

```javascript
window.VIDEO_LOUNGE_URL = "https://your-actual-site.netlify.app";
```

---

## Step 3 — Add an app icon

Tizen requires `tizen/icon.png`.

1. Create a **117×117** PNG (square logo or letter “VL”).
2. Save it as `tizen/icon.png`.

Tip: Any simple square PNG works for personal sideloading.

---

## Step 4 — Samsung Developer account

1. Go to [developer.samsung.com](https://developer.samsung.com).
2. Sign up / sign in (free).
3. Accept the developer terms.

You do **not** need TV Seller Office or a paid plan for sideloading to your own TV.

---

## Step 5 — Enable Developer Mode on the TV

1. On the TV remote, open **Apps**.
2. Press **1 → 2 → 3 → 4 → 5** quickly (while Apps is open).
3. **Developer Mode** panel appears.
4. Turn **Developer mode** **On**.
5. Enter your PC’s IP if prompted (optional for some steps).
6. Click **OK** — the TV reboots.
7. After reboot, open Developer Mode again and note your **DUID** (device ID). You need this for signing.

Also enable **Developer Mode** app from Apps if it appears separately.

---

## Step 6 — Install Tizen Studio

1. Download **Tizen Studio** with **TV SDK** from:  
   [Samsung — Installing TV SDK](https://developer.samsung.com/smarttv/develop/getting-started/setting-up-sdk/installing-tv-sdk.html)
2. Run the installer.
3. In **Package Manager**, install:
   - **TV Extensions** (Samsung TV)
   - **Samsung Certificate Extension** (required for modern TVs)
   - **TV Emulator** (optional; real TV testing is better)

Default install path (Windows):

```text
C:\tizen-studio
```

---

## Step 7 — Import this project

1. Open **Tizen Studio**.
2. **File → Import → Tizen → Tizen Web Project**.
3. Choose **Archive File** or **Select root directory**.
4. Point to this folder: `custom-youtube-player/tizen`
5. Project name: **VideoLounge** → Finish.

If import fails, use **File → New → Tizen Web Project → TV → Samsung TV**:

- Template: **Basic UI**
- Copy `index.html`, `config.js`, and `config.xml` from this folder over the generated files.

---

## Step 8 — Create a signing certificate (one-time)

Modern Samsung TVs **reject** generic Tizen certs. You need a **Samsung certificate** tied to your TV’s **DUID**.

1. **Tools → Certificate Manager** (or **Tizen Studio → Tools → Certificate Manager**).
2. Click **+** to create a profile → **Samsung** → **TV**.
3. **Author certificate:** Create new → sign in with Samsung account → save password somewhere safe.
4. **Distributor certificate:** Create new → **Public** privilege (enough for a URL launcher).
5. Add your TV **DUID** from Step 5 (click **+** next to connected device, or paste manually).
6. Finish and set this profile as **Active**.

Back up the `.p12` / profile files Samsung creates — you need the **same author cert** to publish updates later.

---

## Step 9 — Connect the TV

1. PC and TV on the **same network**.
2. On TV: Developer Mode → note IP address if shown.
3. In Tizen Studio: **Remote Device Manager** (or **Connection Explorer**).
4. **Scan** or **Add** device → enter TV IP → **Connect**.

If connection fails:

- Temporarily disable PC firewall for private network
- Reboot TV after enabling Developer Mode
- Confirm TV firmware is up to date

---

## Step 10 — Build and install

### Option A — Tizen Studio (easiest)

1. Right-click **VideoLounge** project → **Run As → Tizen Web Application**.
2. Select your **TV** as target.
3. Tizen Studio builds, signs, and installs. The app appears under **Apps**.

### Option B — Command line

```powershell
cd C:\Repo\custom-youtube-player\tizen

# Set active certificate profile name (from Certificate Manager)
$profile = "YourProfileName"

& "C:\tizen-studio\tools\ide\bin\tizen.bat" build-web -- .
& "C:\tizen-studio\tools\ide\bin\tizen.bat" package -t wgt -s $profile -- .

# Install (TV IP from Developer Mode panel)
& "C:\tizen-studio\tools\sdb.exe" connect 192.168.1.50
& "C:\tizen-studio\tools\sdb.exe" install VideoLounge.wgt
```

The `.wgt` file is created in the project folder or `.buildResult`.

---

## Step 11 — Launch from home screen

1. Open **Apps** on the TV.
2. Find **Video Lounge**.
3. Optionally: **Hold/select app → Add to Home** (wording varies by TV year).

The app redirects to your Netlify URL. Smart shuffle, history, and filters work as in the browser.

---

## Updating the app

When you change `config.js` or bump version in `config.xml`:

1. Increase `version="1.0.1"` in `config.xml`.
2. Rebuild and reinstall with the **same author certificate**.
3. Or use **Run As** again from Tizen Studio.

Site-only changes (no Tizen changes): just redeploy Netlify — the TV app picks them up on next launch.

---

## Certificate renewal

Samsung distributor certs **expire** (often ~1 year). When install fails with certificate errors:

1. Open **Certificate Manager**.
2. Renew or recreate distributor cert (same author cert if possible).
3. Rebuild `.wgt` and reinstall.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `1010 App install fail` | Wrong/expired cert, or DUID not in distributor cert |
| `You are not an authorized user` | Use **Samsung** cert, not plain Tizen cert (2023+ TVs) |
| Blank screen after launch | Check `VIDEO_LOUNGE_URL` in `config.js`; open URL in TV browser first |
| YouTube won’t play | Same embed restrictions as browser; video must allow embedding |
| Can’t connect to TV | Same Wi‑Fi, Developer Mode on, try `sdb connect TV_IP` |
| App not on home screen | Pin from Apps list; some models require “Add to Home” manually |

---

## Optional — TV-friendly tweaks on the website

For better remote control on TV, consider later on the main site:

- Larger focus outlines on buttons
- Bigger text for 10-foot viewing
- `Enter` key on focused grid items

The Tizen wrapper does not require those to work.

---

## Files in this folder

| File | Purpose |
|------|---------|
| `config.xml` | Tizen app manifest (name, icon, TV profile) |
| `index.html` | Redirects to your hosted Video Lounge |
| `config.js` | **Your Netlify URL** — edit this |
| `icon.png` | Home screen icon (you add this) |
| `.project` / `.tproject` | Tizen Studio project metadata |

---

## Quick checklist

- [ ] Video Lounge live on Netlify
- [ ] `tizen/config.js` updated with real URL
- [ ] `tizen/icon.png` added (117×117)
- [ ] Samsung developer account created
- [ ] TV Developer Mode on, DUID copied
- [ ] Tizen Studio + Samsung Certificate Extension installed
- [ ] Certificate profile with TV DUID
- [ ] TV connected in Remote Device Manager
- [ ] App installed and pinned to home screen
