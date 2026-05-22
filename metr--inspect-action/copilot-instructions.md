## cli-structure

> - `cli.py` contains command definitions and presentation logic only

# CLI Development Structure

## Architecture
- `cli.py` contains command definitions and presentation logic only
- Each command's business logic lives in a separate file (e.g., `login.py`, `view.py`, `runs.py`)
- Keep `cli.py` focused on:
  - Command definitions with Click decorators
  - Argument parsing and validation
  - Output formatting and presentation
  - User interaction logic

## Click Patterns
- Use `click.echo()` for output, not print()
- Import business logic in-line, only in the command handlers where it is used

Example structure:
```python
# In cli.py
@click.command()
@click.option('--format', type=click.Choice(['json', 'table']))
def my_command(format):
    import hawk.my_command

    result = my_command()
    click.echo(format_output(result, format))
```

---
> Source: [METR/inspect-action](https://github.com/METR/inspect-action) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
