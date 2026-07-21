# Boosting pacman speed (ArchLinux)
Сначала делаем бэкап `/etc/pacman.d/mirrorlist`. Затем используя пакет `reflector` запустить команду для изменения списка зеркал с сохранением в файл:
```bash
sudo reflector --verbose --country 'DE','NL','SE','FI' --latest 15 --sort rate --protocol https --save /etc/pacman.d/mirrorlist
```
# Traffic redirect
The `net.ipv4.ip_forward` parameter is a Linux kernel sysctl variable that controls whether your system routes network packets between different network interfaces.
Check:
```bash
sysctl net.ipv4.ip_forward
```
Change:
```bash
sysctl -w net.ipv4.ip_forward=1
```
# VS Code
## GDB Terminal error
Issue: warning: GDB: Failed to set controlling terminal: Operation not permitted
Solution: Add this to `settings.json`:
```json
"terminal.integrated.automationProfile.linux": {
	"path": "/bin/bash"
}
```
Or simply restart the system maybe?
# Obsidian
## Low fps
**Force GPU Usage via Windows Graphics Settings:**
- Open the Start menu, type **Graphics Settings**, and press Enter.
- Click **Browse** and locate your Obsidian executable (typically found at `C:\Users\YOUR_USERNAME\AppData\Local\Obsidian\Obsidian.exe`).
- Once added, click on Obsidian, select **Options**, and set it to use your **High Performance GPU** (NVIDIA or AMD).