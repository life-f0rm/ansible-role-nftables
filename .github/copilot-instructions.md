# Copilot Instructions for ansible-role-nftables

## Project Overview
This is an Ansible role designed to manage nftables firewall configuration using template files. The role provides a flexible and maintainable way to configure Linux firewall rules using nftables, the modern replacement for iptables.

## Repository Structure
This Ansible role follows the standard Ansible Galaxy role structure:
- `tasks/` - Contains the main task files for the role
- `handlers/` - Contains handlers triggered by tasks
- `templates/` - Contains Jinja2 templates for nftables configuration files
- `defaults/` - Contains default variables for the role
- `vars/` - Contains other variables for the role
- `meta/` - Contains role metadata and dependencies
- `files/` - Contains static files to be deployed
- `tests/` or `molecule/` - Contains test scenarios

## Technology Stack
- **Ansible**: Automation tool for configuration management
- **nftables**: Modern Linux packet filtering framework
- **Jinja2**: Template engine used by Ansible
- **YAML**: Configuration file format

## Coding Standards

### Ansible Best Practices
- Use fully qualified collection names (FQCN) for modules (e.g., `ansible.builtin.template`)
- Follow YAML syntax with 2-space indentation
- Use meaningful variable names with the role prefix (e.g., `nftables_`)
- Include `become: true` when tasks require root privileges
- Use handlers for service restarts and reloads
- Always validate configurations before applying them

### Variable Naming
- Prefix all role variables with `nftables_` to avoid conflicts
- Use snake_case for variable names
- Provide sensible defaults in `defaults/main.yml`
- Document all variables with comments

### Templates
- Use Jinja2 templates for nftables configuration files
- Include header comments indicating the file is managed by Ansible
- Use appropriate filters for security (e.g., `| regex_escape` for user input)
- Keep templates readable and well-documented

### Task Organization
- Break complex tasks into logical files in `tasks/`
- Use `import_tasks` for static includes, `include_tasks` for dynamic
- Add descriptive names to all tasks
- Use `tags` for selective task execution
- Include check mode support where possible

## nftables Specific Guidelines
- Validate nftables syntax before applying rules (use `nft -c -f`)
- Always have a rollback mechanism for firewall changes
- Test rules in a safe environment before production deployment
- Document rule purposes with comments in templates
- Follow nftables table/chain hierarchy (table → chain → rule)
- Use named sets and maps for efficient rule management
- Consider using atomic ruleset replacement for safety

## Security Considerations
- Never commit sensitive data (IPs, credentials) to the repository
- Use Ansible Vault for any sensitive variables
- Implement safe defaults that don't lock out administrators
- Always include SSH access rules before deploying firewall changes
- Test firewall configurations in isolated environments first
- Include emergency access mechanisms

## Testing
- Use Molecule for testing if available
- Test on various Linux distributions (Debian, Ubuntu, CentOS, Rocky Linux)
- Verify idempotency - running the role twice should not make changes
- Include both positive and negative test cases
- Test firewall rules with actual network traffic where possible

## Documentation
- Update README.md with role usage examples
- Document all variables in README.md
- Include example playbooks
- Add inline comments for complex logic
- Keep CHANGELOG.md updated with notable changes

## Git Workflow
- Write clear, descriptive commit messages
- Keep commits focused and atomic
- Reference issue numbers in commit messages where applicable
- Ensure all changes are properly tested before committing

## Common Patterns
```yaml
# Example task structure
- name: Deploy nftables configuration
  ansible.builtin.template:
    src: nftables.conf.j2
    dest: /etc/nftables.conf
    owner: root
    group: root
    mode: '0644'
    validate: 'nft -c -f %s'
  notify: Reload nftables

# Example handler
- name: Reload nftables
  ansible.builtin.systemd:
    name: nftables
    state: reloaded
```

## Tips for AI Assistance
- When modifying firewall rules, always consider the security implications
- Suggest validation steps for any configuration changes
- Recommend testing strategies for firewall modifications
- Consider compatibility across different Linux distributions
- Propose rollback mechanisms for critical changes
