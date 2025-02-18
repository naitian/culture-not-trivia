<script>
	export let indices;
	export let height = 100;
	import * as d3 from 'd3';
	import { onMount } from 'svelte';
	import { fade, scale } from 'svelte/transition';
	import { cubicOut } from 'svelte/easing';

	// First create node IDs (this won't change)
	$: nodeIds = [...new Set(indices.flatMap((d) => [d.from, d.to]))];

	// Then create the node objects with positions
	$: nodeObjects = nodeIds.map((id, i) => ({
		id,
		isSource: i % 2 == 0,
		x: i % 2 === 0 ? 0 : 1,
		y: Math.floor(i / 2) * 10
	}));
	// Pre-calculate sizes once nodes are created
	$: {
		nodeObjects.forEach((node) => {
			node.height = getNodeHeight(node);
			node.width = getNodeWidth(node);
		});
	}

	// Now create links using the map
	$: links = indices.map((d) => ({
		source: d.from,
		target: d.to
	}));

	let trueNodeObjects = nodeObjects;
	let trueLinks = links;

	let width;
	let text_width = 80;
	let simulation;
	let nodeRadius = 5;
	let nodePadding = 5;

	$: console.log(links);

	$: if (width && height && nodeObjects.length) {
		simulation = d3
			.forceSimulation(nodeObjects)
			.force(
				'link',
				d3
					.forceLink(links)
					.id((d) => d.id)
					.distance((d) => {
						// Adjust link distance based on connected nodes' sizes
						const sourceSize = Math.max(d.source.width, d.source.height) / 2;
						const targetSize = Math.max(d.target.width, d.target.height) / 2;
						return 100 + sourceSize + targetSize;
					})
			)
			.force('charge', d3.forceManyBody().strength(-3))
			.force('center', d3.forceCenter(width / 2, height / 2))
			.force(
				'collision',
				d3.forceCollide().radius((d) => Math.max(d.width, d.height) / 2 + nodePadding)
			)
			.on('tick', () => {
				trueNodeObjects = nodeObjects;
				trueLinks = links;
			})


		trueNodeObjects = nodeObjects;
		trueLinks = links;
		console.log(links[0]);
	}

	// Helper function to calculate adjusted start and end points with padding
	function getLinkPoints(link) {
		const dx = link.target.x - link.source.x;
		const dy = link.target.y - link.source.y;
		const distance = Math.sqrt(dx * dx + dy * dy);

		// Calculate unit vector
		const unitX = dx / distance;
		const unitY = dy / distance;

		// Calculate start and end points with padding
		return {
			x1: link.source.x + (nodeRadius + nodePadding) * unitX,
			y1: link.source.y + (nodeRadius + nodePadding) * unitY,
			x2: link.target.x - (nodeRadius + nodePadding) * unitX,
			y2: link.target.y - (nodeRadius + nodePadding) * unitY
		};
	}

	function getLinkTextProperties(link) {
		const dx = link.target.x - link.source.x;
		const dy = link.target.y - link.source.y;
		const angle = (Math.atan2(dy, dx) * 180) / Math.PI;

		// Calculate midpoint
		const midX = (link.source.x + link.target.x) / 2;
		const midY = (link.source.y + link.target.y) / 2;

		// Offset perpendicular to the line
		const offset = -5;
		const perpX = (-dy / Math.sqrt(dx * dx + dy * dy)) * offset;
		const perpY = (dx / Math.sqrt(dx * dx + dy * dy)) * offset;

		return {
			x: midX + perpX,
			y: midY + perpY,
			angle: angle,
			// Flip text if angle is beyond 90 or -90 degrees
			flip: angle > 90 || angle < -90
		};
	}
	function getNodeHeight(node) {
		const lines = wrapText(node.id, text_width);
		return Math.max(nodeRadius * 2, lines.length * 14);
	}

	// Calculate node width based on text
	function getNodeWidth(node) {
		const lines = wrapText(node.id, text_width);
		const maxLine = lines.reduce((max, line) => Math.max(max, line.length), 0);
		return Math.max(
			nodeRadius * 2,
			maxLine * 6 // Same multiplier as in wrapText
		);
	}

	function wrapText(text, width) {
		const words = text.split(/\s+/);
		const lines = [];
		let currentLine = words[0];

		for (let i = 1; i < words.length; i++) {
			const word = words[i];
			const testLine = currentLine + ' ' + word;
			// Rough estimate of text width (you might want to adjust the multiplier)
			const testWidth = testLine.length * 6;

			if (testWidth < width) {
				currentLine = testLine;
			} else {
				lines.push(currentLine);
				currentLine = word;
			}
		}
		lines.push(currentLine);
		return lines;
	}

	function dragstart(event, d) {
		if (!event.active) simulation.alphaTarget(0.3).restart();
		d.fx = d.x;
		d.fy = d.y;
	}

	function drag(event, d) {
		d.fx = event.x;
		d.fy = event.y;
		trueNodeObjects = nodeObjects;
		trueLinks = links;
	}

	function dragend(event, d) {
		if (!event.active) simulation.alphaTarget(0);
		d.fx = null;
		d.fy = null;
	}

	const dragBehavior = d3.drag().on('start', dragstart).on('drag', drag).on('end', dragend);

	function bindDrag(node) {
		dragBehavior(d3.select(node));
	}
</script>

<div class="container" bind:clientWidth={width}>
	<svg {width} {height}>
		<defs>
			<marker
				id="arrowhead"
				viewBox="-10 -5 10 10"
				refX="0"
				refY="0"
				orient="auto"
				markerWidth="8"
				markerHeight="8"
			>
				<path d="M-10,-5L0,0L-10,5" fill="#999" />
			</marker>
		</defs>

		{#each trueLinks as link}
			{#if link.source.x !== undefined && link.target.x !== undefined}
				{@const textProps = getLinkTextProperties(link)}
				{@const points = getLinkPoints(link)}

				<line
					x1={points.x1}
					y1={points.y1}
					x2={points.x2}
					y2={points.y2}
					stroke="#999"
					stroke-width="1"
					marker-end="url(#arrowhead)"
				/>

				<text
					x={textProps.x}
					y={textProps.y}
					text-anchor="middle"
					dominant-baseline="middle"
					fill="#666"
					transform="rotate({textProps.flip
						? textProps.angle + 180
						: textProps.angle}, {textProps.x}, {textProps.y})"
				>
					indexes
				</text>
			{/if}
		{/each}

		{#each trueNodeObjects as node (node.id)}
			{#if node.x !== undefined}
				<g
					transform="translate({node.x},{node.y})"
					class:source={node.isSource}
					use:bindDrag
					in:scale={{
						duration: 2000,
						easing: cubicOut,
						start: 0
					}}
					out:scale={{
						duration: 2000,
						easing: cubicOut,
						start: 0
					}}
				>
					<circle r={nodeRadius} fill="#69b3a2" />
					<text class="node-label" y="1" font-family="sans-serif">
						{#each wrapText(node.id, text_width) as line, i}
							<tspan
								x={node.isSource ? -10 : 10}
								y={i * 14 - (wrapText(node.id, text_width).length - 1) * 7 + 3}
							>
								{line}
							</tspan>
						{/each}
					</text>
				</g>
			{/if}
		{/each}
	</svg>
</div>

<style>
	.container {
		width: 100%;
		background-color: #f0e9dcbb;
	}

	text {
		font-family: 'Open Sans', sans-serif;
		font-size: 12px;
	}

	text.node-label {
		dominant-baseline: middle;
		font-size: 14px;
	}

	.source text.node-label {
		text-anchor: end;
	}
</style>
