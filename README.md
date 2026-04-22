### Cloudflare SSH Debug Session (India Optimized)

Fork this repo into your account, then follow the steps below to set up secure SSH access to your GitHub Actions runner using Cloudflare Tunnel — no tmate lag, no relay servers, just fast and direct SSH from your phone.

---

### How It Works

```
Your Phone (Termux)
      │
      │  SSH
      ▼
Cloudflare Tunnel  ◄──────►  GitHub Actions Runner (Ubuntu)
      │
  (No relay,
  direct tunnel)
```

When the workflow runs, it:
1. Fetches your SSH public key from GitHub automatically
2. Starts a Cloudflare Tunnel that exposes port 22 (SSH) of the runner
3. Prints a unique tunnel URL in the Actions log
4. Only your key can authenticate — nobody else can get in

---

### Is It Safe?

**Yes.** Here's why:

- The tunnel URL is **random and changes every run** (e.g. `random-words.trycloudflare.com`)
- Even if someone finds the URL, they **cannot connect without your private SSH key**
- Your private key **never leaves your phone** — only the public key is on GitHub
- The runner is a **fresh isolated VM** every run — wiped after the job ends
- Session automatically dies after **6 hours max**

Think of it like this:
> The tunnel URL is your house address. Your SSH key is the door key. Finding the address is useless without the key.

---

### Requirements

### On Termux (Android)

```bash
# Install required packages
pkg update && pkg upgrade -y
pkg install openssh cloudflared -y
```

### On Ubuntu / Linux / PC

```bash
# Install required packages
sudo apt install openssh-client -y

# Install cloudflared
curl -fsSL https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -o cloudflared
chmod +x cloudflared
sudo mv cloudflared /usr/local/bin/cloudflared
```

---

### Step 1 — Generate SSH Key

Run this in **Termux** or your terminal, replacing the email with your GitHub account email:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

- **Enter file in which to save:** Press Enter to use the default location
- **Enter passphrase:** Press Enter twice to skip, or set a password for extra security

---

### Step 2 — Copy Your Public Key

Display your public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Select and copy the entire output — it starts with `ssh-ed25519` and ends with your email.

---

### Step 3 — Add Key to GitHub

1. Open your browser and go to [github.com/settings/keys](https://github.com/settings/keys)
2. Click **New SSH key**
3. **Title:** Enter a name like `Termux` or `My Phone`
4. **Key type:** Authentication Key
5. **Key:** Paste the public key you copied
6. Click **Add SSH key**

---

### Step 4 — Test GitHub SSH Connection

Verify your key works with GitHub:

```bash
ssh -T git@github.com
```

If successful you will see:

```
Hi YourUsername! You've successfully authenticated...
```

---

### Step 5 — Run the Workflow

1. Go to your forked repo on GitHub
2. Click the **Actions** tab
3. Select **Cloudflare SSH Debug Session (India Optimized)**
4. Click **Run workflow** → **Run workflow**
5. Open the running job and expand the **Start Cloudflare Tunnel** step
6. Wait for the tunnel URL to appear in the logs

You will see something like:

```
============================================
Tunnel is ready!

STEP 1 - Install cloudflared on Termux:
  pkg install cloudflared

STEP 2 - Connect via SSH:
  ssh -o ProxyCommand='cloudflared access tcp --hostname https://random-words.trycloudflare.com' runner@localhost

Tunnel URL: https://random-words.trycloudflare.com
Triggered by: YourUsername
============================================
```

---

## Step 6 — Connect via SSH

Copy the SSH command from the logs and run it in **Termux**:

```bash
ssh -o ProxyCommand='cloudflared access tcp --hostname https://YOUR-TUNNEL-URL.trycloudflare.com' runner@localhost
```

If your key is in a custom location, specify it with `-i`:

```bash
ssh -i ~/.ssh/id_ed25519 -o ProxyCommand='cloudflared access tcp --hostname https://YOUR-TUNNEL-URL.trycloudflare.com' runner@localhost
```

You are now inside the GitHub Actions Ubuntu runner!

---

## Tips

**Hide tunnel URL from public logs (optional)**

If your repo is public, send the tunnel URL privately to yourself via Telegram instead of printing it in logs. Add these secrets to your repo:

- `TG_BOT_TOKEN` — your Telegram bot token
- `TG_CHAT_ID` — your Telegram chat ID

Then add this step to your workflow:

```yaml
- name: Send URL via Telegram
  run: |
    curl -s "https://api.telegram.org/bot${{ secrets.TG_BOT_TOKEN }}/sendMessage" \
      -d chat_id=${{ secrets.TG_CHAT_ID }} \
      -d text="SSH: ssh -o ProxyCommand='cloudflared access tcp --hostname ${CF_URL}' runner@localhost"
```

**Run tasks in background so disconnects don't matter:**

```bash
### Inside the runner — start any task in tmux
tmux new -d -s work 'your_command 2>&1 | tee output.log'

### Reconnect anytime and check progress
tmux attach -t work

### Watch only errors
tail -f output.log | grep -i error
```

**Cancel the workflow when done** — don't leave it idle to avoid wasting your free GitHub Actions minutes.

---

## Why Cloudflare Instead of Tmate?

| Feature | Tmate | Cloudflare Tunnel |
|---|---|---|
| Relay server | Yes (adds lag) | No (direct tunnel) |
| India latency | High | Low |
| Auth token needed | No | No |
| Stability | Drops sometimes | More stable |
| Free | Yes | Yes |

---

## Thanks

Big thanks to **GitHub** for providing free Actions runners with 16GB RAM, 4 cores, and ~140GB disk — making powerful cloud computing accessible to everyone for free. 🙏

And thanks to **Cloudflare** for their free tunnel service that makes this lag-free SSH connection possible.
