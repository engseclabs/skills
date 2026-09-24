---
name: mermaid-diagram
description: >-
  Rules for small, readable Mermaid diagrams that sit under explanatory prose.
  Use whenever writing or editing a Mermaid diagram, or when asked to "diagram
  this", "draw the architecture", or "sketch the data flow".
---

# Mermaid diagrams

- One diagram answers one question. State it in the sentence before the diagram.
- Use at most 6 boxes and 6 arrows, or 10 boxes if some are nested. If it doesn't fit, split it.
- Every box is a bolded term in the prose above, and every bolded term is a box.
- Never label arrows. Explain each connection in the prose above.
- Arrows point from caller to callee. Use two-headed arrows only for true peer links.
- Each diagram is a zoomed-in piece of the overall map. Draw anything outside the zoom as one collapsed box.
- Always call a thing by the same name, in Sentence case. If it has its own page, use that page's title.
- Use subgraphs only for boundaries: trust, network, account, or deployment.
- Shape shows kind: rectangles for services, cylinders for data stores, stadiums for people. No colors or styling.
- Use readable node IDs (`api`, `db`) and quote every label.
