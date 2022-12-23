<script>
    import { onMount } from "svelte";
    import * as THREE from "three";
    import Stats from "three/addons/libs/stats.module.js";
    import { createNoise2D } from "simplex-noise";

    const noise = createNoise2D();

    let canvas;
    let container;

    onMount(() => {
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, 2, 0.1, 1000);

        const renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
        renderer.setPixelRatio(window.devicePixelRatio);

        const stats = new Stats();
        container.appendChild(stats.domElement);

        const geometry = new THREE.BufferGeometry();

		const positions = [];
        const colours = [];
        const sideLength = 100;
	    const spread = 0.5;
        const noiseScale = 50;
        const noiseIntensity = 5;
		for (let x = 0; x < sideLength; x++) {
			for (let y = 0; y < sideLength; y++) {
                positions.push(
                    (x - (sideLength / 2)) * spread,
                    0, // noise(x / noiseScale, y / noiseScale) * noiseIntensity,
                    (y - (sideLength / 2)) * spread
                );
                colours.push(x / sideLength, y / sideLength, 0.5, 1);
            }
		}

		geometry.setAttribute("position", new THREE.Float32BufferAttribute(positions, 3));
		geometry.setAttribute("color", new THREE.Float32BufferAttribute(colours, 4));

		geometry.computeBoundingSphere();

		const material = new THREE.PointsMaterial({
			size: 0.15,
			vertexColors: true,
			transparent: true,
		});

		const points = new THREE.Points(geometry, material);
		scene.add(points);

        camera.position.set(0, 20, -25);
        camera.lookAt(0, 0, 0);

        function resizeCanvas(canvas, renderer, camera) {
            renderer.setSize(canvas.clientWidth, canvas.clientHeight);
            camera.aspect = canvas.clientWidth / canvas.clientHeight;
            camera.updateProjectionMatrix();
        }

	    resizeCanvas(canvas, renderer, camera);
        window.addEventListener("resize", () => resizeCanvas(canvas, renderer, camera));

        function octaves(x, y, scale, time, octaves) {
            let result = 0;
            let max = 0;
            let amp = 1;
            let freq = 1;

            for (let i = 0; i < octaves; i++) {
                result += noise(x / scale * freq + time, y / scale * freq + time) * amp;
                max += amp;
                amp /= 2;
                freq *= 2;
            }

            return result / max;
        }

        function animate() {
            requestAnimationFrame(animate);

            points.rotation.y += 0.0025;

            for (let i = 1; i < positions.length; i+=3) {
                const x = positions[i - 1];
                const y = positions[i];
                const z = positions[i + 1];
                const time = performance.now() / 2500;
                const scale = 50;

                positions[i] = octaves(x, z, scale, time, 8) * noiseIntensity;
            }

            // for (let i = 0; i < colours.length; i++) {
            //     // Only update the y position
            //     if (i % 4 !== 1) continue;

            //     colours[i] = Math.sin(i / 1000 + Date.now() / 100);
            // }

            geometry.setAttribute("position", new THREE.Float32BufferAttribute(positions, 3));
            // geometry.setAttribute("color", new THREE.Float32BufferAttribute(colours, 4));

            stats.update();
            renderer.render(scene, camera);
        }
        animate();
    });
</script>

<div bind:this={container}>
    <canvas bind:this={canvas} />
</div>
<style>
    canvas {
        position: absolute;
        inset: 0;
        width: 100vw;
        height: 100vh;
    }
</style>
