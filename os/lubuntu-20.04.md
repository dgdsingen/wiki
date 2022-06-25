# Boot Repair

```bash
sudo add-apt-repository ppa:yannubuntu/boot-repair
sudo apt-get update
sudo apt-get install -y boot-repair && boot-repair
# Click "Recommended Repair"
```

# Locale

- Go to Start menu > Preferences > LXQt settings > Locale
    - Under "Region:", select the upmost entry "No change".
    - Click "Close" and confirm to save the new settings.
- Go to Start menu > Preferences > LXQt settings > Session Settings
    - in section "Environment (Advanced)" make sure that no locale-related environment variables are set here: LANG, LANGUAGE and any LC_* variables.



```bash
sudo vi /etc/default/locale

LANG=ko_KR.UTF-8
LANGUAGE=ko_KR.UTF-8
LC_ADDRESS=ko_KR.UTF-8
LC_IDENTIFICATION=ko_KR.UTF-8
LC_MEASUREMENT=ko_KR.UTF-8
LC_MONETARY=ko_KR.UTF-8
LC_NAME=ko_KR.UTF-8
LC_NUMERIC=ko_KR.UTF-8
LC_PAPER=ko_KR.UTF-8
LC_TELEPHONE=ko_KR.UTF-8
LC_TIME=ko_KR.UTF-8
LC_MESSAGES=POSIX

cp /etc/default/locale ~/.pam_environment
```

# DateTime

MM-dd (ddd) HH:mm

# Set Google Chrome as default web browser

- by CLI

```bash
sudo update-alternatives --set x-www-browser /usr/bin/google-chrome-stable
sudo update-alternatives --set gnome-www-browser /usr/bin/google-chrome-stable
sudo update-alternatives --config x-www-browser # 자동 모드 선택
sudo update-alternatives --config gnome-www-browser # 자동 모드 선택
```

- by GUI
  - start on the taskbar > Preferences > Alternative Configurator
  - select gnome-www-browser
  - select google-chrome-stable
  - click auto button to make it as default web browser

# Stopping Keyring Popup after reboot

```bash
sudo apt-get install -y seahorse
```

- After the installation is complete, launch "seahorse" from programs.
- Right-click on "Login" and select "Change Password."
- Enter the old password when you see the pop-up. Then leave the new password field blank. Don’t enter even space. Click 'Continue'.
- You should see an obvious warning pop-up that passwords will be unencrypted. Click 'Continue'.
- That’s it! Restart your computer for the setting to take effect. Next time you launch Chrome or Chromium browser, you should not see the keyring request.