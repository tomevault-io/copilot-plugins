## python-flask-json-guide-cursorrules-prompt-file

> Cursor rules for Python Flask development with JSON guide.

This project is heavily reliant on our custom Drawscape Factorio python module.

Here is code examples of how to use the module:

```python
from drawscape_factorio import create as createFactorio
from drawscape_factorio import importFUE5

with open('/path/to/exported-entities.json', 'r') as file:
    json_data = json.load(file)
    data = importFUE5(json_data)
    result = createFactorio(data, {
        'theme_name': 'default',
        'color_scheme': 'main',
        'show_layers': ['assets', 'belts', 'walls', 'rails', 'electrical', 'spaceship']
    })

with open(output_file_name, 'w') as f:
    f.write(result['svg_string'])

---
> Source: [XD3an/awesome-ai-coding-all-in-one](https://github.com/XD3an/awesome-ai-coding-all-in-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
