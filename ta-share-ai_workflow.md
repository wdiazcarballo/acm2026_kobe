# Phase 0 – HOW to Prepare the Environment for Agentic AI Development

> Goal: You end this phase with VSCode, Node, Git, GitHub CLI, Claude Code, and the AI backend all working together.[file:31]

---

## Step 0.1 – Install VSCode (HOW)

### On Windows

1. **Option A: Using winget (recommended on Windows 11)**  
   1. Press `Win + R`.  
   2. Type `powershell` and press Enter.  
   3. In PowerShell, run:

      ```powershell
      winget install Microsoft.VisualStudioCode
      ```

   4. Wait until the command finishes (it will show progress bars).
   5. Press `Win` and type “Visual Studio Code”. You should see VSCode in the Start menu.

2. **Option B: Using the website**  
   1. Open a browser and go to <https://code.visualstudio.com/>.[file:31]  
   2. Click the **Download for Windows** button.  
   3. Run the downloaded `.exe` file.  
   4. In the installer, keep the default options and click **Next** until **Install**.  
   5. After installation, start VSCode from the Start menu.

### On macOS

1. Open **Terminal** (`Cmd + Space` → type “Terminal”).  
2. Install Homebrew if you don’t have it yet (copy–paste this and press Enter):

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

3. Once Homebrew is installed, run:

   ```bash
   brew install --cask visual-studio-code
   ```

4. Open Launchpad and click **Visual Studio Code**.[file:31]

### On Ubuntu/Debian

1. Open **Terminal**.  
2. Update packages:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

3. Install VSCode (snap method):

   ```bash
   sudo snap install code --classic
   ```

4. Start VSCode:

   ```bash
   code
   ```[file:31]

**Check:** In any system, run:

```bash
code --version
```

If you see a version (e.g. `1.85.0` or higher), VSCode is installed correctly.[file:31]

---

## Step 0.2 – Install Node.js (HOW)

### On Windows

1. **Option A: Using winget**  
   1. Open **PowerShell**.  
   2. Run:

      ```powershell
      winget install OpenJS.NodeJS.LTS
      ```

2. **Option B: Using the website**  
   1. Go to <https://nodejs.org/>.[file:31]  
   2. Click the **LTS** version (recommended).  
   3. Download the Windows installer (`.msi`).  
   4. Run it, keep defaults, click **Next** until **Install**.

3. After installation, reopen PowerShell and run:

   ```powershell
   node --version
   npm --version
   ```

You should see something like `v18.x.x` for Node and `9.x.x` for npm.[file:31]

### On macOS

1. Ensure Homebrew is installed (see VSCode step).  
2. Run:

   ```bash
   brew install node
   ```

3. Check:

   ```bash
   node --version
   npm --version
   ```[file:31]

### On Ubuntu/Debian

1. Open Terminal.  
2. Run:

   ```bash
   curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
   sudo apt install -y nodejs
   ```

3. Check:

   ```bash
   node --version
   npm --version
   ```[file:31]

---

## Step 0.3 – Install Git (HOW)

### On Windows

1. **Option A: Using winget**  
   1. Open PowerShell.  
   2. Run:

      ```powershell
      winget install Git.Git
      ```

2. **Option B: Using the website**  
   1. Go to <https://git-scm.com/>.[file:31]  
   2. Click **Download for Windows**.  
   3. Run the installer and accept defaults (unless you know you need specific options).

3. Check in a new PowerShell window:

   ```powershell
   git --version
   ```

You should see something like `git version 2.x.x`.[file:31]

### On macOS

1. Open Terminal.  
2. Run:

   ```bash
   brew install git
   ```

3. Check:

   ```bash
   git --version
   ```[file:31]

### On Ubuntu/Debian

1. Open Terminal.  
2. Run:

   ```bash
   sudo apt install git -y
   ```

3. Check:

   ```bash
   git --version
   ```[file:31]

---

## Step 0.4 – Install GitHub CLI (gh) (HOW)

### On Windows

1. **Using winget:**  
   1. Open PowerShell.  
   2. Run:

      ```powershell
      winget install GitHub.cli
      ```

2. After installation, run:

   ```powershell
   gh --version
   ```

You should see the GitHub CLI version.[file:31]

### On macOS

1. Open Terminal.  
2. Run:

   ```bash
   brew install gh
   ```

3. Check:

   ```bash
   gh --version
   ```[file:31]

### On Ubuntu/Debian

1. Open Terminal.  
2. Run exactly these commands in order:[file:31]

   ```bash
   type -p curl >/dev/null || (sudo apt update && sudo apt install curl -y)
   curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
   sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
   sudo apt update
   sudo apt install gh -y
   ```

3. Check:

   ```bash
   gh --version
   ```[file:31]

---

## Step 0.5 – Log In to GitHub via gh (HOW)

**Why:** The later workflow (`ccc`, `nnn`, `gogogo`, `rrr`) relies on creating issues and PRs programmatically.[file:30][file:31]

1. Run:

   ```bash
   gh auth login
   ```

2. Answer prompts:
   - “What account do you want to log into?” → `GitHub.com`
   - “Preferred protocol?” → `HTTPS`
   - “Authenticate Git with your GitHub credentials?” → `Yes`
   - “How would you like to authenticate GitHub CLI?” → `Login with a web browser`[file:31]

