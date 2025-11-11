# Ansible Vault Setup Guide

Quick reference guide for setting up and using Ansible Vault with this project.

## Initial Setup (First Time)

### Step 1: Create Vault Password File

```bash
# Copy the example
cp .vault_pass.example .vault_pass

# Add your secure password (use a strong, random password)
echo "your-secure-vault-password-123" > .vault_pass

# Secure the file permissions
chmod 600 .vault_pass
```

**Important:** Keep this password safe! You'll need it to decrypt the vault files.

### Step 2: Set Up Vault Variables

**Option A: Create from example**

```bash
# Copy the example vault file
cp group_vars/splunk_servers/vault.yml.example group_vars/splunk_servers/vault.yml

# Edit with your actual credentials
nano group_vars/splunk_servers/vault.yml

# Encrypt the file
ansible-vault encrypt group_vars/splunk_servers/vault.yml
```

**Option B: Create encrypted file directly**

```bash
# This will prompt for the vault password and open an editor
ansible-vault create group_vars/splunk_servers/vault.yml
```

Add your credentials:

```yaml
---
splunk_service_user: "your_splunk_username"
splunk_service_password: "your_secure_password"
```

## Common Operations

### View Encrypted Content

```bash
ansible-vault view group_vars/splunk_servers/vault.yml
```

### Edit Encrypted Content

```bash
ansible-vault edit group_vars/splunk_servers/vault.yml
```

### Change Vault Password

```bash
ansible-vault rekey group_vars/splunk_servers/vault.yml
```

### Decrypt File (for manual editing)

```bash
# Decrypt
ansible-vault decrypt group_vars/splunk_servers/vault.yml

# Make changes with any editor
nano group_vars/splunk_servers/vault.yml

# Re-encrypt
ansible-vault encrypt group_vars/splunk_servers/vault.yml
```

### Encrypt Individual Strings

```bash
ansible-vault encrypt_string 'MySecretValue' --name 'my_variable'
```

## Running Playbooks with Vault

### Method 1: Using vault_password_file in ansible.cfg (Automatic)

If `vault_password_file = .vault_pass` is set in `ansible.cfg`:

```bash
ansible-playbook cert_install_withrole.yml
```

### Method 2: Specify password file explicitly

```bash
ansible-playbook cert_install_withrole.yml --vault-password-file .vault_pass
```

### Method 3: Interactive password prompt

```bash
ansible-playbook cert_install_withrole.yml --ask-vault-pass
```

### Method 4: Environment variable

```bash
export ANSIBLE_VAULT_PASSWORD_FILE=/path/to/.vault_pass
ansible-playbook cert_install_withrole.yml
```

## Verifying Vault Setup

### Check if file is encrypted

```bash
# Encrypted files start with $ANSIBLE_VAULT;
head -1 group_vars/splunk_servers/vault.yml
```

### Test vault decryption

```bash
ansible-vault view group_vars/splunk_servers/vault.yml
```

### Test playbook with vault

```bash
# Syntax check
ansible-playbook cert_install_withrole.yml --syntax-check

# Dry run
ansible-playbook cert_install_withrole.yml --check
```

## Troubleshooting

### Error: "Decryption failed"

**Cause:** Wrong vault password

**Solution:** Ensure you're using the correct password in `.vault_pass` or entering it correctly

### Error: "vault password file not found"

**Cause:** `.vault_pass` file doesn't exist

**Solution:** Create the file with your vault password:
```bash
echo "your-vault-password" > .vault_pass
chmod 600 .vault_pass
```

### File is not encrypted

**Cause:** Forgot to encrypt the vault.yml file

**Solution:**
```bash
ansible-vault encrypt group_vars/splunk_servers/vault.yml
```

## Security Best Practices

1. ✅ **Never commit** `.vault_pass` to git (it's in `.gitignore`)
2. ✅ **Use strong passwords** for vault (minimum 20 characters, random)
3. ✅ **Restrict file permissions**: `chmod 600 .vault_pass`
4. ✅ **Rotate passwords regularly** using `ansible-vault rekey`
5. ✅ **Use different vault passwords** for different environments
6. ✅ **Keep backup** of vault password in a secure password manager
7. ✅ **Never store plain text credentials** in version control

## Quick Reference

| Command | Purpose |
|---------|---------|
| `ansible-vault create <file>` | Create new encrypted file |
| `ansible-vault encrypt <file>` | Encrypt existing file |
| `ansible-vault decrypt <file>` | Decrypt file |
| `ansible-vault edit <file>` | Edit encrypted file |
| `ansible-vault view <file>` | View encrypted file |
| `ansible-vault rekey <file>` | Change vault password |
| `ansible-vault encrypt_string <string>` | Encrypt a single string |

## File Locations

- Vault password file: `.vault_pass` (not committed to git)
- Vault password example: `.vault_pass.example`
- Encrypted variables: `group_vars/splunk_servers/vault.yml`
- Vault template: `group_vars/splunk_servers/vault.yml.example`
- Ansible config: `ansible.cfg`
