# Geospatial Indexing: QuadTree vs GeoHash

## Overview
Both QuadTree and GeoHash are spatial indexing techniques for efficiently storing and querying latitude/longitude data.

## QuadTree

### Concept
- **Tree-based structure** that recursively divides 2D space into 4 quadrants
- Each node represents a geographic region
- Leaf nodes contain actual data points

### How it works
```
World Map
├── NW Quadrant
│   ├── NW-NW
│   ├── NW-NE  
│   ├── NW-SW
│   └── NW-SE
├── NE Quadrant
├── SW Quadrant
└── SE Quadrant
```

### Advantages
- **Dynamic partitioning** - adapts to data density
- **Efficient range queries** - O(log n) for point queries
- **Good for sparse data** - doesn't create empty cells

### Disadvantages
- **Complex implementation** - tree structure maintenance
- **Memory overhead** - pointer storage for tree nodes
- **Not easily distributable** across multiple servers

### Use Cases
- Real-time location tracking
- Game engines (collision detection)
- Image processing

## GeoHash

### Concept
- **String-based encoding** that converts lat/lng into a single string
- Uses **Z-order curve** (Morton encoding) to map 2D space to 1D
- Longer strings = higher precision

### How it works
```
Latitude: 37.7749 → Binary: 10110...
Longitude: -122.4194 → Binary: 01001...
Interleave bits → GeoHash: "9q8yy"
```

### Precision Levels
| Length | Lat Error | Lng Error | Area Size |
|--------|-----------|-----------|-----------|
| 1      | ±23°      | ±23°      | 5000km    |
| 3      | ±2.8°     | ±5.6°     | 156km     |
| 5      | ±0.17°    | ±0.35°    | 4.9km     |
| 7      | ±0.011°   | ±0.022°   | 153m      |
| 9      | ±0.0007°  | ±0.0014°  | 4.8m      |

### Advantages
- **Simple implementation** - just string operations
- **Database friendly** - works with standard B-tree indexes
- **Easy distribution** - can shard by prefix
- **Proximity property** - similar strings = nearby locations

### Disadvantages
- **Fixed grid** - doesn't adapt to data density
- **Edge effects** - nearby points may have different prefixes
- **Less precise** than QuadTree for some queries

### Use Cases
- Database indexing (Redis, MongoDB)
- Distributed systems (Cassandra, DynamoDB)
- Location-based services (Uber, Foursquare)

## Comparison

| Feature | QuadTree | GeoHash |
|---------|----------|---------|
| **Structure** | Tree | String |
| **Precision** | Variable | Fixed levels |
| **Implementation** | Complex | Simple |
| **Database Support** | Custom | Native B-tree |
| **Distribution** | Difficult | Easy |
| **Memory Usage** | Higher | Lower |
| **Query Performance** | O(log n) | O(log n) |

## Implementation Examples

### GeoHash in Redis
```bash
# Add locations
GEOADD locations 13.361389 38.115556 "Palermo"
GEOADD locations 15.087269 37.502669 "Catania"

# Find nearby locations
GEORADIUS locations 15 37 100 km
```

### GeoHash in PostgreSQL
```sql
-- Using PostGIS extension
CREATE INDEX idx_location_geohash ON places USING btree(ST_GeoHash(geom, 7));

-- Query by geohash prefix
SELECT * FROM places WHERE ST_GeoHash(geom, 7) LIKE '9q8yy%';
```

### QuadTree in Application Code
```python
class QuadTree:
    def __init__(self, boundary, capacity=4):
        self.boundary = boundary  # Rectangle
        self.capacity = capacity
        self.points = []
        self.divided = False
        
    def insert(self, point):
        if not self.boundary.contains(point):
            return False
            
        if len(self.points) < self.capacity:
            self.points.append(point)
            return True
            
        if not self.divided:
            self.subdivide()
            
        return (self.nw.insert(point) or self.ne.insert(point) or 
                self.sw.insert(point) or self.se.insert(point))
```

## When to Use Which?

### Choose QuadTree when:
- Data density varies significantly
- Need adaptive partitioning
- Building custom spatial engine
- Memory is not a constraint

### Choose GeoHash when:
- Using existing databases
- Need simple implementation
- Building distributed systems
- Want standard indexing support

## Real-World Applications

### GeoHash Users
- **Uber**: Driver-rider matching
- **MongoDB**: Geospatial queries
- **Redis**: Location caching
- **DynamoDB**: Spatial partitioning

### QuadTree Users
- **Game engines**: Collision detection
- **GIS systems**: Spatial analysis
- **Image processing**: Region queries
- **Scientific computing**: Particle simulations