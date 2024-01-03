# Linuxbrew (`ctorgalson.linuxbrew`)

Manually installs linuxbrew, brew packages, and taps on Ubuntu/Debian to avoid
piping a shell script to `sh` :)

## Requirements

No special requirements.

## Role Variables

### Vars

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `lb__prefix`            | string | `/home/linuxbrew/.linuxbrew`                           | Location for all `brew` related files. |
| `lb__brew`              | string | `{{ lb__prefix }}/bin/brew`                            | Path to the `brew` binary. |
| `lb__homebrew_dir`      | string | `{{ lb__prefix }}/Homebrew`                            | Path to the Homebrew repo directory. |
| `lb__homebrew_core_dir` | string | `{{ lb__prefix }}/Homebrew/Library/Taps/homebrew-core` | Path to the Homebrew core repo directory. |
| `lb__directories      ` | list   | See `vars/main.yml`                                    | List of directories to be created in `lb__prefix` dir. |
| `lb__repos`             | list   | See `vars/main.yml`                                    | List of repos to be cloned during install. Each item must have `repo`, `dest`, and `version` properties suitable for `ansible.builtin.git`. |
| `lb__dependencies`      | list   | See `vars/main.yml`                                    | List of apt packages required for `brew` install and use. |

### Defaults

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `lb__owner`                                    | string  | `{{ ansible_user }}` | The name of the owner for the `{{ lb__prefix }}` directory and contents. |
| `lb__group`                                    | string  | `{{ ansible_user }}` | The name of the group for the `{{ lb__prefix }}` directory and contents. |
| `lb__shell_configuration_tasks`                | list    | `[]`                 | A list of paths to Ansible task include files to be run after the basic installation. |
| `lb__update_homebrew_when_installing_packages` | boolean | `true`               | Whether or not to update `brew` when installing new pacakges. |
| `lb__upgrade_all_when_installing_packages`     | boolean | `true`               | Whether or not to upgrade Linuxbrew package when installing new pacakges. |
| `lb__packages`                                 | list    | `[]`                 | A list of Linuxbrew packages to install. Each item must specify a `name` property, and can have optional `state`, `path`, and `install_options` properties suitable for `ansible.community.homebrew`. | 
| `lb__tabs`                                     | list    | `[]`                 | A list of Linuxbrew taps to install. Each item must specify a `name` property, and can have `state`, `path`, and `url` properties suitable for `ansible.community.homebrew_tap`. |

## Dependencies

This role relies on the `ansible.community` collection for the `homebrew` and
`homebrew_tap` modules.


## Example Playbook

    - hosts: servers
      become: true
      
      vars:
        lb__owner: "ctorgalson"
        lb__group: "{{ lb__owner }}"
        lb__packages:
          - name: "bottom"
            state: "present"
          - name: "starship"
            state: "present"

      tasks:
         - name: "Install and configure Linuxbrew, packages, and taps."
           ansible.builtin.import_role:
             name: "ctorgalson.linuxbrew"

## License

GPL-3.0-only
