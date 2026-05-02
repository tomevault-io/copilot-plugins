## tutorial-generator-v2

> This project is a web application which generates static HTML pages and serves them in a dynamic frontend


## Project Guidelines

**Purpose:**
This project is a web application which generates static HTML pages and serves them in a dynamic frontend

**Repo structure**
- Backend
    - Python
    - entry point is /backend/app.py
    - FastAPI uvicorn server
- Frontend
    - React with vite dev server
    - entry point is /frontend/src/App.tsx
    - tailwind css and shadcn

## Python Guidelines

We follow very particular formatting guidelines.
- always use type hints in function parameters. Put no space arount the colon, but always have space around equal signs
- break long lists of parameters or entries into one per line
- indent parentheses so that the closing one aligns with the name of the assigned variable
- dicts with a single entry can be written on one line. When defining an empty dict, add a comment with an example entry.
- use complete docstrings for helper functions that are called in multiple places. Include args and return values, omit exceptions.

Good example:
```python
def send_tts(self, message:str, speaker:str, cancel_upon_interruption:bool = True):
    self.send_message(
        event = Events.TTS_REQUEST,
        data = {
            "message": message,
            "speaker": speaker,
            "cancel": cancel_upon_interruption
        }
    )
```


## Frontend Guidelines

Apply similar formatting style to javascript and typescript files as described in the python section above.

**Use Shadcn components when possible!** The following components are available in `/src/components/ui`: Accordion, Alert Dialog, Alert, Aspect Ratio, Avatar, Badge, Breadcrumb, Button Group, Button, Calendar, Card, Carousel, Chart, Checkbox, Collapsible, Combobox, Command, Context Menu, Data Table, Date Picker, Dialog, Drawer, Dropdown Menu, Empty, Field, Form, Hover Card, Input Group, Input, Item, Kbd, Label, Menubar, Native Select, Navigation Menu, Pagination, Popover, Progress, Radio Group, Resizable, Scroll Area, Select, Separator, Sheet, Sidebar, Skeleton, Slider, Sonner, Spinner, Switch, Table, Tabs, Textarea, Toast, Toggle Group, Toggle, Tooltip

---
> Source: [groenerfelix/tutorial-generator-v2](https://github.com/groenerfelix/tutorial-generator-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
