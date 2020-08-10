<template>
  <div class="home">
    <svg width="300" height="500" id="demo"></svg>
  </div>
</template>

<script>
import dagreD3 from "dagre-d3";
import * as d3 from "d3";
export default {
  name: "Home",
  data() {
    return {
      dataSet: {
        nodes: [
          { id: 0, label: "流动人员", shape: "rect" },
          { id: 1, label: "安全筛查", shape: "rect" },
          { id: 2, label: "热像仪人体测温筛查", shape: "diamond" },
          { id: 3, label: "人工复测", shape: "diamond" },
          { id: 4, label: "快速通过", shape: "rect" },
          { id: 5, label: "紧急处理", shape: "rect" }
        ],
        edges: [
          { source: 0, target: 1, label: "" },
          { source: 1, target: 2, label: "" },
          { source: 2, target: 4, label: "正常" },
          { source: 2, target: 3, label: "不正常" },
          { source: 3, target: 5, label: "不正常" },
          { source: 3, target: 4, label: "正常" }
        ]
      },
      tooltip: ""
    };
  },
  mounted() {
    this.tooltip = this.createTooltip();
    // 创建graph对象
    let g = new dagreD3.graphlib.Graph();
    // 设置图
    g.setGraph({
      rankdir: "TB", // T:top B:bottom
      marginx: 10,
      marginy: 10
    });
    this.dataSet.nodes.forEach(item => {
      g.setNode(item.id, {
        // 节点标签
        label: item.label,
        // 节点形状
        shape: item.shape,
        // 节点样式
        style: "fill:#fff;stroke:#000"
      });
    });
    this.dataSet.edges.forEach(item => {
      g.setEdge(item.source, item.target, {
        // 边标签
        label: item.label,
        // 边样式
        style: "fill:#fff;stroke:#333;stroke-width:1.5px"
      });
    });
    // 创建渲染器
    let render = new dagreD3.render();
    // 选择svg并添加一个g元素作为绘图容器
    let svgGroup = d3.select("svg").append("g");
    // 建立拖拽缩放
    let svg = d3.select("svg");
    let zoom = d3.zoom().on("zoom", function() {
      svgGroup.attr("transform", d3.event.transform);
    });
    svg.call(zoom);

    // 在绘图容器上运行渲染器生成流程图
    render(svgGroup, g);

    // 鼠标悬停显示隐藏tooptip
    svgGroup
      .selectAll("g.node")
      .on("mouseover", v => {
        this.tipVisible(g.node(v).label);
      })
      .on("mouseout", () => {
        this.tipHidden();
      });
  },
  methods: {
    // 创建提示框
    createTooltip() {
      return d3
        .select("body")
        .append("div")
        .classed("tooltip", true)
        .style("opacity", 0)
        .style("display", "none");
    },
    // tooltip显示
    tipVisible(textContent) {
      this.tooltip
        .transition()
        .duration(400)
        .style("opacity", 0.9)
        .style("display", "block");
      this.tooltip
        .html(textContent)
        .style("left", d3.event.pageX + 15 + "px")
        .style("top", d3.event.pageY + 15 + "px");
    },

    // tooltip隐藏
    tipHidden() {
      this.tooltip
        .transition()
        .duration(400)
        .style("opacity", 0)
        .style("display", "none");
    }
  }
};
</script>

<style lang="less">
#demo {
  border: 1px solid #333333;
}
.tooltip {
  position: absolute;
  font-size: 12px;
  text-align: center;
  background-color: white;
  border-radius: 3px;
  box-shadow: rgb(174, 174, 174) 0px 0px 10px;
  cursor: pointer;
  display: inline-block;
  padding: 10px;
}
.tooltip > div {
  padding: 10px;
}
</style>
