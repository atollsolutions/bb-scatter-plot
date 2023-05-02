<script>
	import { onMount } from 'svelte';
	import { scaleLinear } from 'd3-scale';
	import chroma from 'chroma-js';

	export let points;
	export let points2;
	export let xaxis;
	export let yaxis;
	export let timeField;
	export let xaxis2;
	export let yaxis2;
	export let timeField2;
	export let xscale;
	export let yscale;
	export let ar;
	export let title;

	let svg;
	let width = 500;
	let height = 500;

	let maxX = Number.NEGATIVE_INFINITY;
	let maxY = Number.NEGATIVE_INFINITY;
	let minX = 0;
	let minY = 0;
	let tMax = !!points && !!points.rows && timeField ? new Date(points.rows[0][timeField]) : undefined;
	let tMin = !!points && !!points.rows && timeField ? new Date(points.rows[0][timeField]) : undefined;

	let t2Max = !!points2 && !!points2.rows && timeField2 ? new Date(points2.rows[0][timeField2]) : undefined;
	let t2Min =  !!points2 && !!points2.rows && timeField2 ? new Date(points2.rows[0][timeField2]) : undefined;
	let xRange = [];
	let yRange = [];

	if (!!points && !!points.rows) {
		for (let item of points.rows) {
			if (!!item[xaxis] && !!item[yaxis] && item[yaxis] != "" && item[xaxis] != "") {
				maxX = parseFloat(item[xaxis]) > maxX ? parseFloat(item[xaxis]) : maxX;
				maxY = parseFloat(item[yaxis]) > maxY ? parseFloat(item[yaxis]) : maxY;
				minX = parseFloat(item[xaxis]) < minX ? parseFloat(item[xaxis]) : minX;
				minY = parseFloat(item[yaxis]) < minY ? parseFloat(item[yaxis]) : minY;

				if (!!timeField && !!item[timeField] && tMax && tMin) {
					let time = new Date(item[timeField]);
					tMax = time > tMax ? time : tMax;
					tMin = time < tMin ? time : tMin;
				}
			}
		}
	}

	if (!!points2 && !!points2.rows) {
		for (let item of points2.rows) {
			if (!!item[xaxis2] && !!item[yaxis2] && item[yaxis2] != "" && item[xaxis2] != "") {
				maxX = parseFloat(item[xaxis2]) > maxX ? parseFloat(item[xaxis2]) : maxX;
				maxY = parseFloat(item[yaxis2]) > maxY ? parseFloat(item[yaxis2]) : maxY;
				minX = parseFloat(item[xaxis2]) < minX ? parseFloat(item[xaxis2]) : minX;
				minY = parseFloat(item[yaxis2]) < minY ? parseFloat(item[yaxis2]) : minY;

				if (!!timeField2 && !!item[timeField2] && t2Max && t2Min) {
				let time = new Date(item[timeField2]);
				t2Max = time > t2Max ? time : t2Max;
				t2Min = time < t2Min ? time : t2Min;
				}
			}
		}
	}

	minX = Math.floor(parseInt(minX)/xscale) * xscale - xscale;;
	minY = Math.floor(parseInt(minY)/yscale) * yscale - yscale;
	maxX = Math.floor(parseInt(maxX)/xscale) * xscale + xscale;
	maxY = Math.floor(parseInt(maxY)/yscale) * yscale + yscale;

	for (let j = minX; j <= maxX; j=j+ xscale) {
		xRange.push(j);
	}

	for (let j = minY; j <= maxY; j=j+ yscale) {
		yRange.push(j);
	}

	let timeDiff = Math.abs(tMax - tMin);

	let pointColor = (time) => {
		if (!time) {
			return "#ccc"
		}
		let f = chroma.scale(['#b0f2bc', '#4cc8a3', '#257d98']);
		let t = new Date(time);
		let fraction =  Math.abs(t - tMin)/timeDiff
		return f(fraction).toString() 
	}

	let pointColor2 = (time) => {
		if (!time) {
			return "tomato"
		}

		let f = chroma.scale(['#f3cbd3', '#ca699d', '#6c2167']);
		let t = new Date(time);
		let fraction =  Math.abs(t - tMin)/timeDiff
		return f(fraction).toString() 
	}

	let padding = { top: 20, right: 40, bottom: 40, left: 40 };

	
	let xLen = parseInt((width - padding.right)/(xRange.length-1));
	let yLen = parseInt((height - padding.bottom)/(yRange.length-1));
   	let minRange = Math.min(xLen, yLen);

	$: xScale = scaleLinear()
		.domain([minX, maxX])
		.range([padding.left, !ar ? minRange*(xRange.length-1) :  width - padding.right]);

	$: yScale = scaleLinear()
		.domain([minY, maxY])
		.range([!ar ? minRange*(yRange.length-1) : height - padding.bottom, padding.top]);

	$: xTicks = 
		xRange;

	$: yTicks = 
		yRange;

	onMount(resize);

	function resize() {
		({ width, height } = svg.getBoundingClientRect());
	}

