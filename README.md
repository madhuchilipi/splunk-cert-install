# Splunk Certificate Installation with Ansible

An Ansible playbook and role for automated Splunk certificate installation on Linux and Windows systems with integrated Ansible Vault for secure credential management.

## Features

- Automated certificate installation for Splunk
- Cross-platform support (Linux and Windows)
- Secure credential management with Ansible Vault
- Certificate backup before installation
- Error handling and recovery
- Role-based structure for easy reuse

## Prerequisites

- Ansible 2.9 or higher
- Target systems with Splunk installed
- SSH access to Linux hosts (for remote installation)
- WinRM configured for Windows hosts (for remote installation)
- Appropriate sudo/admin privileges

> **Security Notice:** This repository includes a vault password file (`.vault_pass`) with password `test_vault_password_123` for testing/demonstration purposes. **In production, you MUST create your own secure vault password** using a strong, randomly generated password.

## Quick Start

### 1. Clone the Repository

```bash
git clone <repository-url>
cd splunk-cert-install
```

### 2. Set Up Ansible Vault

Create a vault password file:

```bash
# Copy the example file
cp .vault_pass.example .vault_pass

# Edit it with your secure password
nano .vault_pass

# Secure the password file
chmod 600 .vault_pass
```

### 3. Configure Vault Variables

Create and encrypt your vault file with sensitive credentials:

```bash
# Copy the example vault file
cp group_vars/splunk_servers/vault.yml.example group_vars/splunk_servers/vault.yml

# Edit with your actual credentials
nano group_vars/splunk_servers/vault.yml

# Encrypt the file
ansible-vault encrypt group_vars/splunk_servers/vault.yml
```

Alternatively, create an encrypted file directly:

```bash
ansible-vault create group_vars/splunk_servers/vault.yml
```

Add your credentials in the encrypted file:

```yaml
---
splunk_service_user: "your_splunk_user"
splunk_service_password: "your_secure_password"
```

### 4. Configure Inventory

Edit `splunk_inventory.ini` with your Splunk servers:

```ini
[splunk_servers]
splunk-server1.example.com
splunk-server2.example.com
localhost ansible_connection=local
```

### 5. Update Certificate Paths

Edit `splunk_cert_role/defaults/main.yml` to match your environment:

```yaml
cert_dir: /opt/splunk/etc/auth/certs
cert_source_path: /path/to/your/certificate.pem
cert_dest_path: /opt/splunk/etc/auth/certs/splunk_cert.pem
cert_backup_path: /opt/splunk/etc/auth/certs/splunk_cert.pem.bak
```

### 6. Run the Playbook

With vault password file configured in ansible.cfg:

```bash
ansible-playbook cert_install_withrole.yml
```

Or specify the vault password file explicitly:

```bash
ansible-playbook cert_install_withrole.yml --vault-password-file .vault_pass
```

Or prompt for the vault password interactively:

```bash
ansible-playbook cert_install_withrole.yml --ask-vault-pass
```

## Ansible Vault Usage

### Working with Vault Files

**View encrypted vault file:**

```bash
ansible-vault view group_vars/splunk_servers/vault.yml
```

**Edit encrypted vault file:**

```bash
ansible-vault edit group_vars/splunk_servers/vault.yml
```

**Decrypt vault file (for editing manually):**

```bash
ansible-vault decrypt group_vars/splunk_servers/vault.yml
```

**Re-encrypt after editing:**

```bash
ansible-vault encrypt group_vars/splunk_servers/vault.yml
```

**Change vault password:**

```bash
ansible-vault rekey group_vars/splunk_servers/vault.yml
```

### Encrypting Individual Variables

You can also encrypt individual variables instead of the entire file:

```bash
# Encrypt a string and add to vault.yml
ansible-vault encrypt_string 'MySecretPassword' --name 'splunk_service_password'
```

This generates output you can paste directly into your vault.yml:

```yaml
splunk_service_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          ...encrypted content...
```

### Vault Password File Options

