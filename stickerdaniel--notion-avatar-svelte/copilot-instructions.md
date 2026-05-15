## notion-avatar-svelte

> Usage instructions for the notion svelte avatar editor

To render the Avatar Editor:

<!-- src/routes/+page.svelte -->
<script lang="ts">
	import AvatarCreator from '$lib/components/ui/avatar-editor/avatar-editor.svelte';
</script>

<AvatarCreator />

To access the avatar context anywhere in your app add to your +layout.svelte:

<!-- src/routes/+layout.svelte -->
<script lang="ts">
	import '../app.css';
	import { avatarContext } from '$lib/components/ui/avatar-editor/avatarContext';
	import { AvatarStoreClass } from '$lib/components/ui/avatar-editor/AvatarStore.svelte';

	let { children } = $props();

	// Instantiate and set the AvatarStore in the context
	// This makes it available to all child components within this layout.
	avatarContext.set(new AvatarStoreClass());
</script>

{@render children()}

then you can use in any component 

<script lang="ts">
	import * as Avatar from '$lib/components/ui/avatar';
	import { avatarContext } from '$lib/components/ui/avatar-editor/avatarContext';
	// Get the shared avatar store
	const avatar = avatarContext.get();
</script>

<pre class="text-xs">{JSON.stringify(JSON.parse(avatar.configJSON), null, 2)}</pre>
<Avatar.Root>
	<Avatar.Image src={avatar.svgDataUrl} />
	<Avatar.Fallback>Avatar</Avatar.Fallback>
</Avatar.Root>

---
> Source: [stickerdaniel/notion-avatar-svelte](https://github.com/stickerdaniel/notion-avatar-svelte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
