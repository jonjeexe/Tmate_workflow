## Tmate Workflow
Fork the repo into your account then

Generate and add an SSH key to GitHub using the command line. This setup is required to use `limit-access-to-actor: true` in your GitHub Actions tmate session.

<br>

1. Install Necessary Tools.

```bash
sudo apt install openssh-client git -y
```
<br>

2. Generate the SSH Key.
Run the following command, replacing the email with your GitHub account email.

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

- Enter file in which to save: Press Enter to use the default location.
- Enter passphrase: Press Enter twice to skip, or provide a password for extra security.
  
<br>

3. Copy the Public Key
Display the content of your public key so you can copy it

```bash
cat ~/.ssh/id_ed25519.pub
```

Select the entire output text (starting with ssh-ed25519 and ending with your email) and Copy it.


<br>

4. Add the Key to GitHub.

- Open your browser and log in to your GitHub SSH Settings.
- Click New SSH key.
- Title: Enter a name like "Terminal-debugging".
- Key: Paste the public key you copied from Terminal.
- Click Add SSH key.

<br>
5. Test the Connection
Verify that Terminal can successfully connect to GitHub via SSH

```bash
ssh -T git@github.com
```

If successful, you will see a message: "Hi [YourUsername]! You've successfully authenticated..."

<br>

6. Connect to your GitHub Action
Once the Action starts and provides an SSH string, use it directly in Terminal.

```bash
ssh <provided_tmate_string>
```

Because you added your key to GitHub and used `limit-access-to-actor: true` in your workflow, Terminal will now be the only device allowed to connect to that session <br>

# Tmate tweak to off green line

```bash
tmux set-option -g status off
```
