<script>
    import { onMount } from "svelte";
    import * as THREE from "three";
    import Stats from "three/addons/libs/stats.module.js";
    import { createNoise2D } from "simplex-noise";

    const noise = createNoise2D();

    let canvas;
    let container;

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
        const sideLength = 50;
	    const spread = 0.5;
        const noiseScale = 50;
        const noiseIntensity = 10;
		for (let x = 0; x < sideLength; x++) {
			for (let y = 0; y < sideLength; y++) {
                positions.push(
                    (x - (sideLength / 2)) * spread,
                    0,
                    (y - (sideLength / 2)) * spread
                );
                // colours.push(x / sideLength, y / sideLength, 0.5, 1);
                colours.push(0, 0, 0, 1);
            }
		}

		geometry.setAttribute("position", new THREE.Float32BufferAttribute(positions, 3));
		geometry.setAttribute("color", new THREE.Float32BufferAttribute(colours, 4));

		geometry.computeBoundingSphere();

		const material = new THREE.PointsMaterial({
			size: 0.1,
			vertexColors: true,
			transparent: true,
		});

		const points = new THREE.Points(geometry, material);
		scene.add(points);

        camera.position.set(0, 15, -15);
        camera.lookAt(0, 0, 0);

        function resizeCanvas(canvas, renderer, camera) {
            renderer.setSize(canvas.clientWidth, canvas.clientHeight);
            camera.aspect = canvas.clientWidth / canvas.clientHeight;
            camera.updateProjectionMatrix();
        }

	    resizeCanvas(canvas, renderer, camera);
        window.addEventListener("resize", () => resizeCanvas(canvas, renderer, camera));

        function animate() {
            requestAnimationFrame(animate);

            points.rotation.y += 0.0025;

            const time = performance.now();

            for (let i = 0; i < (sideLength ** 2) * 4; i++) {
                const x = positions[i];
                const y = positions[i + 1];
                const z = positions[i + 2];

                if (i % 3 === 0 && i < (sideLength ** 2) * 3) {
                    positions[i + 1] = octaves(x, z, noiseScale, time / 3000, 8) * noiseIntensity;
                }

                if (i % 4 === 0) {
                    const y = positions[i / 4 * 3 + 1];

                    // Generate an r, g and b value from a single hue value
                    const hue = (y / noiseIntensity) * 0.5 + 0.5;
                    const rgb = new THREE.Color(`hsl(${hue * 360}, 100%, 50%)`);

                    colours[i] = rgb.r;
                    colours[i + 1] = rgb.g;
                    colours[i + 2] = rgb.b;
                    
                }
            }

            geometry.setAttribute("position", new THREE.Float32BufferAttribute(positions, 3));
            geometry.setAttribute("color", new THREE.Float32BufferAttribute(colours, 4));

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
