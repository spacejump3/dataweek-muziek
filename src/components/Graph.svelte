<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    onMount(async () => {
        const svg = d3.select('svg');

        const data = await d3.csv('../dataset/dataset.csv', d3.autoType);

        const artists = data.map((d) => d.artist);
        const years = Array.from(new Set(data.map((d) => d.year))); // Get unique years from the dataset

        console.log(data)

        const colorScale = d3
            .scaleLinear()
            .domain([0, d3.max(data, (d) => d.value)])
            .range(['yellow', 'purple']); // Adjust the colors based on your preference

        const cellSize = 50;

        const heatmap = svg
            .selectAll('rect')
            .data(data)
            .enter()
            .append('rect')
            .attr('width', cellSize)
            .attr('height', cellSize)
            .attr('x', (d) => years.indexOf(d.year) * cellSize)
            .attr('y', (d) => artists.indexOf(d.artist) * cellSize)
            .attr('fill', (d) => colorScale(d.value))
            .on('mouseover', function (event, d) {
                handleMouseOver.call(this, d);
            })
            .on('mouseout', handleMouseOut);

        function handleMouseOver(d) {
            // Add a text element for the tooltip
            svg.append('text')
                .attr('class', 'tooltip')
                .text(`${d.artist} - ${d.value}`)
                .attr('x', (years.indexOf(d.year) + 0.5) * cellSize) // Adjust the x-coordinate based on your preference
                .attr('y', (artists.indexOf(d.artist) + 1.3) * cellSize) // Adjust the y-coordinate based on your preference
                .attr('font-size', '.8em') // Adjust the font size based on your preference
                .attr('text-anchor', 'middle') // Center the text
                .attr('alignment-baseline', 'middle') // Center the text
                .attr('fill', 'black') // Adjust the text color based on your preference
                .style(
                    'text-shadow',
                    '.1em .1em .1em white, -.1em -.1em .1em white, 0 -.1em .1em white, -.1em 0 .1em white'
                );

            d3.select(this).attr('stroke', 'black').attr('stroke-width', 3);
        }

        function handleMouseOut() {
            // Remove the tooltip text element on mouseout
            svg.select('.tooltip').remove();

            d3.select(this).attr('stroke', 'none');
        }
    });
</script>

<svg width="1000" height="1000"></svg>
