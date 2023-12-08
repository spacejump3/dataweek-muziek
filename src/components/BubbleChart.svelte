<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    onMount(async () => {
        // Fetch JSON
        const response = await fetch('../dataset/dataset.json');
        const data = await response.json();

        // Set up the SVG container
        const svg = d3.select('svg').attr('width', 1000).attr('height', 1000);

        // Create a force simulation
        const simulation = d3
            .forceSimulation()
            .force('charge', d3.forceManyBody().strength(10)) // Repulsion force
            .force('center', d3.forceCenter(500, 500)) // Centering force
            .force(
                'collision',
                d3.forceCollide().radius((d) => radiusScale(d.value))
            );

        // Flatten the data into an array of nodes
        const nodes = Object.values(data).reduce((acc, artist) => {
            Object.entries(artist.values).forEach(([key, value]) => {
                acc.push({
                    artist: artist.artist,
                    key,
                    value,
                    image: artist.image,
                    album: artist.album,
                });
            });
            return acc;
        }, []);

        // Create a color scale for artists
        const colorScale = d3.scaleOrdinal(d3.schemeCategory10);
        const radiusScale = d3
            .scaleSqrt()
            .domain([0, d3.max(nodes, (d) => d.value)])
            .range([5, 80]);

        // Create SVG elements for nodes (circles)
        const circles = svg
            .selectAll('.circle')
            .data(nodes)
            .enter()
            .append('g') // Create a group for each circle
            .attr('class', 'circle-group')
            .call(
                d3
                    .drag()
                    .on('start', (event, d) => {
                        if (!event.active)
                            simulation.alphaTarget(0.3).restart();
                        d.fx = d.x;
                        d.fy = d.y;
                    })
                    .on('drag', (event, d) => {
                        d.fx = event.x;
                        d.fy = event.y;
                    })
                    .on('end', (event, d) => {
                        if (!event.active) simulation.alphaTarget(0);
                        d.fx = null;
                        d.fy = null;
                    })
            );

        svg.append('defs')
            .append('filter')
            .attr('id', 'blur-filter')
            .append('feGaussianBlur')
            .attr('in', 'SourceGraphic') // Apply the blur to the entire element
            .attr('stdDeviation', 5); // Adjust the standard deviation for the blur

        circles
            .append('circle')
            .attr('class', 'circle')
            .attr('r', (d) => radiusScale(d.value)) // Adjust the scale for radius
            .attr('fill', (d) => colorScale(d.artist))
            .style('stroke', (d) => colorScale(d.artist))
            .style('stroke-width', '4')
            .style('filter', 'drop-shadow(.1em .1em .2em black)');

        circles
            .append('clipPath') // Create a clipPath for each circle
            .attr('id', (d, i) => `clip-circle-${i}`)
            .append('circle') // Create a circle within each clipPath
            .attr('cx', 0)
            .attr('cy', 0)
            .attr('r', (d) => radiusScale(d.value)); // Use the same radius as the circle

        // Add image to each circle using clip-path
        circles
            .append('image')
            .attr('xlink:href', (d) => d.image)
            .attr('x', (d) => -radiusScale(d.value)) // Adjust the x position based on the radius
            .attr('y', (d) => -radiusScale(d.value)) // Adjust the y position based on the radius
            .attr('width', (d) => radiusScale(d.value) * 2) // Use twice the radius as the width
            .attr('height', (d) => radiusScale(d.value) * 2) // Use twice the radius as the height
            .attr('clip-path', (d, i) => `url(#clip-circle-${i})`)
            .style('box-shadow', '1em 1em 1em pink')
            .on('mouseover', function (event, d) {
                d3.select(this)
                    .transition(20)
                    .ease(d3.easeCircleOut)
                    .style('transform', 'scale(1.5)');

                d3.select(this.parentNode).raise();

                d3.select(this.parentNode)
                    .selectAll('text')
                    .attr('visibility', 'visible');

                d3.select(this.parentNode)
                    .selectAll('rect')
                    .attr('visibility', 'visible');

                d3.select(this.parentNode)
                    .append('circle')
                    .attr('class', 'shadowCircle')
                    .attr('cx', 20)
                    .attr('cy', 20)
                    .attr('r', (d) => radiusScale(d.value * 2))
                    .style('filter', 'url(#blur-filter)')
                    .attr('opacity', 0.4)
                    .lower();
            })
            .on('mouseout', function () {
                d3.select(this).transition(200).style('transform', 'scale(1)');

                d3.select(this.parentNode).lower();

                d3.select(this.parentNode)
                    .selectAll('text')
                    .attr('visibility', 'hidden');

                d3.select(this.parentNode)
                    .selectAll('rect')
                    .attr('visibility', 'hidden');

                d3.select('.shadowCircle').remove();
            });

        // tooltips
        circles
            .append('rect')
            .attr('width', 170) // Adjust the width as needed
            .attr('height', 70) // Adjust the height as needed
            .attr('x', -165) // Adjust the x position based on the width
            .attr('y', 18) // Adjust the y position based on the height
            .attr('rx', 5) // Border radius
            .attr('ry', 5) // Border radius
            .style('fill', 'black') // Background color
            .style('stroke', (d) => colorScale(d.artist))
            .style('stroke-width', 1.5)
            .style('filter', 'drop-shadow(.3em .3em .6em black)')
            .attr('visibility', 'hidden');

        const textLeftAlign = -155;

        circles
            .append('text')
            .text((d) => d.artist) // You can dynamically set the text based on your data
            .style('fill', 'white')
            .attr('dy', 40)
            .attr('dx', textLeftAlign)
            .attr('text-anchor', 'start')
            .attr('alignment-baseline', 'middle')
            .attr('font-size', '.8em')
            .attr('visibility', 'hidden');

        circles
            .append('text')
            .text((d) => `Album: ${d.album}`) // You can dynamically set the text based on your data
            .style('fill', 'white')
            .attr('dy', 55)
            .attr('dx', textLeftAlign)
            .attr('text-anchor', 'start')
            .attr('alignment-baseline', 'middle')
            .attr('font-size', '.8em')
            .attr('visibility', 'hidden');

        circles
            .append('text')
            .text((d) => `${d.value} ${d.key}'s`) // You can dynamically set the text based on your data
            .style('fill', (d) => colorScale(d.artist))
            .attr('dy', 75)
            .attr('dx', textLeftAlign)
            .attr('text-anchor', 'start')
            .attr('alignment-baseline', 'middle')
            .style('font-weight', 900)
            .attr('visibility', 'hidden');

        // Add text on top of each circle
        // svg.selectAll('.text')
        //     .data(nodes)
        //     .enter()
        //     .append('text')
        //     .attr('class', 'text')
        //     .attr('text-anchor', 'middle')
        //     .attr('alignment-baseline', 'middle')
        //     .text((d) => d.key)
        //     .attr('fill', '#fff')
        //     .style('text-shadow', '.1em .1em .1em black');

        // Update positions on each tick
        simulation.nodes(nodes).on('tick', () => {
            circles.attr('transform', (d) => `translate(${d.x},${d.y})`);

            // Update text positions
            svg.selectAll('.text')
                .attr('x', (d) => d.x)
                .attr('y', (d) => d.y);
        });
    });
</script>

<audio id="audio" src="audio/oh.mp3"></audio>

<svg></svg>

<style>
    svg {
        margin: auto;
    }
</style>
