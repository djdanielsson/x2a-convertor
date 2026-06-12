## Ansible Content Best Practices (Red Hat CoP / ai-forge)

These rules apply to ALL generated Ansible content. They are sourced from the
Red Hat Communities of Practice automation good practices and the Ansible
community ai-forge project.

### YAML Formatting

1. **Two-space indentation.** Never use tabs.
2. **`.yml` extension**, not `.yaml`.
3. **YAML-style module arguments** — never use `key=value` inline style.
4. **`true`/`false` booleans** — never `yes`/`no`/`True`/`False`.
5. **Blank line between every task** — each `- name:` block must be separated
   by exactly one blank line from the previous task. This is critical for
   readability.
6. **`---` document start marker** — every YAML file must begin with `---`.
7. **Lines under 120 characters.** Use YAML folding or list-style `when:` for
   long expressions.
8. **Folded scalars use `>-`** (no trailing newline), not `>`
9. **Quote Jinja2 variables:** `"{% raw %}{{ variable }}{% endraw %}"` — never bare `{% raw %}{{ variable }}{% endraw %}`
10. **Long `when:` as list** — Ansible automatically ANDs list elements:
    ```yaml
    when:
      - ansible_facts['os_family'] == 'RedHat'
      - ansible_facts['distribution_major_version'] | int >= 8
      - myapp_enabled
    ```

### Task Writing

1. **Name every task** in imperative mood: "Install nginx", not "nginx installation".
2. **FQCN for all modules** — `ansible.builtin.copy`, never `copy`.
3. **Explicit `state:`** — always specify it; defaults vary across modules.
4. **`loop:` not `with_*`** — `with_items`, `with_dict` etc. are deprecated.
5. **Prefer modules over `command`/`shell`/`raw`:**
   - First choice: purpose-built module
   - Second: `ansible.builtin.command` (no shell features)
   - Last resort: `ansible.builtin.shell` (when pipes, redirects needed)
6. **`failed_when:` not `ignore_errors: true`** — define exactly which failures
   are acceptable.
7. **`changed_when:` on command/shell** — they always report changed otherwise.
8. **`delegate_to:` not `local_action:`**.
9. **`block`/`rescue`/`always`** for structured error handling.
10. **Sub-task prefixes** — tasks in included files should be prefixed with the
    filename: `install | Install required packages`.

### Task Componentization

Split complex roles into component task files:

```yaml{% raw %}
# tasks/main.yml
- name: Include install tasks
  ansible.builtin.include_tasks:
    file: "{{ role_path }}/tasks/install.yml"

- name: Include configure tasks
  ansible.builtin.include_tasks:
    file: "{{ role_path }}/tasks/configure.yml"

- name: Include service tasks
  ansible.builtin.include_tasks:
    file: "{{ role_path }}/tasks/service.yml"
{% endraw %}```

### Handler Rules

1. **Prefix handler names with role name:** `webserver | Restart nginx`
2. **Use `listen:` for decoupling** — tasks notify a logical name, multiple
   handlers can respond.
3. **Validate config before restart** when applicable.
4. **Handlers are for change-triggered actions only.**

### Variable Conventions

1. **`snake_case`** for all variable names.
2. **Role-prefix all variables:** `webserver_port`, not `port`.
3. **Internal variables use `__` prefix:** `__webserver_config_path`.
4. **`defaults/main.yml`** = user-facing variables with safe defaults (lowest
   precedence).
5. **`vars/main.yml`** = internal constants only (high precedence, hard to
   override).
6. **`ansible_facts['os_family']` bracket notation** — not `ansible_os_family`.

### Template Rules

1. **`{% raw %}{{ ansible_managed | comment }}{% endraw %}`** header in every generated template.
2. **`backup: true`** on template tasks.
3. **No timestamps** — dynamic dates break idempotency.

### Content Trust Order

Before writing custom automation, prefer existing content in this order:
1. `ansible.builtin`
2. Vendor-supported collections/roles
3. Content from verified authors
4. General Galaxy content
5. Custom content only as a last resort

### Anti-Patterns (NEVER do these)

- `key=value` inline module arguments
- `yes`/`no` booleans
- Short module names without FQCN
- Omitting `state:` parameter
- `with_items` instead of `loop`
- `ignore_errors: true` instead of `failed_when`
- Mixing `roles:` and `tasks:` in a playbook
- User-facing variables in `vars/main.yml`
- Unprefixed role variables
- Missing `ansible_managed` in templates
- `ansible_os_family` instead of `ansible_facts['os_family']`
- `shell` when `command` suffices
- `.yaml` extension instead of `.yml`
- No blank line between tasks
