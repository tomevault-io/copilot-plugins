## artifacts-ansible-roles-set-hostname

> Rola Ansible `set-hostname` — ustawia hostname serwera i aktualizuje `/etc/hosts` (wpis `127.0.1.1`). Obsługuje Debian, Ubuntu, AlmaLinux (EL 7/8/9), Alpine.

# CLAUDE.md

## Projekt
Rola Ansible `set-hostname` — ustawia hostname serwera i aktualizuje `/etc/hosts` (wpis `127.0.1.1`). Obsługuje Debian, Ubuntu, AlmaLinux (EL 7/8/9), Alpine.

## Struktura
```
defaults/main.yml   — domyślne wartości zmiennych
vars/main.yml       — zmienne wewnętrzne
tasks/main.yml      — logika roli (2 taski)
handlers/main.yml   — handler reboot
meta/main.yml       — metadane Ansible Galaxy
```

## Konwencje kodu

### Ansible
- FQCN dla modułów (`ansible.builtin.hostname`, nie `hostname`)
- Każdy plik YAML zaczyna się od `---`
- Taski mają `name` po angielsku

### Zmienne
- Zmienne wejściowe mają prefiks `in_` (np. `in_hostname_fqdn`, `in_disable_reboot`)
- Format snake_case

### Git
- Conventional Commits: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `ci:`
- Branche feature: `feat/<nazwa>`
- Repozytorium na GitLab — Merge Requesty

## Język
- Kod (zmienne, taski, handlery) — angielski
- Dokumentacja (README, CONTRIBUTING) — polski

## Polecenia
Brak dedykowanych skryptów build/test w repozytorium. Rola jest testowana zewnętrznie (Molecule — plik `molecule_test_dev.sh` w `.gitignore`).

---
> Source: [rachuna-net-pl/artifacts.ansible-roles.set-hostname](https://github.com/rachuna-net-pl/artifacts.ansible-roles.set-hostname) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
