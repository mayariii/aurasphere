<script lang="ts">
	import * as THREE from 'three';
	import { T, useFrame } from '@threlte/core';
	import { Environment, Float, OrbitControls, Text, interactivity } from '@threlte/extras';
	import { spring } from 'svelte/motion';
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';
	import AuraSphereText from '../models/AuraSphereText.svelte';
	import Friends from '../models/Friends.svelte';

	// enter button
	const scale = spring(1);
	let isButtonHovered = false;

	$: isButtonHovered
		? (document.body.style.cursor = 'pointer')
		: (document.body.style.cursor = 'default');

	// background particles

	let particles: THREE.Points;
	let particleRotationX = 0;
	let particleRotationY = 0;
	const geometry = new THREE.BufferGeometry();
	const vertices: number[] = [];
	const material = new THREE.PointsMaterial({ size: 2, sizeAttenuation: true });

	interactivity();

	onMount(() => {
		// set up background particles
		for (let i = 0; i < 10000; i++) {
			const x = (Math.random() - 0.5) * 2000;
			const y = (Math.random() - 0.5) * 2000;
			const z = (Math.random() - 0.5) * 2000;

			vertices.push(x, y, z);
		}

		geometry.setAttribute('position', new THREE.Float32BufferAttribute(vertices, 3));
		particles = new THREE.Points(geometry, material);
	});

	useFrame((_, delta) => {
		// animations 
		particleRotationX += delta * 0.005;
		particleRotationY += delta * 0.005;
	});
</script>

<!-- CAMERAS -->
<T.PerspectiveCamera makeDefault position={[15, 5, 50]} fov={15}>
	<OrbitControls enableZoom enableDamping autoRotateSpeed={0.5} target.y={1.5} />
</T.PerspectiveCamera>

<!-- LIGHTS -->

<T.DirectionalLight intensity={0.8} position.x={5} position.y={10} />

<T.AmbientLight intensity={0.5} />

<!-- ENVIRONMENT -->

{#if particles}
	<T.Points {geometry} {material} rotation.x={particleRotationX} rotation.y={particleRotationY} />
{/if}

<Environment path="/envmap//" files="light-bg.jpg" />

<!-- OBJECTS -->
<!-- text title -->
<T.Mesh position={[0.5, 3, 1]} envMapIntensity={11}>
	<AuraSphereText scale={0.5} />
	<T.MeshPhongMaterial shininess={1} />
</T.Mesh>

<!-- enter button -->
<T.Group
	on:click={() => goto('/breathe')}
	on:pointerenter={() => {
		scale.set(1.1);
		isButtonHovered = true;
	}}
	on:pointerleave={() => {
		scale.set(1);
		isButtonHovered = false;
	}}
	on:pointercancel={() => {
		scale.set(1);
		isButtonHovered = false;
	}}
	scale={$scale}>
	<T.Mesh position={[1.2, 1.5, 0.75]} scale={0.5} envMapIntensity={11}>
		<T.BoxGeometry args={[2, 1, 1]} />
		<Text text="enter" color="black" fontSize={0.5} position={[-0.55, 0.25, 0.55]} />
		<T.MeshStandardMaterial color="pink" envMapIntensity={2} />
	</T.Mesh>
</T.Group>

<!-- character -->
<Float floatIntensity={1} floatingRange={[0, 1]}>
	<T.Mesh scale={0.3} position={[-2, -0.4, 0.75]} rotation={[-0.1, 0.1, 0.2]} envMapIntensity={11}>
		<Friends />
	</T.Mesh>
</Float>
