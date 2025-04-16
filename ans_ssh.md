Вот полноценная Ansible-роль для управления SSH-ключами на нескольких хостах, включая проверку, добавление и удаление ключей.

## Структура роли

```
roles/ssh_key_management/
├── defaults
│   └── main.yml       # переменные по умолчанию
├── tasks
│   └── main.yml       # основные задачи
├── templates
│   └── authorized_keys.j2  # шаблон для authorized_keys (опционально)
└── README.md
```

## 1. defaults/main.yml

```yaml
---
# Пользователь, для которого управляем ключами
ssh_user: "{{ ansible_user }}"

# SSH-ключи для добавления (список)
ssh_public_keys_to_add: []

# SSH-ключи для удаления (список)
ssh_public_keys_to_remove: []

# Строгое соответствие (удаляет все ключи, кроме указанных в ssh_public_keys_to_add)
strict_mode: false

# Права для .ssh директории и authorized_keys
ssh_dir_mode: "0700"
authorized_keys_mode: "0600"
```

## 2. tasks/main.yml

```yaml
---
- name: Ensure .ssh directory exists
  ansible.builtin.file:
    path: "/home/{{ ssh_user }}/.ssh"
    state: directory
    mode: "{{ ssh_dir_mode }}"
    owner: "{{ ssh_user }}"
    group: "{{ ssh_user }}"
  when: ssh_user != 'root'

- name: Ensure .ssh directory exists for root
  ansible.builtin.file:
    path: "/root/.ssh"
    state: directory
    mode: "{{ ssh_dir_mode }}"
  when: ssh_user == 'root'

- name: Get current authorized_keys content