There are multiple ways to provide the vault password:

1. **Using a password file (Recommended for automation):**
   - Set in `ansible.cfg`: `vault_password_file = .vault_pass`
   - Or use command line: `--vault-password-file .vault_pass`

2. **Interactive prompt (Recommended for manual runs):**
   ```bash
   ansible-playbook cert_install_withrole.yml --ask-vault-pass
   ```

3. **Environment variable:**
   ```bash
   export ANSIBLE_VAULT_PASSWORD_FILE=/path/to/.vault_pass
   ansible-playbook cert_install_withrole.yml
   ```

## Project Structure

```
.
├── ansible.cfg                          # Ansible configuration
├── splunk_inventory.ini                 # Inventory file
├── cert_install.yml                     # Standalone playbook
├── cert_install_withrole.yml            # Playbook using role
├── group_vars/
│   └── splunk_servers/
│       ├── vault.yml                    # Encrypted vault file
│       └── vault.yml.example            # Example template
├── splunk_cert_role/                    # Ansible role
│   ├── defaults/
│   │   └── main.yml                     # Default variables
│   ├── files/
│   │   └── dummy_cert.pem              # Example certificate
│   └── tasks/
│       ├── main.yml                     # Main task file
│       ├── linux.yml                    # Linux-specific tasks
│       └── windows.yml                  # Windows-specific tasks
└── README.md                            # This file
```

## Playbooks

### cert_install.yml

Standalone playbook that directly installs certificates without using a role.

### cert_install_withrole.yml

Uses the `splunk_cert_role` for modular and reusable certificate installation.

## Role Variables

Variables can be set in `splunk_cert_role/defaults/main.yml`:

| Variable | Description | Default |
|----------|-------------|---------|
| `cert_dir` | Directory for certificates | `/root/2025/splunk-install/splunk_cert_install/etc/auth/certs` |
| `cert_source_path` | Source certificate path | `/root/2025/dummy_cert.pem` |
| `cert_dest_path` | Destination certificate path | `{{ cert_dir }}/dummy_cert.pem` |
| `cert_backup_path` | Backup certificate path | `{{ cert_dest_path }}.bak` |

Vault-encrypted variables in `group_vars/splunk_servers/vault.yml`:

| Variable | Description |
|----------|-------------|
| `splunk_service_user` | Splunk service account username |
| `splunk_service_password` | Splunk service account password |

## Security Best Practices

1. **Never commit unencrypted credentials** to version control
2. **Add `.vault_pass` to `.gitignore`** (already configured)
3. **Use strong vault passwords** (minimum 20 characters, random)
4. **Rotate vault passwords regularly** using `ansible-vault rekey`
5. **Restrict vault password file permissions**: `chmod 600 .vault_pass`
6. **Use separate vault files** for different environments (dev, staging, prod)
7. **Enable `no_log: true`** in tasks that handle sensitive data

## Troubleshooting

### Vault password incorrect

```
ERROR! Decryption failed (no vault secrets were found that could decrypt)
```

**Solution:** Ensure you're using the correct vault password file or entering the correct password.

### Cannot find vault password file

```
ERROR! The vault password file .vault_pass was not found
```

**Solution:** Create the `.vault_pass` file with your vault password.

### Permission denied on certificate files

**Solution:** Ensure the playbook runs with appropriate privileges (`become: true`) and the certificate paths are correct.

### Windows WinRM connection issues

**Solution:** Ensure WinRM is properly configured on Windows hosts. See [Ansible Windows Setup](https://docs.ansible.com/ansible/latest/os_guide/windows_setup.html).

## Testing

To test the playbook without making changes:

```bash
ansible-playbook cert_install_withrole.yml --check --diff
```

To test on a single host:

```bash
ansible-playbook cert_install_withrole.yml --limit splunk-server1.example.com
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

[Add your license information here]

## Support

For issues and questions, please open an issue in the repository.

## References

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible Vault Documentation](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
- [Splunk Documentation](https://docs.splunk.com/)
