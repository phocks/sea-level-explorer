<script lang="ts">
  import seaLevels from "../assets/sealevels-daily_data_last.json";
  import contours from "../assets/Merged_0_to_150.geo.json";

  import * as d3GeoProjection from "d3-geo-projection";
  import * as d3Geo from "d3-geo";
  import * as d3Scale from "d3-scale";
  const d3 = {
    ...d3GeoProjection,
    ...d3Geo,
    ...d3Scale,
  };

  /**
   * Function to turn seconds into years.
   * NOTE: Purposely not exactly accounting for leap years etc.
   * @param seconds - the number of seconds to turn into years
   */
  const secondsToYears = (seconds: number): number =>
    seconds / 60 / 60 / 24 / 365;

  const yearsToSeaLevel = new Map(
    seaLevels.map((d: any) => [secondsToYears(d[0] || 0), d[1]]),
  );

  const keys = Array.from(yearsToSeaLevel.keys());
  const values = Array.from(yearsToSeaLevel.values());
  const scale = d3.scaleLinear().domain(keys).range(values);

  const MARGIN_CONSTANT = -0.06;

  const MARGIN = {
    top: MARGIN_CONSTANT,
    right: MARGIN_CONSTANT,
    bottom: MARGIN_CONSTANT,
    left: MARGIN_CONSTANT,
  };

  interface Props {
    yearsAgo?: number;
  }

  let { yearsAgo = 0 }: Props = $props();

  let rootWidth = $state(600);

  function roundToNearest10(num: number): number {
    return Math.round(num / 10) * 10;
  }

  function roundToNearest5(num: number): number {
    return Math.round(num / 5) * 5;
  }

  let aspectRatio = $derived(1.3333333); //rootWidth < 744 ? 1.2 : 1.4);

  let height = $derived(rootWidth / aspectRatio || 1);
  let seaLevel = $derived(scale(-yearsAgo) ?? 0);
  let roundSeaLevel10 = $derived(roundToNearest10(seaLevel));
  let roundSeaLevel5 = $derived(roundToNearest5(seaLevel));
  $inspect("yearsAgo:", yearsAgo, "seaLevel:", roundSeaLevel5);

  let projection = $derived(
    d3.geoPatterson().fitExtent(
      [
        [rootWidth * MARGIN.left, rootWidth * MARGIN.top],
        [
          rootWidth - rootWidth * MARGIN.right,
          height - rootWidth * MARGIN.bottom,
        ],
      ],
      contours,
    ),
  );

  let path = $derived(d3.geoPath().projection(projection));

  const getFilteredGeoJson = (seaLevel: number) => {
    const filteredGeoJson = contours.features.filter(
      (feature: any) => feature.properties.ELEV === seaLevel,
    );
    return {
      type: "FeatureCollection",
      name: "Merged_0_to_150",
      crs: {
        type: "name",
        properties: { name: "urn:ogc:def:crs:OGC:1.3:CRS84" },
      },
      features: filteredGeoJson,
    };
  };
</script>

<div
  bind:clientWidth={rootWidth}
  class="sea-level-map"
  style="--width: {rootWidth}px; --height: {height}px"
>
  <svg id="sea-level-map-svg-element" width={rootWidth} {height}>
    <defs>
      <pattern
        id="texture-pattern"
        patternUnits="userSpaceOnUse"
        width={rootWidth}
        height={rootWidth}
      >
        <!-- <image
            href={texture}
            x="0"
            y="0"
            width={rootWidth}
            height={rootWidth}
          /> -->
      </pattern>
    </defs>
    <path
      d={path(getFilteredGeoJson(roundSeaLevel5))}
      fill="hsl(223, 18%, 46%)"
    />

    <path
      d={path(getFilteredGeoJson(0))}
      fill="none"
      stroke="#C7D8FF"
      stroke-width="1.0"
      opacity="0.5"
    />
  </svg>
</div>

<style lang="scss">
  .sea-level-map {
    background-color: #252c5a;
  }

  svg {
    display: block;
  }
</style>
