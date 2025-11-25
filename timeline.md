---
layout: page
title: Timeline
subtitle: Documentation is never innocent; it is always an instrument of power.
---

> This is a timeline for me, and it's maintained as a YAML file [here](https://github.com/YardRat0117/timeline)

<div id="timeline" style="height: 90vh; border: 1px solid lightgray;"></div>

<!-- js-yaml 库 -->
<script src="https://cdn.jsdelivr.net/npm/js-yaml@4.1.0/dist/js-yaml.min.js"></script>
<!-- vis-js Standalone build -->
<script src="https://unpkg.com/vis-timeline@7.7.1/standalone/umd/vis-timeline-graph2d.min.js"></script>

<script>
document.addEventListener("DOMContentLoaded", async () => {
  const SCHEDULE_URL = "https://raw.githubusercontent.com/YardRat0117/timeline/main/schedule.yml";
  const container = document.getElementById("timeline");

  try {
    // 1. 获取 YAML 并解析
    const text = await fetch(SCHEDULE_URL).then(r => r.text());
    const data = jsyaml.load(text);
    console.log("Parsed YAML:", data);

    // 2. 构造 groups（每个 group 保持原顺序，可重复）
    const groups = data.map((g, idx) => ({
      id: `group_${idx}`,
      content: g.group
    }));

    // 3. 构造 items
    let itemId = 1;
    const itemsArray = [];
    data.forEach((group, gIdx) => {
      if (group.steps && Array.isArray(group.steps)) {
        group.steps.forEach(step => {
          const statusColors = {
            done: "#8bc34a",
            active: "#2196f3",
            pending: "#ccc"
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

    // 4. 使用正确的构造函数传入 groups
    new vis.Timeline(container, items, groups, {
      stack: false,
      editable: {
        add: true,
        updateTime: true,
        remove: true
      },
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

