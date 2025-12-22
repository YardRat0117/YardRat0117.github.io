---
layout: page
title: Timeline
subtitle: Documentation is never innocent; it is always an instrument of power.
---

> This is a timeline for me, and it's maintained as a YAML file [here](https://github.com/YardRat0117/timeline)

<div id="timeline"></div>

<!-- js-yaml -->
<script src="https://cdn.jsdelivr.net/npm/js-yaml@4.1.0/dist/js-yaml.min.js"></script>
<!-- vis-js Standalone build -->
<script src="https://unpkg.com/vis-timeline@7.7.1/standalone/umd/vis-timeline-graph2d.min.js"></script>

<script>
document.addEventListener("DOMContentLoaded", async () => {
  const SCHEDULE_URL = "https://raw.githubusercontent.com/YardRat0117/timeline/main/schedule.yml";
  const container = document.getElementById("timeline");

  try {
    // Load YAML file and parse with js-yaml
    const text = await fetch(SCHEDULE_URL).then(r => r.text());
    const data = jsyaml.load(text);
    console.log("Parsed YAML:", data);

    // 2. Construct groups
    const groups = data.map((g, idx) => ({
      id: `group_${idx}`,
      content: g.group
    }));

    // 3. Construct items
    let itemId = 1;
    const itemsArray = [];
    data.forEach((group, gIdx) => {
      if (group.steps && Array.isArray(group.steps)) {
        group.steps.forEach(step => {
          const statusColors = {
            done: "#8bc34a",
            wip: "#2196f3",
            pending: "#ff9800"
          };
          itemsArray.push({
            id: itemId++,
            start: step.start,
            end: step.end || undefined,
            content: `<b>${step.title}</b>`,
            group: `group_${gIdx}`,
            style: `background-color:${statusColors[step.status]||"#ccc"}; color:white;`
          });
        });
      }
    });

    const items = new vis.DataSet(itemsArray);

    // 4. Build figures
    new vis.Timeline(container, items, groups, {
      stack: true,
      editable: false,
      horizontalScroll: true,
      zoomKey: "ctrlKey",
      maxHeight: "90vh",
      tooltip: { followMouse: true }
    });

  } catch(e) {
    console.error("Failed to load timeline:", e);
    container.innerText = "Failed to load timeline，check YAML or URL";
  }
});
</script>

