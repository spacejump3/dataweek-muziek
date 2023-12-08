<script>
    import BubbleChart from '../components/BubbleChart.svelte';
</script>

<body>
    <div class="introText">
        <h1>Hoe vaak weten artiesten niet wat ze moeten zeggen?</h1>
        <p>
            Hoor je ook niet veelste vaak al die <em>oh's</em>, <em>ahh's</em>
            of misschien zelfs <em>grrahs</em> in je favoriete pop nummers? Het
            voelt altijd een beetje als filler tekst. Dit wordt ook wel een
            vocalise genoemd, waarbij er wordt gezongen zonder echte woorden te
            gebruiken. Wij hebben een bubble chart gemaakt van hoe vaak een
            vocalise wordt gebruikt van zes verschillende artiesten. Van elke
            artiest hebben we een album uit 2023 gekozen. Sleep je cursor over
            de cirkels om exact te zien hoe vaak ze een specifiek woord zeggen
            en je zult zien <em>hoe vaak</em> artiesten niet weten wat ze moeten
            zeggen. Geniet van de resultaten!
        </p>
    </div>
    <BubbleChart />
    <h2>Posters</h2>
    <div class="images">
        <img src="../img/doja_cat_graph.png" alt="" />
        <img src="../img/drake_graph.png" alt="" />
        <img src="../img/janelle_graph.png" alt="" />
        <img src="../img/olivia_rodrigo_graph.png" alt="" />
        <img src="../img/post_malone_graph.png" alt="" />
        <img src="../img/troye_sivan_graph.png" alt="" />
    </div>
</body>

<footer>Gemaakt door Stephan en Michelle &copy; 2023</footer>

<style>
    body {
        font-family: Georgia, 'Times New Roman', Times, serif;
        text-align: center;
        color: white;
        background: linear-gradient(
            180deg,
            rgba(28, 27, 31, 1) 0%,
            rgb(0, 0, 0) 100%
        );
        display: flex;
        flex-direction: column;
    }

    img {
        width: 30em;
    }

    .images {
        max-width: 100em;
        margin: auto;
    }

    .introText {
        margin: auto;
        width: 50em;
    }

    h1 {
        font-size: 2.5em;
        width: 20em;
        text-align: start;
    }

    p {
        width: 30em;
        font-size: 1.5em;
        line-height: 1.4em;
        text-align: start;
    }

    footer {
        opacity: 0.5;
    }
</style>
