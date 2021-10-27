<template>
  <div>
    <div
      style="width: 800px; height: 600px"
      width="800"
      height="600"
      ref="surface"
    ></div>
  </div>
</template>

<script lang="ts">
import * as echarts from "echarts";
import { Component, Vue } from "vue-property-decorator";

@Component
export default class KGML extends Vue {
  private readonly kgmlSource =
    '<?xml version="1.0"?>' +
    '<!DOCTYPE pathway SYSTEM "https://www.kegg.jp/kegg/xml/KGML_v0.7.2_.dtd">' +
    "<!-- Creation date: Dec 9, 2020 10:40:09 +0900 (GMT+9) -->" +
    '<pathway name="path:hsa05200" org="hsa" number="05200" title="Pathways in cancer" image="https://www.kegg.jp/kegg/pathway/hsa/hsa05200.png" link="https://www.kegg.jp/kegg-bin/show_pathway?hsa05200">' +
    '  <entry id="2" name="path:hsa05200" type="map" link="https://www.kegg.jp/dbget-bin/www_bget?hsa05200">' +
    '    <graphics name="TITLE:Pathways in cancer" fgcolor="#000000" bgcolor="#FFFFFF" type="roundrectangle" x="141" y="40" width="180" height="27"/>' +
    "  </entry>" +
    '  <entry id="119" name="hsa:1630" type="gene" link="https://www.kegg.jp/dbget-bin/www_bget?hsa:1630">' +
    '    <graphics name="DCC, CRC18, CRCR1, HGPPS2, IGDCC1, MRMV1, NTN1R1" fgcolor="#000000" bgcolor="#BFFFBF" type="rectangle" x="133" y="221" width="46" height="17"/>' +
    "  </entry>" +
    '  <entry id="120" name="hsa:836" type="gene" link="https://www.kegg.jp/dbget-bin/www_bget?hsa:836">' +
    '    <graphics name="CASP3, CPP32, CPP32B, SCA-1" fgcolor="#000000" bgcolor="#BFFFBF" type="rectangle" x="215" y="189" width="46" height="17"/>' +
    "  </entry>" +
    "</pathway>";

  public mounted(): void {
    // TODO
    const surface = this.$refs["surface"] as HTMLDivElement;
    const xmlParser = new DOMParser();
    const kgmlDoc = xmlParser.parseFromString(this.kgmlSource, "text/xml");
    const data = [];
    // Find the first pathway
    const pathway = kgmlDoc.getElementsByTagName("pathway")[0];
    // Go through each entry node in the pathway
    for (const entry of pathway.getElementsByTagName("entry")) {
      // Find the graphics node of the entry node
      const graphics = entry.getElementsByTagName("graphics")[0];
      const type = graphics.getAttribute("type");
      if (type === "rectangle") {
        // As a test we define the data items as rectangles in the form of [x, y, width, height].
        data.push([
          parseInt(graphics.getAttribute("x") as string),
          parseInt(graphics.getAttribute("y") as string),
          parseInt(graphics.getAttribute("width") as string),
          parseInt(graphics.getAttribute("height") as string),
        ]);
      } else {
        console.log("Unsupported KGML graphics type", type);
      }
    }
    const options = {
      xAxis: {
        min: 0,
        // This must be the same or a factor of the surface container width.
        max: 800,
        // For testing show a tick every 100 "pixels"
        interval: 100,
      },
      yAxis: {
        min: 0,
        // This must be the same or a factor of the surface container height.
        max: 600,
        // We need to inverse the y axis as our coordinates for drawing start in the top left corner.
        inverse: true,
        // For testing show a tick every 100 "pixels"
        interval: 100,
      },
      grid: {
        // As echarts applies padding to the actual coordinate system between the axes (to render the ticks etc.)
        // we need to apply even padding on all sides to keep the aspect ratio.
        top: 30,
        left: 30,
        bottom: 30,
        right: 30,
      },
      series: [
        {
          type: "custom",
          renderItem: this.renderItem,
          data: data,
          color: "red",
        },
      ],
    } as echarts.EChartsOption;
    const chart = echarts.init(surface);
    chart.setOption(options);
  }

  private renderItem(
    params: echarts.CustomSeriesRenderItemParams,
    api: echarts.CustomSeriesRenderItemAPI
  ): echarts.CustomSeriesRenderItemReturn {
    // For each data item, this method is called.
    // api contains the information of the data item and means to use it.
    // api.value is the means to access the data item values.
    // As an example, the data item [10, 20, 30, 40] can be used with api.value(0) to access
    // the value at index 0 which is 10. api.value(1) returns 20 and so on.
    const x = api.value(0) as number;
    const y = api.value(1) as number;
    const width = api.value(2) as number;
    const height = api.value(3) as number;
    console.log("draw rectangle at", x, y, "with size", width, height);
    const start = api.coord([x, y]);
    const end = api.coord([x + width, y + height]);
    const color = api.visual("color");
    return {
      // Can be one of [circle, rect, sector, polygon, polyline, line, arc, bezierCurve, ring, ellipse, compoundPath].
      type: "rect",
      // How the shape is defined depends on the type.
      // clipRectByRect makes sure that rectangles that would reach outside of our drawing area are clipped, so that only
      // the part inside the drawing area is visible.
      shape: echarts.graphic.clipRectByRect(
        {
          x: start[0],
          y: start[1],
          width: end[0] - start[0],
          height: end[1] - start[1],
        },
        {
          x: (params.coordSys as any).x,
          y: (params.coordSys as any).y,
          width: (params.coordSys as any).width,
          height: (params.coordSys as any).height,
        }
      ),
      style: {
        fill: color,
        stroke: echarts.color.lift(color as string, 1),
      },
    };
  }
}
</script>
