<script>
    let maxIterations = 50;
    let width = 650;
    let height = 500;
    let canvasRef;

    function initIterationArray(){
        let iterationArray = new Array(width).fill(new Array(height).fill(0));
        for(let x = 0; x < width; x++){
            let xArray = [];
            for(let y = 0; y < height; y++){
                let it = pixelIterationCount(x,y);
                xArray.push(it);
            }
            iterationArray[x]=(xArray);
        }
        return iterationArray;
    }

    function numIterationsPerPixel(iterationCounts){
        let numIterationsPerPixel = new Array(maxIterations+1).fill(0);
        for(let x = 0; x < width; x++){
            for(let y = 0; y < height; y++){
                let i = iterationCounts[x][y];
                numIterationsPerPixel[i]++;
            }
        }

        return numIterationsPerPixel;
    }

    function totalValue(numIterations){
        let total = 0;
        for(let i = 0; i <= maxIterations; i++){
            total+=numIterations[i];
        } 
        return total;
    }

    function hue(iterationCounts, numIterations, total){
        let hue = Array(width).fill(new Array(height).fill(0));
        for(let x = 0; x < width; x++){
            for(let y = 0; y < height; y++){
                let it = iterationCounts[x][y];
                for(let i = 0; i <= it; i++){
                    hue[x][y]+=(numIterations[i] / total);
                }
            }
        }
        return hue;
    }

    function pixelIterationCount(px,py){
        let x0 = 2.47*(px/width)-2.00; 
        let y0 = 2.24*(py/height)-1.12;
        let x = 0, y= 0;
        let iteration = 0;
        let x2=0, y2=0, w = 0;
        while((x2+y2)<=4 && (iteration<maxIterations)){
            x=x2-y2+x0;
            y=w-x2-y2+y0;
            x2=x*x;
            y2=y*y;
            w=(x+y)*(x+y);
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
    onMount(()=>{
        canvasCtx = canvasRef.getContext('2d');
        generateMandelbrotImage();
    });

    afterUpdate(()=>{
        generateMandelbrotImage();
    })

    function plotMandelToImageData(width, height, hue){
        if(canvasCtx != undefined){
            var imagedata = canvasCtx.createImageData(width, height);
            let highestHue = 0;
            for(let x1 = 0; x1 < width;x1++){
                for(let y1 = 0; y1<height;y1++){
                    let hueCur = hue[x1][y1];
                    if(hueCur>highestHue){
                        highestHue=hueCur;
                    }
                }
            }
            for(let x = 0; x < width; x++){
                for(let y = 0; y < height; y++){
                    let [redIndex, greenIndex, blueIndex, alphaIndex] = getColorIndicesForCoord(x,y,width);
                    let color = getColorFromPalette((hue[x][y]/highestHue)*255);
                    imagedata.data[redIndex]=color[0];
                    imagedata.data[greenIndex]=color[1];
                    imagedata.data[blueIndex]=color[2];
                    imagedata.data[alphaIndex]=color[3];
                    }
                }
                canvasCtx.putImageData(imagedata,0,0);
            }
        }

    function plotMandelToImageDataSimple(width,height, itArray){
        if(canvasCtx != undefined){
            var imagedata = canvasCtx.createImageData(width, height);
            let highestIterationNumber = 0;
            for(let x = 0; x<width; x++){
                for(let y = 0; y<height;y++){
                    if(itArray[x][y]>highestIterationNumber){
                        highestIterationNumber=itArray[x][y];
                    }
                }
            }
            for(let x = 0; x<width; x++){
                for(let y = 0; y<height;y++){
                    let [redIndex, greenIndex, blueIndex, alphaIndex] = getColorIndicesForCoord(x,y,width);
                    let color = getColorFromPalette((itArray[x][y]/highestIterationNumber)*255);
                    imagedata.data[redIndex]=color[0];
                    imagedata.data[greenIndex]=color[1];
                    imagedata.data[blueIndex]=color[2];
                    imagedata.data[alphaIndex]=color[3];
                }   
            }
            canvasCtx.putImageData(imagedata,0,0);
        }
    }
    function getColorFromPalette(i){
        return [Math.max(i,0),Math.max(i,0),Math.max(i,0),255];
    }
    function generateMandelbrotImage(){
        let iterationArray = initIterationArray();
        /*let numIterationsArray = numIterationsPerPixel(iterationArray);
        let total = totalValue(numIterationsArray);
        let hueArray = hue(iterationArray,numIterationsArray,total); 
        plotMandelToImageData(width,height,hueArray);*/
        plotMandelToImageDataSimple(width,height,iterationArray);
    }
    
</script>

<main class="flex flex-col gap-5 p-20 justify-center items-center">
    <h1 class="text-5xl">Mandelbrot Set.</h1>
    <canvas bind:this={canvasRef} width="{width}px" height="{height}px" class="rounded-xl border border-black"></canvas>

    <div class="flex flex-row gap-10">
        <div class="flex flex-col gap-3">
            <label class="label" for="maxIterations">Max Iterations:</label>
            <input class="input" type="number" min="10" name="maxIterations" id="maxIterations" bind:value={maxIterations}>
        </div>
        <div class="flex flex-col gap-3">
            <label class="label" for="width">Width:</label>
            <input class="input" type="number" min="10" max="1000"name="width" id="width" bind:value={width}>
        </div>
        <div class="flex flex-col gap-3">
            <label class="label" for="height">Height:</label>
            <input class="input" type="number" min="10" max="1000" name="height" id="height" bind:value={height}>
        </div>
    </div>
        <div>
            <h3><b>Implementations to work in:</b></h3>
            <ul>
                <li>Zoom in and zoom out</li>
                <li>Move around on screen</li>
                <li>Histogram colouring instead of iteration colouring. </li>
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
<button
	class="btn absolute top-[5%] right-[5%]"
	data-toggle-theme="dark,light"
	data-act-class="ACTIVECLASS">Dark</button
>