# Oranuss Academy AI Editing: install steps for Claude Code

> Developed by Mehdi ORANUSS | Powered by Claude

You are installing **Oranuss Academy AI Editing** (Claude Code video editing for Moroccan Darija content) on this computer, because the user asked you to.
- Talk to the user in **Moroccan Darija written in Arabic letters**, short and simple.
- Say what you do before each step. Do every step in order and never skip a check.
- If something fails, stop, explain, and fix it with the user.

The editing files are in a **private** GitHub repo (`meeehdy832/oranuss-academy-ai-editing`). You download them only through the GitHub API with a read-only key.
- The key is in `key.lock` next to this file, **locked with a password** that the user got from Oranuss Academy.
- Do not use `git clone`. Do not open the repo page. Do not ask the user to sign in to GitHub.
- **Never print, log or repeat the password or the unlocked key.** Keep them only inside the commands below.

Names used below:
- `SETUP` = `https://raw.githubusercontent.com/meeehdy832/oranuss-academy-ai-editing-setup/main`
- `API` = `https://api.github.com/repos/meeehdy832/oranuss-academy-ai-editing/contents`
- Install folder: `oranuss-academy-ai-editing` in the user's Documents.

### 1. Detect the system
- **Mac:** `uname -sm` must be `Darwin arm64`. On an Intel Mac, stop and say the kit needs Apple Silicon.
- **Windows:** use PowerShell.
  - `[Environment]::OSVersion.Version` must be 10.0 or more.
  - `winget --version` must work. If winget is missing, ask the user to install **App Installer** from the Microsoft Store.
- Check there are 5 GB free (Mac: `df -h ~`; Windows: `Get-PSDrive C`).

### 2. Get the locked key
- **Mac:** `curl -fsSL "$SETUP/key.lock" -o /tmp/oranuss-key.lock`
- **Windows:** `Invoke-WebRequest -UseBasicParsing "$SETUP/key.lock" -OutFile "$env:TEMP\oranuss-key.lock"`

(Write `SETUP` out in full in the command.)

### 3. Password
- Ask in Darija: «عافاك كتب الكود السري (كلمة السر) اللي عطاتك Oranuss Academy». Do not repeat it back.
- **Unlock snippet.** It sets `KIT_TOKEN` from the password. Put it at the start of every command that needs the key (steps 3, 5 and 6), because each command runs in a fresh shell. Replace `<password>` with what the user typed, inside single quotes.
  - **Mac:**
    `KIT_TOKEN=$(KIT_PASS='<password>' /usr/bin/openssl enc -d -aes-256-cbc -pbkdf2 -iter 200000 -md sha256 -a -A -pass env:KIT_PASS -in /tmp/oranuss-key.lock 2>/dev/null)`
  - **Windows** (PowerShell):
    `$b=[Convert]::FromBase64String((Get-Content "$env:TEMP\oranuss-key.lock" -Raw).Trim()); $kdf=New-Object Security.Cryptography.Rfc2898DeriveBytes('<password>',[byte[]]$b[8..15],200000,[Security.Cryptography.HashAlgorithmName]::SHA256); $kv=$kdf.GetBytes(48); $aes=[Security.Cryptography.Aes]::Create(); $aes.Key=[byte[]]$kv[0..31]; $aes.IV=[byte[]]$kv[32..47]; $ct=[byte[]]$b[16..($b.Length-1)]; try { $env:KIT_TOKEN=[Text.Encoding]::UTF8.GetString($aes.CreateDecryptor().TransformFinalBlock($ct,0,$ct.Length)) } catch { $env:KIT_TOKEN='' }`
- **Check the password** without printing the key:
  - **Mac:** `<snippet>; case "$KIT_TOKEN" in github_pat_*|ghp_*) echo ok ;; *) echo wrong ;; esac`
  - **Windows:** `<snippet>; if ($env:KIT_TOKEN -match '^(github_pat_|ghp_)') { 'ok' } else { 'wrong' }`
- If it says `wrong`, say «الكود ماشي صحيح» and ask again. Allow at most **3 tries**. After 3 failures, stop and tell the user to contact Oranuss Academy.

### 4. Ask before the big downloads
Ask in Darija, yes or no for each:
- the tools: about 1 GB;
- the transcription model: 2.9 GB, needed to turn Darija speech into captions;
- earshot: optional, about 100 MB.

