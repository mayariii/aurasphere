<script lang="ts">
	import AuraSphere from '$lib/components/scenes/AuraSphere.svelte';

	import { Canvas } from '@threlte/core';
</script>

<Canvas>
	<AuraSphere />
</Canvas>

<span
	class="fixed top-0 left-0 text-white bg-white/20 p-2 m-2 rounded-md text-xs text-center max-w-[350px]">
	<p>inhale/exhale in time with planet breathe's movement for a moment of calm ☁️</p>
	<br />
	<p>click planet to start/stop sound ✨</p>
	<br />
	<p>drag/zoom/pan to explore the aurasphere 🌙</p>
</span>
