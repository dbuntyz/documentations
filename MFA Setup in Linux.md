# Multi-Factor Authentication (MFA) Setup on Linux

This guide outlines how to set up Multi-Factor Authentication (MFA) using Google Authenticator for SSH access on a Linux system.

## Step 1: Install Required Packages

You will need to install `pam_google_authenticator`, which is the PAM (Pluggable Authentication Module) used to integrate Google Authenticator with SSH.

1. **Update your package list**:
   ```bash
   sudo yum update  # For RHEL/CentOS-based systems
   sudo yum install google-authenticator
   google-authenticator
2. **Follow the on-screen instructions**:

- You will be asked whether you want to update the .google_authenticator file (answer yes).
- The system will display a QR code for you to scan using the Google Authenticator mobile app (available on iOS/Android).
- After scanning the QR code, you will be prompted with several options. You can accept the defaults (answer yes to everything, or configure according to your needs).
- The setup will generate a secret key, emergency scratch codes, and a QR code. Make sure to save the scratch codes in case you lose access to the Google Authenticator app.
## Step 3: Configure PAM (Pluggable Authentication Modules)

 ```bash
 sudo nano /etc/pam.d/sshd

 auth       substack     password-auth
auth       required     pam_google_authenticator.so
auth       include      postlogin
account    required     pam_sepermit.so
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
session    required     pam_selinux.so close
session    required     pam_loginuid.so
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    optional     pam_motd.so
session    include      password-auth
session    include      postlogin

sudo nano /etc/ssh/sshd_config

ChallengeResponseAuthentication yes
AuthenticationMethods publickey,keyboard-interactive
UsePAM yes

/etc/ssh/sshd_config.d/50-redhat.conf

# This system is following system-wide crypto policy. The changes to
# crypto properties (Ciphers, MACs, ...) will not have any effect in
# this or following included files. To override some configuration option,
# write it before this block or include it before this file.
# Please, see manual pages for update-crypto-policies(8) and sshd_config(5).
Include /etc/crypto-policies/back-ends/opensshserver.config

SyslogFacility AUTHPRIV

ChallengeResponseAuthentication yes
PasswordAuthentication yes
PubkeyAuthentication yes
KbdInteractiveAuthentication yes

GSSAPIAuthentication yes
GSSAPICleanupCredentials no

UsePAM yes

X11Forwarding yes

# It is recommended to use pam_motd in /etc/pam.d/sshd instead of PrintMotd,
# as it is more configurable and versatile than the built-in version.
PrintMotd no


sudo systemctl restart sshd