3. Copy the one‑time code shown in the terminal.
4. Your browser should open automatically; paste the code and click **Authorize**.
5. Back in the terminal:

   ```bash
   gh auth status
   ```

   You should see something like:

   ```text
   ✓ Logged in to github.com as YOUR_USERNAME
   ✓ Git operations for github.com configured to use https protocol.
   ✓ Token: *******************
   ```[file:31]

---

## Step 0.6 – Install Claude Code Extension (HOW)

**Why:** This is the actual agentic coding assistant that will follow the workflow defined in `CLAUDE.md`.[file:31]

1. Open VSCode.
2. Press `Ctrl+Shift+X` (or `Cmd+Shift+X` on Mac) to open Extensions.[file:31]
3. In the search box, type:

   ```
   Claude Code
   ```

4. Find the extension where the publisher is **Anthropic**.
5. Click **Install**.
6. After it finishes, click **Reload** or close and reopen VSCode.
7. Confirm:
   - On the left Activity Bar, you should see a Claude lightning icon (⚡).[file:31]

---

## Step 0.7 – Configure AI Backend and Token (HOW)

**Why:** Without this, Claude Code cannot call the AI models (e.g. via z.ai).[file:31]

### On Windows

1. Get your API token from the z.ai dashboard and copy it.[file:31]
2. Open **PowerShell** and run:

   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude-zz"
   ```

3. Create `config.json` with your token:

   ```powershell
   @"
   {
     "primaryApiKey": "YOUR_API_TOKEN_HERE"
   }
   "@ | Out-File -FilePath "$env:USERPROFILE\.claude-zz\config.json" -Encoding utf8
   ```

4. Check the file:

   ```powershell
   Get-Content "$env:USERPROFILE\.claude-zz\config.json"
   ```

   You should see JSON with your key.[file:31]

5. In VSCode, open **Settings (JSON)**:
   - `Ctrl+,` → click the `{}` icon in the top‑right (Open Settings JSON), or
   - `Ctrl+Shift+P` → “Preferences: Open User Settings (JSON)”.

6. Add environment variables for Claude Code:

   ```json
   {
     "claude-code.environmentVariables": [
       {
         "name": "ANTHROPIC_BASE_URL",
         "value": "https://api.z.ai/api/anthropic"
       },
       {
         "name": "ANTHROPIC_AUTH_TOKEN",
         "value": "YOUR_API_TOKEN_HERE"
       },
       {
         "name": "ANTHROPIC_MODEL",
         "value": "glm-4.6"
       },
       {
         "name": "CLAUDE_CONFIG_DIR",
         "value": "C:\\Users\\YOUR_USERNAME\\.claude-zz"
       }
     ]
   }
   ```

   Replace:
   - `YOUR_API_TOKEN_HERE` with your real token.
   - `YOUR_USERNAME` with the actual Windows username (you can get it with `echo %USERNAME%` in cmd).[file:31]

7. Save the file, close VSCode completely, and reopen it.

### On macOS / Linux

1. Create the config directory:

   ```bash
   mkdir -p ~/.claude-zz
   ```

2. Create `config.json`:

   ```bash
   cat > ~/.claude-zz/config.json << 'EOF'
   {
     "primaryApiKey": "YOUR_API_TOKEN_HERE"
   }
   EOF
   ```

3. Open VSCode settings JSON as above and add:

   ```json
   {
     "claude-code.environmentVariables": [
       {
         "name": "ANTHROPIC_BASE_URL",
         "value": "https://api.z.ai/api/anthropic"
       },
       {
         "name": "ANTHROPIC_AUTH_TOKEN",
         "value": "YOUR_API_TOKEN_HERE"
       },
       {
         "name": "ANTHROPIC_MODEL",
         "value": "glm-4.6"
       },
       {
         "name": "CLAUDE_CONFIG_DIR",
         "value": "/Users/YOUR_USERNAME/.claude-zz"
       }
     ]
   }
   ```

   Replace:
   - `YOUR_API_TOKEN_HERE` with your token.
   - `YOUR_USERNAME` with your actual username (e.g. `john`).[file:31]

4. Save and restart VSCode.

---

## Step 0.8 – Smoke Test the Agent

**What you are doing:** Checking that everything is wired up **before** you try the full development workflow.[file:31]

1. Open any folder in VSCode.
2. Open the Claude panel (⚡ icon on the left).
3. Type a simple request, for example:

   ```
   สวัสดี! ช่วยเขียน function JavaScript ที่ reverse string ให้หน่อย
   ```

4. Expect:
   - Claude responds with code.
   - No errors like “Authentication failed” or “No available IDEs detected”.[file:31]

If there is an error, go back to the appropriate step (token, config paths, or settings JSON) and correct it.

---

**End of Phase 0**  

At this point the environment is fully prepared: tools installed, GitHub authenticated, and the agentic AI connected and tested.[file:31] The next phases (repo setup, `CLAUDE.md` contract, and the ccc/nnn/gogogo/rrr loop) build on this foundation.

---

Given this more concrete “how” for environment prep, would you like the next part of the workflow (ccc → nnn → gogogo → rrr) written with the same level of detail—i.e., exact `gh` and `git` commands and example instructions for each short code?
