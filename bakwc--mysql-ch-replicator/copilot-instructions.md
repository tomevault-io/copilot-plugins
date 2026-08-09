## mysql-ch-replicator

> 1. Use following command to run tests:

1. Use following command to run tests:

docker exec -w /app/ -it tests-replicator-1 python3 -m pytest -v -s tests/ -k test_your_test_name

2. NEVER CREATE CONFIG FILES IN RUNTIME!!!!!!. Always create a real config files. Use log level info for newly added config files.

3. Dont create analyzes md files.

4. DOUBLE CHECK YOU FOLLOWED EVERY RULE EVERY TIME !!!!!!

---
> Source: [bakwc/mysql_ch_replicator](https://github.com/bakwc/mysql_ch_replicator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
