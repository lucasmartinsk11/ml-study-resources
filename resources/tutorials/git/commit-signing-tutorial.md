# Git Commit Signing Tutorial

> **Purpose**: Complete guide to set up and use Git commit signing for verified commits on GitHub/GitLab.  
> **Author**: Lucas Martins  
> **Date**: June 25, 2025  
> **OS**: macOS (with notes for other systems)

## 📋 Table of Contents

1. [Why Sign Commits?](#why-sign-commits)
2. [Prerequisites](#prerequisites)
3. [Method 1: GPG Signing (Recommended)](#method-1-gpg-signing-recommended)
4. [Method 2: SSH Signing (Simpler)](#method-2-ssh-signing-simpler)
5. [Platform Setup](#platform-setup)
6. [Verification & Testing](#verification--testing)
7. [Troubleshooting](#troubleshooting)
8. [Quick Reference](#quick-reference)

---

## Why Sign Commits?

**Benefits:**
- ✅ **Authenticity**: Proves commits actually came from you
- ✅ **Security**: Prevents impersonation of your commits
- ✅ **Trust**: GitHub/GitLab shows "Verified" badge
- ✅ **Compliance**: Required in many enterprise environments

**Visual Result:**
```
✓ Verified commit on GitHub/GitLab
vs
⚠️ Unverified commit (could be anyone)
```

---

## Prerequisites

### Check Current Git Configuration
```bash
# Verify your Git identity
git config --global user.name
git config --global user.email

# If not set, configure them:
git config --global user.name "Lucas Martins"
git config --global user.email "your.email@example.com"
```

### Install Required Tools (macOS)
```bash
# Install GPG tools via Homebrew
brew install gnupg pinentry-mac

# Or install GitHub CLI (includes signing support)
brew install gh
```

---

## Method 1: GPG Signing (Recommended)

### Step 1: Generate GPG Key

```bash
# Start GPG key generation
gpg --full-generate-key
```

**Interactive Prompts:**
1. **Key type**: `(1) RSA and RSA (default)`
2. **Key size**: `4096`
3. **Expiration**: `0` (never expires) or set your preference
4. **Real name**: `Lucas Martins` (same as Git config)
5. **Email**: `your.email@example.com` (same as Git config)
6. **Comment**: Leave empty or add description
7. **Passphrase**: Choose a strong passphrase

### Step 2: Get Your GPG Key ID

```bash
# List your GPG keys
gpg --list-secret-keys --keyid-format=long

# Output will look like:
# sec   rsa4096/ABC123DEF456 2025-06-25 [SC]
#       Your-Long-Key-Fingerprint-Here
# uid                 [ultimate] Lucas Martins <your.email@example.com>
# ssb   rsa4096/XYZ789GHI012 2025-06-25 [E]

# Your Key ID is: ABC123DEF456 (after rsa4096/)
```

### Step 3: Configure Git

```bash
# Set your GPG key (replace with your actual key ID)
git config --global user.signingkey ABC123DEF456

# Enable automatic signing for all commits
git config --global commit.gpgsign true

# Set GPG program path
git config --global gpg.program gpg
```

### Step 4: Export Public Key

```bash
# Export your public key for GitHub/GitLab
gpg --armor --export ABC123DEF456

# Copy the entire output (including BEGIN/END lines)
```

### Step 5: Configure GPG Agent (macOS)

```bash
# Create or edit GPG agent config
echo "pinentry-program /opt/homebrew/bin/pinentry-mac" >> ~/.gnupg/gpg-agent.conf

# Add to shell profile (if using zsh)
echo 'export GPG_TTY=$(tty)' >> ~/.zshrc

# Reload shell configuration
source ~/.zshrc

# Restart GPG agent
gpgconf --kill gpg-agent
```

---

## Method 2: SSH Signing (Simpler)

### Step 1: Use Existing SSH Key or Generate New

```bash
# Check if you have an SSH key
ls -la ~/.ssh/

# If no key exists, generate one
ssh-keygen -t ed25519 -C "your.email@example.com"
# Or for older systems:
ssh-keygen -t rsa -b 4096 -C "your.email@example.com"
```

### Step 2: Configure Git for SSH Signing

```bash
# Set SSH key for signing (adjust path if needed)
git config --global user.signingkey ~/.ssh/id_ed25519.pub

# Set Git to use SSH format for signing
git config --global gpg.format ssh

# Enable automatic signing
git config --global commit.gpgsign true
```

### Step 3: Get Public Key Content

```bash
# Copy your SSH public key
cat ~/.ssh/id_ed25519.pub
# or
cat ~/.ssh/id_rsa.pub
```

---

## Platform Setup

### GitHub Setup

1. **Go to**: GitHub.com → Settings → SSH and GPG keys
2. **For GPG**: Click "New GPG key" → Paste GPG public key
3. **For SSH**: Click "New SSH key" → Select "Signing Key" → Paste SSH public key

### GitLab Setup

1. **Go to**: GitLab.com → Preferences → GPG Keys (or SSH Keys)
2. **Add your key**: Paste the public key content

### Verification

After adding your key, commits will show as "Verified" ✅

---

## Verification & Testing

### Test Your Setup

```bash
# Navigate to a repository
cd /path/to/your/repo

# Make a test commit
echo "test signing" > test-signing.txt
git add test-signing.txt
git commit -m "Test signed commit"

# Verify the signature
git log --show-signature -1

# Clean up test file
git reset --hard HEAD~1
rm test-signing.txt
```

### Check Git Configuration

```bash
# View all signing-related config
git config --global --list | grep -E "(sign|gpg|user)"
```

---

## Troubleshooting

### Common Issues & Solutions

#### 1. "gpg: signing failed: No secret key"
```bash
# List keys and verify key ID
gpg --list-secret-keys --keyid-format=long
git config --global user.signingkey YOUR_CORRECT_KEY_ID
```

#### 2. "gpg: signing failed: Inappropriate ioctl for device"
```bash
# Add to shell profile
echo 'export GPG_TTY=$(tty)' >> ~/.zshrc
source ~/.zshrc
```

#### 3. "error: gpg failed to sign the data"
```bash
# Test GPG directly
echo "test" | gpg --clearsign

# If this fails, restart GPG agent
gpgconf --kill gpg-agent
```

#### 4. Pinentry Issues on macOS
```bash
# Reinstall pinentry-mac
brew reinstall pinentry-mac

# Update GPG agent config
echo "pinentry-program /opt/homebrew/bin/pinentry-mac" > ~/.gnupg/gpg-agent.conf
gpgconf --kill gpg-agent
```

#### 5. SSH Signing Not Working
```bash
# Verify SSH key path
ls -la ~/.ssh/
git config --global user.signingkey

# Ensure format is set correctly
git config --global gpg.format ssh
```

### Debug Commands

```bash
# Test GPG functionality
gpg --version
gpg --list-keys
echo "test" | gpg --clearsign

# Test SSH key
ssh-add -l
ssh-keygen -y -f ~/.ssh/id_ed25519

# Check Git config
git config --list --show-origin | grep sign
```

---

## Quick Reference

### Essential Commands

```bash
# Generate GPG key
gpg --full-generate-key

# List GPG keys
gpg --list-secret-keys --keyid-format=long

# Export public key
gpg --armor --export KEY_ID

# Configure Git GPG signing
git config --global user.signingkey KEY_ID
git config --global commit.gpgsign true

# Configure Git SSH signing
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global gpg.format ssh
git config --global commit.gpgsign true

# Manual signing (if auto-signing disabled)
git commit -S -m "Signed commit message"

# Verify signatures
git log --show-signature
```

### Configuration Files

```bash
# Git config location
~/.gitconfig

# GPG agent config
~/.gnupg/gpg-agent.conf

# Shell profile (add GPG_TTY export)
~/.zshrc  # or ~/.bash_profile
```

---

## Notes & Best Practices

### Security Tips
- 🔐 Use a strong passphrase for GPG keys
- 💾 Backup your GPG private key securely
- ⏰ Consider setting key expiration dates
- 🔄 Rotate keys periodically for maximum security

### Workflow Integration
- ✅ Enable automatic signing to avoid forgetting
- 🧪 Test signing setup on new machines
- 📝 Document your key management process
- 🔄 Keep platform keys (GitHub/GitLab) updated

### Troubleshooting Strategy
1. **First**: Check Git configuration with `git config --list`
2. **Second**: Test GPG/SSH directly outside of Git
3. **Third**: Verify platform (GitHub/GitLab) key setup
4. **Last**: Check system-specific issues (pinentry, agents, etc.)

---

## Additional Resources

- [GitHub: Managing commit signature verification](https://docs.github.com/en/authentication/managing-commit-signature-verification)
- [GitLab: Signing commits with GPG](https://docs.gitlab.com/ee/user/project/repository/gpg_signed_commits/)
- [Git Documentation: Signing Your Work](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work)

---

**Last Updated**: June 25, 2025  
**Status**: ✅ Tested on macOS with GPG and SSH signing
