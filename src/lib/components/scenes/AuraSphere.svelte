<script lang="ts">
	import * as THREE from 'three';
	import { T, useFrame } from '@threlte/core';
	import { Environment, Float, OrbitControls, interactivity, HTML } from '@threlte/extras';
	import { spring } from 'svelte/motion';
	import { onMount } from 'svelte';

	import Breathe from '../models/Breathe.svelte';
	import Rocket from '../models/Rocket.svelte';
	import Great from '../models/Great.svelte';

	// planets
	let scale = spring(1);
	let planetRotation = 0;

	// background particles
	let particles: THREE.Points;
	let particleRotationX = 0;
	let particleRotationY = 0;
	const geometry = new THREE.BufferGeometry();
	const vertices: number[] = [];
	const material = new THREE.PointsMaterial({ size: 2, sizeAttenuation: true });

	// galaxy particles
	let galaxyParticles: THREE.Points;
	let galaxyRotationZ = 0;

	const numGalaxyParticles = 5000;
	const spiralFactor = 2;
	const spiralRandomness = 0.4;
	const galaxyGeometry = new THREE.BufferGeometry();
	const positions = new Float32Array(numGalaxyParticles * 3);
	const colors = new Float32Array(numGalaxyParticles * 3);
	const galaxyMaterial = new THREE.PointsMaterial({
		size: 0.1,
		vertexColors: true,
		blending: THREE.AdditiveBlending
	});

	// audio
	let isPlaying = false;
	let audioElement: HTMLAudioElement;

	function playAudio() {
		if (audioElement) {
			audioElement.play();
		}
	}

	function pauseAudio() {
		if (audioElement) {
			audioElement.pause();
		}
	}

	// interactivity
	let isButtonHovered = false;

	$: isButtonHovered
		? (document.body.style.cursor = 'pointer')
		: (document.body.style.cursor = 'default');

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

		// set up galaxy particles
		for (let i = 0; i < numGalaxyParticles; i++) {
			const angle = i * spiralFactor;
			const randomness = (Math.random() - 0.5) * spiralRandomness;
			const radius = i * 0.05 + randomness;
			const x = radius * Math.cos(angle);
			const y = radius * Math.sin(angle);
			const z = (Math.random() - 0.5) * 10;

			positions[i * 3] = x;
			positions[i * 3 + 1] = y;
			positions[i * 3 + 2] = z;

			// gradient colours based on distance from the center
			const gradient = i / numGalaxyParticles;
			colors[i * 3] = gradient; // red channel
			colors[i * 3 + 1] = 0.9 - gradient; // green channel
			colors[i * 3 + 2] = 0.6 - gradient; // blue channel
		}

		galaxyParticles = new THREE.Points(galaxyGeometry, galaxyMaterial);
		galaxyGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
		galaxyGeometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));
	});

	useFrame((_, delta) => {
		// ANIMATIONS ✨
		// background particles
		particleRotationX += delta * 0.002;
		particleRotationY += delta * 0.002;

		// breathing planet animation
		let scaleNum;
		scaleNum = 1 + 0.1 * Math.sin(Date.now() / 1500);
		scale.set(scaleNum);

		// spiral galaxy rotation
		galaxyRotationZ += 0.005;

		// planet rotation
		planetRotation += delta * 0.2;
	});
</script>

<!-- CAMERAS -->

<T.PerspectiveCamera makeDefault position={[12, 5, 40]} fov={15}>
	<OrbitControls enableDamping />
</T.PerspectiveCamera>

<!-- LIGHTS -->

<T.DirectionalLight intensity={0.8} position.x={10} position.y={15} />

<T.AmbientLight intensity={0.8} />

<!-- light in galaxy -->
<T.PointLight intensity={0.9} position={[15, 0, -10]} />

<!-- light in planet -->
<T.PointLight intensity={0.8} distance={1.2} position={[-12, 5, -8]} />

<!-- ENVIRONMENT -->

<Environment path="/envmap/" files="light-bg.jpg" isBackground={false} />

{#if particles}
	<T.Points {geometry} {material} rotation.x={particleRotationX} rotation.y={particleRotationY} />
{/if}

{#if galaxyParticles}
	<T.Points
		geometry={galaxyGeometry}
		material={galaxyMaterial}
		position={[8, 3.52, -15]}
		scale={0.005}
		rotation.x={-Math.PI / 3}
		rotation.z={galaxyRotationZ / 2} />
	<T.Points
		geometry={galaxyGeometry}
		material={galaxyMaterial}
		position={[8, 3.52, -15]}
		scale={0.01}
		rotation.x={-Math.PI / 3}
		rotation.z={galaxyRotationZ / 2} />
	<T.Points
		geometry={galaxyGeometry}
		material={galaxyMaterial}
		position={[8, 3.5, -15]}
		scale={0.015}
		rotation.x={-Math.PI / 3}
		rotation.z={galaxyRotationZ} />
	<T.Points
		geometry={galaxyGeometry}
		material={galaxyMaterial}
		position={[8, 3.51, -15]}
		scale={0.018}
		rotation.x={-Math.PI / 3}
		rotation.z={galaxyRotationZ / 4} />
{/if}

<!-- AUDIO -->
<audio bind:this={audioElement} src="/sounds/meditation.mp3" preload="none" loop />

<!-- OBJECTS-->

<!-- breathing planet -->
<T.Group
	receiveShadow
	on:click={() => {isPlaying ? pauseAudio() : playAudio(); isPlaying = !isPlaying;}}
	on:pointerenter={() => {
		isButtonHovered = true;
	}}
	on:pointerleave={() => {
		isButtonHovered = false;
	}}
	on:pointercancel={() => {
		isButtonHovered = false;
	}}>
	<T.Mesh scale={[$scale, $scale, $scale]} position={[0.2, -1, 0.75]} envMapIntensity={11}>
		<T.SphereGeometry />
		
		<T.MeshNormalMaterial opacity={0.5} />
	</T.Mesh>

	<T.Mesh castShadow position={[0.2, 1, 0.5]} envMapIntensity={2}>
		<Breathe scale={0.4} />
		<T.MeshPhongMaterial shininess={1} />
	</T.Mesh>
</T.Group>

<!-- ringed planet -->
<T.Group receiveShadow scale={1.2} rotation={[0, 0, planetRotation]} position={[-24, 5, -20]}>
	<T.Mesh envMapIntensity={11}>
		<T.SphereGeometry />
		<T.MeshStandardMaterial color="#FFDAB9" />
	</T.Mesh>

	<T.Mesh rotation={[Math.PI / 4, 0, 0]} envMapIntensity={11}>
		<T.TorusGeometry args={[1.4, 0.3, 3, 50]} />
		<T.MeshNormalMaterial />
	</T.Mesh>
</T.Group>

<!-- rocket -->
<T.Mesh castShadow rotation={[galaxyRotationZ / 2, particleRotationX / 3, galaxyRotationZ / 4]}>
	<Rocket scale={1} rotation={[-Math.PI / 2, 0, -Math.PI / 4]} position={[-10, 1.5, 0.75]} />
</T.Mesh>

<T.Mesh>
	<Float floatingRange={[0, 1]} floatIntensity={2}>
		<Great scale={0.2} rotation={[0, 1.2, 0.2]} position={[-18, 5, -5]} />
	</Float>
</T.Mesh>
