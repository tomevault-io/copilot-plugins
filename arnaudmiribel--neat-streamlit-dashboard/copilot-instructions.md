## arnauds-opinionated-rules

> Arnaud's opinionated Streamlit & SQL rules


Streamlit opinions
- In general, prefer Material icons over emojis when building Streamlit apps
- Use emojis sparingly, only to add a very special touch
- Use `st.segmented_control` for a select widget with few options that fit in 1 line
- Don't use `st.radio(..., horizontal=True)` and prefer `st.segmented_control`
- Use `st.pills` for a multiselect with few options that fit in 1 line
- Use `st.selectbox` for a select widget with many options
- Prefer `st.caption` over `st.info` to not make the app too heavy
- Prefer sentence casing over title casing, even in titles or widget labels
- Metadata can be nicely shown using badges (`st.badge` or `:badge[foo]` if available, else `:green-background[:green[foo]]` in Markdown)
- Use dividers sparingly. You can use `st.markdown("")` instead to give some vertical spacing
- Don't use `st.sidebar` for anything else than navigation & app level filters/information
- Prefer st.write over other commands, such as st.markdown, unless you are using kwargs specific to those other commands

SQL opinions
- Use lowercase as much as you can

---
> Source: [arnaudmiribel/neat-streamlit-dashboard](https://github.com/arnaudmiribel/neat-streamlit-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