Recommend yes to the first two.

### 5. Prerequisites, then download and read the installer
**Prerequisites:**
- **Mac:**
  - If `brew` is missing, the user installs it, because it needs their Mac password. Give them this command in a `bash` block for the Terminal app, then wait:
    `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
  - Then: `echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile && eval "$(/opt/homebrew/bin/brew shellenv)"`
  - If `xcode-select -p` fails, the user runs `xcode-select --install`.
- **Windows:** nothing to do first. winget installs Git, FFmpeg, Node.js and Python. Windows may show permission windows: tell the user to click **Yes**.

**Download the installer:**
- **Mac:** `<snippet>; curl -fsSL -H "Authorization: Bearer $KIT_TOKEN" -H "Accept: application/vnd.github.raw" "$API/install.sh" -o /tmp/oranuss-install.sh`
- **Windows:** `<snippet>; Invoke-WebRequest -UseBasicParsing -Headers @{Authorization="Bearer $env:KIT_TOKEN"; Accept="application/vnd.github.raw"; "User-Agent"="oranuss"} -Uri "$API/install.ps1" -OutFile "$env:TEMP\oranuss-install.ps1"`

An HTTP 401 or 404 means the key expired or was revoked. Then stop and tell the user, in Darija, to contact Oranuss Academy.

**Read the whole installer before running it.** It must only:
- copy files;
- download the repo through api.github.com with the key;
- run `npx skills add` from GitHub;
- install tools (brew on Mac, winget on Windows);
- download files from huggingface.co and github.com.

### 6. Install
Run it in the background, once. Never run two at the same time.
- **Mac:** `<snippet>; KIT_TOKEN="$KIT_TOKEN" bash /tmp/oranuss-install.sh ~/Documents/oranuss-academy-ai-editing --tools --model`
  Add `--earshot` if the user accepted it.
- **Windows:** `<snippet>; powershell -ExecutionPolicy Bypass -File "$env:TEMP\oranuss-install.ps1" -Tools -Model`
  Add `-Earshot` if the user accepted it. When it finishes, open a new PowerShell so the new PATH is used.

Drop any flag the user refused. The installer never overwrites existing files, so running it again is safe.

### 7. Verify
- Every installer line must be ✅ / [OK]. Fix each ❌ / [X]: usually run the tool command it names, then run the installer again.
- Then check these yourself:
  - the global skills folder (`~/.claude/skills`, on Windows `%USERPROFILE%\.claude\skills`) has 30 or more folders;
  - the project skills folder (`oranuss-academy-ai-editing/.claude/skills`) has 15 or more;
  - `ffmpeg -version` works;
  - `whisper-cli --help` works;
  - `node -v` works.
- Delete the temporary files:
  - Mac: `/tmp/oranuss-install.sh` and `/tmp/oranuss-key.lock`;
  - Windows: `%TEMP%\oranuss-install.ps1` and `%TEMP%\oranuss-key.lock`.
- Show the user one short table in Darija with ✅ or ❌ for each item.

### 8. Explain (Darija, short)
- The editing folder is `Documents/oranuss-academy-ai-editing`.
- Where to put things:
  - videos to camera → `input/recordings/`;
  - voice-only audio → `input/voiceover/`;
  - images and clips → `input/B-ROLL/`;
  - music → `music/`.
- How a video gets made: Claude asks questions, writes a plan, shows still images, then a preview. The final video comes only after the user's OK.
- Claude Code must be opened **on that folder**, so that the house rules (`CLAUDE.md`) and the project skills load.
- **Windows only:** face tracking uses `project/.venv` with `project/tools/face_track.py` instead of the Mac helper, and `ffmpeg` and `whisper-cli` come from PATH.

### 9. Ask to start
Ask exactly this, then wait:

**«صافي، كلشي تثبت ✅. واش مستعد نبداو نخدمو على فيديو دابا، ولا تبغي نأجلوها لوقت آخر؟»**

- **Now:**
  - If this session is not in `Documents/oranuss-academy-ai-editing`: ask the user to open a new Claude Code session on that folder, put the video in `input/recordings/`, and write «بغيت نصاوب فيديو».
  - If it is: start. Ask about the video, the format (9:16 or 16:9) and the style.
- **Later:** say goodbye in Darija. Next time they only need to open a session on that folder and say «بغيت نصاوب فيديو».
