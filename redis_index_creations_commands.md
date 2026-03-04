# index creation 1
```sh
FT.CREATE documents ON HASH PREFIX 1 docs: SCHEMA doc_embedding VECTOR SVS-VAMANA 12 TYPE FLOAT32 DIM 1536 DISTANCE_METRIC COSINE GRAPH_MAX_DEGREE 40 CONSTRUCTION_WINDOW_SIZE 250 COMPRESSION LVQ8
```

# List all indices
```
FT._LIST
```

# View index info
```
FT.INFO documents
```

# Drop and recreate
```
FT.DROPINDEX documents
```
