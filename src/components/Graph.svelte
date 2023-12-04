<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    onMount(async () => {
        const svg = d3.select('svg');
        const data = ['1', '2', '3', '4', '5', '6'];

        svg.selectAll('rect').data(data).join('rect')
            .attr('width', 100)
            .attr('height', 100)
            .attr('x', (d, i) => i * 160);
    });
</script>

<p>Hello lmao</p>
<p>brooooooooooooo</p>

<svg width='1000'></svg>
