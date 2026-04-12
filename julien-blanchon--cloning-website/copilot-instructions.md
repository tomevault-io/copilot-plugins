## cloning-website

> You must use Iconify as much as possible for the icons and logos, and try not to use custom <svg> tags.

# Icon and logo with Iconify

You must use Iconify as much as possible for the icons and logos, and try not to use custom <svg> tags.

## pickapicon-mcp MCP server for Iconify search

The MCP server provides three main tools to help you work with icons:

Get Icon Repositories
This tool retrieves all available icon repository names, which you can use as prefixes for other tools.

Get Icons by Description and Prefix
This tool helps you find icons based on a text description:

Required inputs:
desc: Description of the icon you want
prefix (optional): Icon set prefix (defaults to the value in your environment configuration)
Get Icon Detail by Prefix and Name
This tool retrieves the actual SVG code for a specific icon:

Required inputs:
svg_name: The name of the SVG icon
prefix (optional): Icon set prefix (defaults to the value in your environment configuration)

## Iconify in Svelte

`@iconify/svelte` is installed in the project.

See the documentation here:
@https://iconify.design/docs/icon-components/svelte/

```svelte
<script lang="ts">
  import Icon from "@iconify/svelte";
</script>

<Icon icon="mdi:home" class="size-4 text-gray-500" />
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julien-blanchon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/julien-blanchon)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
