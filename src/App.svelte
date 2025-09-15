<script lang="ts">
  import SeaLevelMap from "./lib/SeaLevelMap.svelte";
  const YEARS_STEP = 250;

  let yearsAgo = $state(65000);

  function exportSVG() {
    const svgElement = document.getElementById("sea-level-map-svg-element");
    const svgData = svgElement?.outerHTML;
    if (!svgData) return;

    const xmlDeclaration =
      '<?xml version="1.0" encoding="UTF-8" standalone="no"?>\n';
    const svgDoctype =
      '<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">\n';
    const fullSVG = xmlDeclaration + svgDoctype + svgData;

    const blob = new Blob([fullSVG], { type: "image/svg+xml" });
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.href = url;
    link.download = `sea-level-map-${yearsAgo}-years-ago.svg`;
    link.click();
  }
</script>

<main>
  <button onclick={() => (yearsAgo -= YEARS_STEP)}>{yearsAgo}</button>
  <button onclick={exportSVG}>Export SVG</button>
  <SeaLevelMap {yearsAgo} />
</main>

<style lang="scss">
  button {
    border: 1px solid black;
    padding: 0.1em 0.3em;
    margin-top: 0.2em;
    margin-bottom: 0.2em;
  }
</style>
