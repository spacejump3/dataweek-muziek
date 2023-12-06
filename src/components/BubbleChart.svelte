<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    onMount(async () => {
        // Fetch JSON
        const response = await fetch('../dataset/dataset.json');
        const data = await response.json();

        // Set up the SVG container
        const svg = d3.select('svg');

        // Create a force simulation
        const simulation = d3
            .forceSimulation()
            .force('charge', d3.forceManyBody().strength(100)) // Repulsion force
            .force('center', d3.forceCenter(500, 300)) // Centering force
            .force(
                'collision',
                d3.forceCollide().radius((d) => radiusScale(d.value))
            ); // Collision force

        // Flatten the data into an array of nodes
        const nodes = Object.values(data).reduce((acc, artist) => {
            Object.entries(artist.values).forEach(([key, value]) => {
                acc.push({
                    artist: artist.artist,
                    key,
                    value,
                    image: artist.image,
                });
            });
            return acc;
        }, []);

        // Create a color scale for artists
        const colorScale = d3.scaleOrdinal(d3.schemeCategory10);
        const radiusScale = d3
            .scaleSqrt()
            .domain([0, d3.max(nodes, (d) => d.value)])
            .range([5, 60]);

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

        circles
            .append('circle')
            .attr('class', 'circle')
            .attr('r', (d) => radiusScale(d.value)) // Adjust the scale for radius
            .attr('fill', (d) => colorScale(d.artist))
            .style('filter', 'drop-shadow(.1em .1em .2em black)');

        const clipPaths = circles
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
            .on('mouseover', function () {
                d3.select(this)
                    .style('stroke', 'black')
                    .style('stroke-width', '5');

                d3.select(this.parentNode)
                    .selectAll('text')
                    .attr('visibility', 'visible');
            })
            .on('mouseout', function () {
                d3.select(this)
                    .style('stroke', 'none')
                    .style('stroke-width', '0');

                d3.select(this.parentNode)
                    .selectAll('text')
                    .attr('visibility', 'hidden');
            });

        circles
            .append('text')
            .text('hello') // You can dynamically set the text based on your data
            .style('fill', 'white')
            .attr('dy', 20)
            .attr('text-anchor', 'middle')
            .attr('alignment-baseline', 'middle')
            .attr('visibility', 'hidden');

        // Add text on top of each circle
        svg.selectAll('.text')
            .data(nodes)
            .enter()
            .append('text')
            .attr('class', 'text')
            .attr('text-anchor', 'middle')
            .attr('alignment-baseline', 'middle')
            .text((d) => `${d.key}: ${d.value}`)
            .attr('fill', '#fff')
            .style('text-shadow', '.1em .1em .1em black');

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

<p>Dit is een bubblechart &#128511</p>

<svg width="1000" height="1000"></svg>
