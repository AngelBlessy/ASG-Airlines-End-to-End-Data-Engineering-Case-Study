# Diagram sources

The three diagrams in `docs/diagrams/` are generated from these Mermaid files
(`.mmd`) with the shared `config.json` theme.

## Regenerate

Requires Node.js.

```
npm install -g @mermaid-js/mermaid-cli

mmdc -i architecture.mmd -o ../architecture.png -c config.json -b white -s 3
mmdc -i data_flow.mmd    -o ../data_flow.png    -c config.json -b white -s 3
mmdc -i data_model.mmd   -o ../data_model.png   -c config.json -b white -s 3
```

`-s 3` renders at 3x for a crisp image in the Word document. After regenerating,
re-run the document build so the new images are embedded.
