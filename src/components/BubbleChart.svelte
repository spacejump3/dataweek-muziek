<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';

    const data = [23, 14, 50, 22, 5];

    const dummyData = [
        {
            joji: {
                artist: 'Joji',
                album: 'albumname1',
                values: {
                    oh: 3,
                    yea: 5,
                    uhh: 2,
                    la: 10,
                },
            },

            oliviaRodrigo: {
                artist: 'Olivia Rodrigo',
                album: 'albumname2',
                values: {
                    oh: 3,
                    yea: 5,
                    uhh: 2,
                    la: 10,
                },
            },

            brunoMars: {
                artist: 'Bruno Mars',
                album: 'albumname3',
                values: {
                    oh: 3,
                    yea: 5,
                    uhh: 2,
                    la: 10,
                },
            },

            cardiB: {
                artist: 'Cardi B',
                album: 'albumname4',
                values: {
                    oh: 3,
                    yea: 5,
                    uhh: 2,
                    la: 10,
                },
            },

            troyeSivan: {
                artist: 'Troye Sivan',
                album: 'albumname5',
                values: {
                    oh: 3,
                    yea: 5,
                    uhh: 2,
                    la: 10,
                },
            },

            hayleyKiyoko: {
                artist: 'Hayley Kiyoko',
                album: 'albumname6',
                values: {
                    oh: 3,
                    yea: 5,
                    uhh: 2,
                    la: 10,
                },
            },
        },
    ];

    console.log(dummyData);

    onMount(async () => {
        // Fetch JSON
        const response = await fetch('../dataset/dataset.json');
        const data = await response.json();

        // Set up the SVG container
        const svg = d3.select('svg');
        const width = 500;
        const height = 500;

        // Create a force simulation
        const simulation = d3
            .forceSimulation()
            .force('charge', d3.forceManyBody().strength(100)) // Repulsion force
            .force('center', d3.forceCenter(width / 2, height / 2)) // Centering force
            .force('collision', d3.forceCollide().radius((d) => radiusScale(d.value))); // Collision force

        // Flatten the data into an array of nodes
        const nodes = Object.values(data).reduce((acc, artist) => {
            Object.entries(artist.values).forEach(([key, value]) => {
                acc.push({ artist: artist.artist, key, value });
            });
            return acc;
        }, []);

        // Create a color scale for artists
        const colorScale = d3.scaleOrdinal(d3.schemeCategory10);
        const radiusScale = d3.scaleSqrt()
            .domain([0, d3.max(nodes, (d) => d.value)])
            .range([5, 60])

        // Create SVG elements for nodes (circles)
        const circles = svg
            .selectAll('.circle')
            .data(nodes)
            .enter()
            .append('circle')
            .attr('class', 'circle')
            .attr('r', (d) => radiusScale(d.value)) // Adjust the scale for radius
            .attr('fill', (d) => colorScale(d.artist))
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

        // Add text on top of each circle
        svg.selectAll('.text')
            .data(nodes)
            .enter()
            .append('text')
            .attr('class', 'text')
            .attr('text-anchor', 'middle')
            .attr('alignment-baseline', 'middle')
            .text((d) => `${d.key}: ${d.value}`)
            .attr('fill', '#fff');

        // Update positions on each tick
        simulation.nodes(nodes).on('tick', () => {
            circles.attr('cx', (d) => d.x).attr('cy', (d) => d.y);

            // Update text positions
            svg.selectAll('.text')
                .attr('x', (d) => d.x)
                .attr('y', (d) => d.y);
        });
    });
</script>

<p>brothermanbill</p>

<svg width="1000" height="1000"></svg>
