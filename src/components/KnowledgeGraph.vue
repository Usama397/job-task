<template>
  <div>
    <h2>Knowledge Graph</h2>
    <svg ref="svgContainer" class="graph-container"></svg>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import * as d3 from "d3";

export default {
  setup() {
    const svgContainer = ref(null);

    let nodes = [
      { id: 0, label: "Foundations", type: "parent" },
      { id: 1, label: "Basic ROS2", type: "parent" },
      { id: 2, label: "ROS Basics", type: "child", image: "course.png" },
      { id: 3, label: "Intermediate ROS2", type: "parent" },
      { id: 4, label: "Robotics Theory", type: "child", image: "course.png" },
      { id: 5, label: "Navigation ROS2", type: "child" },
      { id: 6, label: "RTAB", type: "child" },
      { id: 7, label: "Manipulation", type: "child" },
      { id: 8, label: "Robot Creation", type: "child", image: "course.png" },
      { id: 9, label: "Artificial Intelligence", type: "child" },
      { id: 10, label: "ROS Debugging", type: "child" },
      { id: 11, label: "Course of Product", type: "child", image: "course.png" },
      { id: 12, label: "Web Development for Robots", type: "child" },
      { id: 13, label: "Simulation", type: "child" },
      { id: 14, label: "Enterprise", type: "child" },
    ];

    let links = [
      { source: 0, target: 1 },
      { source: 1, target: 2 },
      { source: 1, target: 3 },
      { source: 2, target: 5 },
      { source: 3, target: 4 },
      { source: 3, target: 9 },
      { source: 5, target: 7 },
      { source: 6, target: 5 },
      { source: 7, target: 8 },
      { source: 8, target: 1 },
      { source: 10, target: 1 },
      { source: 12, target: 10 },
      { source: 11, target: 1 },
      { source: 13, target: 2 },
      { source: 14, target: 1 },
    ];

    onMounted(() => {
      const width = 1000;
      const height = 800;

      const svg = d3.select(svgContainer.value)
        .attr("width", width)
        .attr("height", height)
        .append("g")
        .attr("transform", `translate(${width / 2}, ${height / 2})`);

      const simulation = d3.forceSimulation(nodes)
        .force("link", d3.forceLink(links).id(d => d.id).distance(120))
        .force("charge", d3.forceManyBody().strength(-300))
        .force("center", d3.forceCenter(0, 0));

      const link = svg.append("g")
        .selectAll("line")
        .data(links)
        .enter().append("line")
        .attr("stroke", "#888")
        .attr("stroke-width", 2);

      const nodeGroup = svg.append("g")
        .selectAll("g")
        .data(nodes)
        .enter().append("g");

        nodeGroup.append("circle")
  .attr("r", d => (d.type === "parent" ? 24 : 18)) // Parent nodes are larger
  .attr("fill", d => (d.type === "parent" ? "#007bff" : "#444"))
  .attr("stroke", "#fff")
  .attr("stroke-width", 2)
  .call(d3.drag()
    .on("start", dragStarted)
    .on("drag", dragged)
    .on("end", dragEnded))
  .on("click", toggleHighlight);


      // Add Icons inside nodes (Centered Correctly)
      const icons = nodeGroup.append("text")
        .attr("text-anchor", "middle")
        .attr("alignment-baseline", "central")
        .attr("font-size", d => (d.type === "parent" ? "14px" : "12px"))
        .text(d => (d.type === "parent" ? "🔘" : "💡"));

      // Add labels inside nodeGroup to keep them intact
      nodeGroup.append("text")
        .attr("class", "label")
        .attr("font-size", "12px")
        .attr("text-anchor", "middle")
        .attr("dy", d => (d.type === "parent" ? 35 : 28)) // Keep text below nodes
        .text(d => d.label)
        .attr("fill", "#333");

      // Add images for course nodes, hide broken images
      nodeGroup.append("image")
        .filter(d => d.image)
        .attr("xlink:href", d => d.image)
        .attr("width", 24)
        .attr("height", 24)
        .attr("x", -12)
        .attr("y", -12)
        .on("error", function() {
          d3.select(this).remove(); // Hide broken images
        });

      function toggleHighlight(event, d) {
        const isHighlighted = d3.select(this).classed("highlighted");

        if (isHighlighted) {
          // Restore original state
          d3.select(this).classed("highlighted", false)
            .attr("fill", d.type === "parent" ? "#007bff" : "#444");

          link.attr("stroke", "#888");
          icons.filter(n => n.id === d.id).text("🔘");
        } else {
          // Highlight parent node and edges
          d3.select(this).classed("highlighted", true)
            .attr("fill", "#ffcc00");

          link.attr("stroke", l => (l.source.id === d.id || l.target.id === d.id ? "#ffcc00" : "#888"));
          icons.filter(n => n.id === d.id).text("🟡");
        }
      }

      function dragStarted(event, d) {
        if (!event.active) simulation.alphaTarget(0.3).restart();
        d.fx = d.x;
        d.fy = d.y;
      }

      function dragged(event, d) {
        d.fx = event.x;
        d.fy = event.y;
      }

      function dragEnded(event, d) {
        if (!event.active) simulation.alphaTarget(0);
        d.fx = null;
        d.fy = null;
      }

      simulation.on("tick", () => {
        link.attr("x1", d => d.source.x)
          .attr("y1", d => d.source.y)
          .attr("x2", d => d.target.x)
          .attr("y2", d => d.target.y);

        nodeGroup.attr("transform", d => `translate(${d.x},${d.y})`);
      });
    });

    return { svgContainer };
  },
};
</script>