</script>

<svelte:window on:resize='{resize}'/>

<h3 class='title' style="width: { !ar ? minRange*(xRange.length-1) :  width}" >{title}</h3>

<div class="timeLegends" style="width: { !ar ? minRange*(xRange.length-1) :  width}">
	{#if !!points && !!points.rows && timeField && tMax && tMin}
		<div class="grad"><div style="color: #257d98;">{0}s</div><div style="color: #b0f2bc;">{parseFloat((tMax.getTime()-tMin.getTime())/1000).toFixed(2)}s</div></div>
	{/if}

	{#if !!points2 && !!points2.rows && timeField2 && t2Max && t2Min}

		<div class="grad2"><div style="color: #6c2167;">{0}s</div><div style="color: #f3cbd3;">{parseFloat((t2Max.getTime()-t2Min.getTime())/1000).toFixed(2)}s</div></div>
	{/if}
</div>


<svg bind:this={svg} style="width: {!ar ? minRange*(xRange.length-1) + 32:  width }; height: {!ar ? minRange*(yRange.length-1)  + 32 : height + 16};">

	<!-- y axis -->
	<g class='axis y-axis'>
		{#if !!points && !!points.rows && !!xaxis && !!yaxis}
			{#each yTicks as tick}
				<g class='tick tick-{tick}' transform='translate(0, {yScale(tick)})'>
					<line x1='{padding.left}' x2='{xScale(maxX)}'/>
					<text x='{padding.left - 8}' y='+4'>{parseFloat(tick).toFixed(0)}</text>
				</g>
			{/each}
		{/if}
	</g>

	<!-- x axis -->
	<g class='axis x-axis'>
		{#if !!points && !!points.rows && !!xaxis && !!yaxis}
			{#each xTicks as tick}
				<g class='tick' transform='translate({xScale(tick)},0)'>
					<line y1='{yScale(minY)}' y2='{yScale(maxY)}'/>
						<text y='{!ar ? minRange*(yRange.length -1) + 16 : height - padding.bottom + 16}'>{parseFloat(tick).toFixed(0)}</text>
				</g>
			{/each}
		{/if}
	</g>

	<!-- data -->
	{#each points.rows as point}
	{#if !!xaxis && !!yaxis && !!point[yaxis] && !!point[xaxis]}
		<circle cx='{xScale(parseFloat(point[xaxis]))}' cy='{yScale(parseFloat(point[yaxis]))}' r='5' fill={pointColor(point[timeField])}/>
	{/if}
	{/each}

		
	{#each points2.rows as point2}
		{#if !!xaxis2 && !!yaxis2 && !!point2[yaxis2] && !!point2[xaxis2]}
			<circle cx='{xScale(parseFloat(point2[xaxis2]))}' cy='{yScale(parseFloat(point2[yaxis2]))}' r='5' fill={pointColor2( point2[timeField2])}/>
		{/if}
	{/each}


	
</svg>


<style>
	.timeLegends {
		display: flex;
		justify-content: center;
	}
	.title {
    	text-align: center;
  	}
	.grad {
		background: rgb(2,0,36);
		background: linear-gradient(90deg, #b0f2bc 0%, #4cc8a3 50%, #257d98 100%);
		height: 19px; 
		width:40%;
		display: flex;
		justify-content: space-between;
		margin: 10px;
	}
	.grad2 {
		background: rgb(2,0,36);
		background: linear-gradient(90deg, #f3cbd3 0%, #ca699d 50%, #6c2167 100%);
		height: 19px; 
		width:40%;
		display: flex;
		justify-content: space-between;
		margin: 10px;
	}

	circle {
		stroke: rgba(0,0,0,0.5);
	}

	.tick line {
		stroke: #ddd;
		stroke-dasharray: 2;
	}

	text {
		font-size: 12px;
		fill: #999;
	}

	.x-axis text {
		text-anchor: middle;
	}

	.y-axis text {
		text-anchor: end;
	}
	
</style>