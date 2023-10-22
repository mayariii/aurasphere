<script lang="ts">
	import LandingScene from '$lib/components/scenes/LandingScene.svelte';
	import { Canvas } from '@threlte/core';
</script>

<Canvas>
	<LandingScene />
</Canvas>

<svelte:head>
	<title>aurasphere 🌙</title>
	<meta name="description" content="a little digispace for when you need a moment of zen ✨">
</svelte:head>

<span
	class="fixed top-0 left-0 text-white bg-white/20 p-2 m-2 rounded-md text-xs text-center max-w-[350px]">
	<p>welcome to the aurasphere ~</p>
	<p>a little digispace for when you need a moment of zen ✨</p>
</span>
