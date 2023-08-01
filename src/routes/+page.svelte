<script>
	let maxIterations = 50;
	let baseIterations = 50;
	let width = 800;
	let height = 600;
	let xmin = -2.25;
	let xmax = 0.75;
	let ymin = -1.5;
	let ymax = 1.5;
	let canvasRef;

	function initIterationArray(xmin, xmax, ymin, ymax) {
		let iterationArray = new Array(width).fill(new Array(height).fill(0));
		for (let x = 0; x < width; x++) {
			let xArray = [];
			for (let y = 0; y < height; y++) {
				let it = pixelIterationCount(xmin, xmax, ymin, ymax, x, y);
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
	function pixelIterationCount(xmin, xmax, ymin, ymax, px, py) {
		let x0 = xmin + (xmax - xmin) * (px / width);
		let y0 = ymin + (ymax - ymin) * (py / height);
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
		generateMandelbrotImage(xmin, xmax, ymin, ymax);
	});

	afterUpdate(() => {
		generateMandelbrotImage(xmin, xmax, ymin, ymax);
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
	function generateMandelbrotImage(xmin, xmax, ymin, ymax) {
		let iterationArray = initIterationArray(xmin, xmax, ymin, ymax);
		plotMandelToImageDataSimple(width, height, iterationArray);
	}

	function handleMouseClick(event) {
		event.preventDefault();

		// Calculate the mouse position relative to the canvas
		const rect = canvasRef.getBoundingClientRect();
		const mouseX = event.clientX - rect.left;
		const mouseY = event.clientY - rect.top;

		// Calculate the zoom factor based on the click type (left-click or right-click)
		const zoomFactor = event.which === 1 ? 0.6 : 1.4;

		// Calculate the new view center based on the click position
		const centerX = xmin + (xmax - xmin) * (mouseX / width);
		const centerY = ymin + (ymax - ymin) * (mouseY / height);

		// Adjust the view region accordingly
		const newWidth = (xmax - xmin) * zoomFactor;
		const newHeight = (ymax - ymin) * zoomFactor;
		const newXmin = centerX - newWidth / 2;
		const newXmax = centerX + newWidth / 2;
		const newYmin = centerY - newHeight / 2;
		const newYmax = centerY + newHeight / 2;

		// Update the view region
		xmin = newXmin;
		xmax = newXmax;
		ymin = newYmin;
		ymax = newYmax;

		maxIterations = Math.max(baseIterations, Math.round(maxIterations / (1.2 * zoomFactor)));

		// Redraw the Mandelbrot set for the new view region
		generateMandelbrotImage(xmin, xmax, ymin, ymax);
	}
</script>

<main class="flex flex-col gap-5 p-20 justify-center items-center">
	<h1 class="text-5xl">Mandelbrot Set.</h1>
	<canvas
		bind:this={canvasRef}
		width="{width}px"
		height="{height}px"
		class="rounded-xl border border-black"
		on:mousedown={handleMouseClick}
		on:contextmenu={(event) => {
			event.preventDefault();
		}}
	/>

	<div class="flex flex-row gap-10">
		<div class="flex flex-col gap-3">
			<label class="label" for="baseIterations">Minimum Iterations (Be careful):</label>
			<input
				class="input"
				type="number"
				min="10"
				name="baseIterations"
				id="baseIterations"
				bind:value={baseIterations}
			/>
		</div>
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
