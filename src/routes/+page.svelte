<script>
	let maxIterations = 50;
	let width = 800;
	let height = 600;
	let canvasRef;

	function initIterationArray() {
		let iterationArray = new Array(width).fill(new Array(height).fill(0));
		for (let x = 0; x < width; x++) {
			let xArray = [];
			for (let y = 0; y < height; y++) {
				let it = pixelIterationCount(x, y);
				xArray.push(it);
			}
			iterationArray[x] = xArray;
		}
		return iterationArray;
	}

	/*
        Mandelbrot is similiar to Julia sets, just that Mandelbrot uses the C as a complex number made of x and y positions of the, in this case, pixel
        while Julia Sets just use some predefined static value. 
    */
	function pixelIterationCount(px, py) {
		let x0 = 2.47 * (px / width) - 2.0;
		let y0 = 2.24 * (py / height) - 1.12;
		let x = 0,
			y = 0;
		let iteration = 0;
		let x2 = 0,
			y2 = 0;
		while (x2 + y2 <= 4 && iteration < maxIterations) {
			y = 2 * x * y + y0;
			x = x2 - y2 + x0;
			x2 = x * x;
			y2 = y * y;
			iteration++;
		}
		return iteration;
	}

	const getColorIndicesForCoord = (x, y, width) => {
		const red = y * (width * 4) + x * 4;
		return [red, red + 1, red + 2, red + 3];
	};

	import { afterUpdate, onMount } from "svelte";
	let canvasCtx;
	onMount(() => {
		canvasCtx = canvasRef.getContext("2d");
		generateMandelbrotImage();
	});

	afterUpdate(() => {
		generateMandelbrotImage();
	});

	function plotMandelToImageDataSimple(width, height, itArray) {
		if (canvasCtx != undefined) {
			var imagedata = canvasCtx.createImageData(width, height);
			for (let x = 0; x < width; x++) {
				for (let y = 0; y < height; y++) {
					let [redIndex, greenIndex, blueIndex, alphaIndex] = getColorIndicesForCoord(x, y, width);
					let color = getColorFromPalette(itArray[x][y] / maxIterations);
					imagedata.data[redIndex] = color[0];
					imagedata.data[greenIndex] = color[1];
					imagedata.data[blueIndex] = color[2];
					imagedata.data[alphaIndex] = color[3];
				}
			}
			canvasCtx.putImageData(imagedata, 0, 0);
		}
	}
	function getColorFromPalette(i) {
		return [Math.max(i * 255, 0), Math.max(i * 255, 0), Math.max(i * 255, 0), 255];
	}
	function generateMandelbrotImage() {
		let iterationArray = initIterationArray();
		plotMandelToImageDataSimple(width, height, iterationArray);
	}
</script>

<main class="flex flex-col gap-5 p-20 justify-center items-center">
	<h1 class="text-5xl">Mandelbrot Set.</h1>
	<canvas bind:this={canvasRef} width="{width}px" height="{height}px" class="rounded-xl border border-black" />

	<div class="flex flex-row gap-10">
		<div class="flex flex-col gap-3">
			<label class="label" for="maxIterations">Max Iterations (Be careful):</label>
			<input
				class="input"
				type="number"
				min="10"
				name="maxIterations"
				id="maxIterations"
				bind:value={maxIterations}
			/>
		</div>
		<div class="flex flex-col gap-3">
			<label class="label" for="width">Width:</label>
			<input class="input" type="number" min="10" max="1000" name="width" id="width" bind:value={width} />
		</div>
		<div class="flex flex-col gap-3">
			<label class="label" for="height">Height:</label>
			<input class="input" type="number" min="10" max="1000" name="height" id="height" bind:value={height} />
		</div>
	</div>
	<div>
		<h3><b>Implementations to work in:</b></h3>
		<ul>
			<li>Zoom in and zoom out</li>
			<li>Move around on screen</li>
		</ul>
	</div>
	<a href="https://github.com/AreOlsen" class="flex flex-row gap-5"
		><span>Created By Are Olsen - 01.03.2023 - V.0.1</span><img
			src="github.png"
			style="height:25px"
			alt="Github logo"
		/></a
	>
</main>
<button class="btn absolute top-[5%] right-[5%]" data-toggle-theme="dark,light" data-act-class="ACTIVECLASS"
	>Dark</button
>
