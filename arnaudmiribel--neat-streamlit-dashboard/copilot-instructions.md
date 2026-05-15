## streamlit

> Streamlit tips and good practice


General:
- Documentation for latest version: @https://docs.streamlit.io/llms-full.txt or @https://docs.streamlit.io/develop/api-reference
- Use context7 MCP to fetch the latest documentation if available
- When calling `st.dataframe`:
	- Use `column_config` to make labels look nicer
	- Don't forget `st.ProgressColumn` for percentage columns
	- Set `hide_index=True` if the index shows no value
	- If many columns, you likely want to use `use_container_width=True`
- When making a multipage app:
	- Define `page = st.navigation(...)` in the main module (usually `streamlit_app.py`)
	- Underlying pages should have dedicated files in the `app_pages/` directory
	- You can display the page title using `page.title` and `page.icon` before calling `page.run()`
	- You should store global variables in `st.session_state` in the main module so pages can access them
- When making a chart:
	- If easy, prefer the native charts st.line_chart, st.scatter_chart, st.bar_chart
	- Else, go for Altair or Plotly (mostly Express)
	- Don't forget to give a human readable label for x, y
- Use `st.divider()` instead of `st.markdown("---")` for visual separation.

Testing:
- Do not try and run the app - it's likely to be running already

Layout and customization:
- Stick to native Streamlit commands as much as possible, unless told otherwise
- You can update a specific Streamlit component style with `key='foo'` in CSS with class `st-key-foo`
- You can edit `.streamlit/config.toml` for native ways to tune the look and feel of your app
- Don't use too many columns in st.columns. If you do, remember to configure the `vertical_alignment` properly
- Remember you can use `@st.dialog` to save some real estate on the UI 
- When adding html, prefer st.html over st.markdown

Performance:
- Use `@st.cache_data` to cache heavy data computations
- Use `@st.cache_resource` to cache resources like DB/API connectors
- Use `@st.fragment` as a decorator for functions/UI pieces that are self-sufficient

---
> Source: [arnaudmiribel/neat-streamlit-dashboard](https://github.com/arnaudmiribel/neat-streamlit-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
