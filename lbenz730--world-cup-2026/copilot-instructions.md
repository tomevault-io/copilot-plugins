## world-cup-2026

> - Always use exactly 1 space before and after `<-`, `~`, and `=` operators — never pad with extra spaces to align multiple lines

# Code Style

- Always use exactly 1 space before and after `<-`, `~`, and `=` operators — never pad with extra spaces to align multiple lines
- `if`/`else` always uses `{` `}` with the body on the next line — never inline; `if(cond) x else y` is not allowed; no space before `(` in `if()`/`for()`/`while()`
- Multi-line function calls: first argument on the same line as `(`, subsequent arguments indented to align with the first, closing `)` on the last argument's line — never a newline after `(` or `)` on its own line
- Long pipelines/chains start on a new line after the assignment:
  ```r
  result <-
    data %>%
    filter(...) %>%
    mutate(...)

# Plot Style

- Base theme is `theme_bw()`, set globally via `theme_set()` in `helpers.R` — source `helpers.R` in every plot script
- Titles and subtitles are centered: `hjust = 0.5`
- Font sizes: `plot.title` = 24, `plot.subtitle` = 20, `axis.title` = 16, `axis.text` = 12, `strip.text` = 14
- Default `legend.position = "none"`; override per-plot in a `theme()` call when a legend is needed

---
> Source: [lbenz730/world_cup_2026](https://github.com/lbenz730/world_cup_2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
