```python
import paramiko
import json

# Load commands from JSON
with open("commands.json", "r") as file:
    data = json.load(file)
    commands = data["commands"]

# SSH setup
ssh_key_path = "C:/Users/Ankit-kushwaha90/.ssh/id_ed25519"
key_passphrase = "ankit99109"

# Load private key with passphrase
pkey = paramiko.Ed25519Key.from_private_key_file(ssh_key_path, password=key_passphrase)

# Connect to localhost
client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
client.connect(hostname="localhost", username="Ankit-kushwaha90", pkey=pkey)

# Run each command and print output
for command in commands:
    stdin, stdout, stderr = client.exec_command(command)
    print(f"\n$ {command}")
    print("Output:", stdout.read().decode())
    print("Error:", stderr.read().decode())

client.close()
```

Here is a clear, direct guide to enable the SSH service (OpenSSH Server) in Windows 11:

## 1️⃣ Check if OpenSSH Server is installed
Open PowerShell as Administrator:

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH.Server*'
```
If it shows:

- State : Installed → already installed.

-  State : NotPresent → you need to install it.

## 2️⃣ Install OpenSSH Server (if not installed)
Run:

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```
Wait for it to complete.

## 3️⃣ Start the SSH service
Run:

```powershell
Start-Service sshd
```
## 4️⃣ Enable SSH to start automatically on boot
Run:

```powershell
Set-Service -Name sshd -StartupType 'Automatic'
```
## 5️⃣ Allow SSH in Windows Firewall
Run:

```powershell
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```
## 6️⃣ Verify SSH Service Status
Run:

```powershell
Get-Service sshd
```
Status should show Running.

## 7️⃣ Test SSH Access
From another system (or your phone using Termux) on the same network:

```bash
ssh username@your_windows_ip
```
Replace username with your Windows username.

- You may accept the host key prompt on first connection.

Troubleshooting
✅ SSH connection refused?

- Ensure the service is running: Get-Service sshd

- Check firewall rules: Get-NetFirewallRule | Where-Object DisplayName -like "*ssh*"

Verify that port 22 is open:

```powershell
Test-NetConnection -ComputerName localhost -Port 22
```
✅ Want to change the port?
Edit:

```makefile
C:\ProgramData\ssh\sshd_config
```
Uncomment and change:

```nginx
Port 22
```
Then restart SSH service:

```powershell
Restart-Service sshd
```
If you need, I can also generate scripts for auto-installation, configuration, and status reporting for your Windows 11 environment to automate this for your workflow. Let me know if you would like that.

Here is your clean, corrected sshd_config specifically for enabling SSH login using username + password on Windows 11 (without public key enforcement):

## ✅ What you need:
1️⃣ Enable password authentication explicitly.
2️⃣ Ensure your user can log in using username + password via SSH.
3️⃣ Allow admin users to log in using password instead of requiring admin authorized_keys.

## ⚡ Here is the correct, minimal sshd_config you should use:
```ini
# This is the sshd server system-wide configuration file. See sshd_config(5).

# Port and Listen settings (default, no change)
#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

# HostKeys (default, no change)
#HostKey __PROGRAMDATA__/ssh/ssh_host_rsa_key
#HostKey __PROGRAMDATA__/ssh/ssh_host_dsa_key
#HostKey __PROGRAMDATA__/ssh/ssh_host_ecdsa_key
#HostKey __PROGRAMDATA__/ssh/ssh_host_ed25519_key

# Authentication:
LoginGraceTime 2m
PermitRootLogin prohibit-password
StrictModes yes
MaxAuthTries 6
MaxSessions 10

# Enable public key authentication if you want to use it later
PubkeyAuthentication yes

# Allow authorized_keys in the user's profile folder
AuthorizedKeysFile .ssh/authorized_keys

# Enable password authentication explicitly
PasswordAuthentication yes

# Disallow empty passwords for security
PermitEmptyPasswords no

# Logging level (optional)
#LogLevel VERBOSE

# Subsystem for sftp (default)
Subsystem sftp sftp-server.exe

# REMOVE the admin override block to allow admins to use normal password login
# Commenting out these lines:
# Match Group administrators
#        AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```
✅ Key explanations:
✅ PasswordAuthentication yes → explicitly enables password login.
✅ PermitEmptyPasswords no → ensures blank passwords are rejected.
✅ PubkeyAuthentication yes → keeps key auth available for future use but is not mandatory.
✅ Commenting out Match Group administrators ensures admin users (like your user) use C:\Users\<user>\.ssh\authorized_keys and can log in with password.

## 🚀 Steps after editing:
## 1️⃣ Save this updated sshd_config to:

```makefile
C:\ProgramData\ssh\sshd_config
```
## 2️⃣ Restart the SSH server:

```powershell
Restart-Service sshd
```
## 3️⃣ Test SSH connection:

```bash
ssh ankit-kushwaha90@localhost
```
✅ Enter your Windows password (not PIN, not key passphrase).

## 4️⃣ If connection succeeds, you are fully set for SSH login using username + password.

## ⚡ What this achieves:
✅ Enables username + password SSH login on your Windows 11 system.
✅ Keeps publickey option open for future key-based login.
✅ Removes admin key override to simplify login for administrator users.
✅ Ensures clear, maintainable SSH configuration for your MCP + terminal automation workflows.

If you want, I can also generate:
✅ An automated PowerShell script to:

- Backup your current sshd_config.

- Apply these safe changes automatically.

- Restart SSH service.

- Test connectivity and report status.

- Let me know if you would like this for one-click setup and testing on your system.
