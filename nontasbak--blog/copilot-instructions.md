## blog

> If the user asks you to convert a markdown file to an MDX document for a blog post, you need to do the following:

If the user asks you to convert a markdown file to an MDX document for a blog post, you need to do the following:

1. Convert any instance of `exegol-<container_name> /workspace # <command>` to `nontas@local:~$ <command>`. If the user changes directories (meaning `/workspace` changes) change the directory accordingly.
2. Convert any notes/warning/error block to a 
```
<Callout title="Title" type="info">
content here
</Callout>
```
don't add the 3 backticks, and make sure the "Callout" React component is written in separate lines, as above (not inline).
Other available types are "warn" and "error".
3. Change the path of the images to `/images/blog/<machine_name>/<image_name>.png`. Change them to ![short title](image path). Give very vague descriptions of the image, no need to guess too hard. The `machine_name` is seen from the title (make sure it's all lowercase, no dashes or underscores).
4. The code/console blocks should have a short title, add it right next to ```console title="Title".
For example:
```console title="Nmap scan"
nmap -sV -sC -oN nmap.txt 10.10.10.10
<output of nmap>
```
Give only short descriptive titles. If there are multiple things happening in the block, give a very vague title. If something very simple is happening (and very short), don't give a title.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NontasBak) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
