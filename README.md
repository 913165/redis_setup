## redis_setup

## Semantic search examples

Queries:

```
"best laptop for gaming"
"lightweight laptop for travel"
"budget laptop for students"
"professional laptop for video editing"
```
Students will clearly see how vector similarity works.

There can be other datasets like below


```
🏠 Real estate dataset
📚 Book store dataset
🍽 Restaurant dataset
🚗 Cars dataset
🏥 Healthcare dataset
```
#in our example we see laptop model dataset..

sample member of dataset is as below

```json
 {
    "model": "Jigger",
    "brand": "Velorim",
    "price": 270,
    "type": "Kids bikes",
    "specs": {
      "material": "aluminium",
      "weight": "10"
    },
    "description": "Small and powerful, the Jigger is the best ride for the smallest of tikes! This is the tiniest kids\u2019 pedal bike on the market available without a coaster brake, the Jigger is the vehicle of choice for the rare tenacious little rider raring to go. We say rare because this smokin\u2019 little bike is not ideal for a nervous first-time rider, but it\u2019s a true giddy up for a true speedster. The Jigger is a 12 inch lightweight kids bicycle and it will meet your little one\u2019s need for speed. It\u2019s a single speed bike that makes learning to pump pedals simple and intuitive. It even has  a handle in the bottom of the saddle so you can easily help your child during training!  The Jigger is among the most lightweight children\u2019s bikes on the planet. It is designed so that 2-3 year-olds fit comfortably in a molded ride position that allows for efficient riding, balanced handling and agility. The Jigger\u2019s frame design and gears work together so your buddingbiker can stand up out of the seat, stop rapidly, rip over trails and pump tracks. The Jigger\u2019s is amazing on dirt or pavement. Your tike will speed down the bike path in no time. The Jigger will ship with a coaster brake. A freewheel kit is provided at no cost. "
  },
```

schema for above json is as below
```json
schema = (
    # Basic Fields
    TextField("$.model", no_stem=True, as_name="model"),
    TextField("$.brand", no_stem=True, as_name="brand"),
    NumericField("$.price", as_name="price"),
    TagField("$.type", as_name="type"),

    # Nested Specs Fields
    TextField("$.specs.processor", as_name="processor"),
    TagField("$.specs.ram", as_name="ram"),
    NumericField("$.specs.weight", as_name="weight"),

    # Description
    TextField("$.description", as_name="description"),

    # Vector Field
    VectorField(
        "$.description_embeddings",
        "HNSW",   # 🔥 Better than FLAT for production
        {
            "TYPE": "FLOAT32",
            "DIM": VECTOR_DIMENSION,
            "DISTANCE_METRIC": "COSINE",
        },
        as_name="vector",
    ),
)
```
