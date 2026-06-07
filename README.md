# Autostart KRDP on Manjaro with KDE Plasma

## 1. Install required packages

```bash
sudo pacman -Syu --needed krdp kwalletmanager kwallet-pam qt6-tools
```

## 2. Enable Autologin

1. Go to **System Settings - Login Screen (SDDM)**
2. In **Behavior**, enable **Automatically log in**
3. Select your user account
4. Apply changes

## 3. Set Blank KWallet Password

1. Open **KWalletManager**
2. Select your default wallet (`kdewallet`)
3. Click on **Change Password...**
4. Leave **both** Old and New password fields **completely empty**
5. Confirm the change

## 4. Configure KRDP Server

1. Go to **System Settings - Networking - Remote Desktop**
2. Toggle on the **Enable RDP Server**
3. Add your user account and set a password under "Usernames"
4. Check **Autostart at login** toggle below
5. Apply all changes

## 5. Configure firewall

```bash
sudo firewall-cmd --permanent --add-port=3389/tcp
sudo firewall-cmd --reload
```

## 6. Create Wallet Auto-Open Service

```bash
mkdir -p ~/.config/systemd/user

cat > ~/.config/systemd/user/open-kwallet.service << EOF
[Unit]
Description=Open KWallet
After=plasma-kwallet-pam.service kwalletd6.service
Wants=plasma-kwallet-pam.service

[Service]
Type=oneshot
ExecStart=/usr/bin/qdbus6 org.kde.kwalletd6 /modules/kwalletd6 org.kde.KWallet.open kdewallet 0 "$(whoami)"
RemainAfterExit=yes

[Install]
WantedBy=graphical-session.target
EOF
```

## 7. Make KRDP Wait for Wallet

```bash
systemctl --user edit app-org.kde.krdpserver.service
```

Edit empty section to look like this:

```txt
### Anything between here and the comment below will become the contents of the drop-in file

[Unit]
After=open-kwallet.service plasma-kwallet-pam.service kwalletd6.service
Wants=open-kwallet.service

### Edits below this comment will be discarded
```

## 8. Enable Services

```bash
systemctl --user daemon-reload
systemctl --user enable --now open-kwallet.service
systemctl --user enable --now app-org.kde.krdpserver.service
```

## 9. Reboot

```bash
sudo reboot
```
