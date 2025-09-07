# Dating App Geospatial Implementation

## Database Schema

### Users Table
```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    age INTEGER,
    gender VARCHAR(10),
    
    -- Location data
    latitude DECIMAL(10, 8),     -- -90 to 90
    longitude DECIMAL(11, 8),    -- -180 to 180
    geohash VARCHAR(12),         -- Precomputed geohash
    location_updated_at TIMESTAMP DEFAULT NOW(),
    
    -- Privacy settings
    max_distance_km INTEGER DEFAULT 50,
    show_distance BOOLEAN DEFAULT true,
    
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for performance
CREATE INDEX idx_users_geohash ON users USING btree(geohash);
CREATE INDEX idx_users_location ON users USING btree(latitude, longitude);
CREATE INDEX idx_users_age_gender ON users (age, gender);
```

### Location History (Optional)
```sql
CREATE TABLE user_locations (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    geohash VARCHAR(12),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_user_locations_geohash ON user_locations (geohash);
```

## GeoHash Implementation

### Precision Selection for Dating Apps
```
Distance Range → GeoHash Length
- 50km radius → 5 characters (4.9km precision)
- 25km radius → 6 characters (1.2km precision)  
- 10km radius → 7 characters (153m precision)
```

### Update User Location
```sql
-- Function to update user location with geohash
CREATE OR REPLACE FUNCTION update_user_location(
    p_user_id BIGINT,
    p_latitude DECIMAL,
    p_longitude DECIMAL
) RETURNS VOID AS $$
BEGIN
    UPDATE users 
    SET 
        latitude = p_latitude,
        longitude = p_longitude,
        geohash = ST_GeoHash(ST_Point(p_longitude, p_latitude), 7),
        location_updated_at = NOW()
    WHERE id = p_user_id;
END;
$$ LANGUAGE plpgsql;
```

## Fast Proximity Queries

### Method 1: GeoHash Prefix Matching
```sql
-- Find users within approximate distance using geohash
WITH target_user AS (
    SELECT latitude, longitude, geohash, max_distance_km
    FROM users WHERE id = $1
),
geohash_candidates AS (
    SELECT u.*, t.latitude as target_lat, t.longitude as target_lng, t.max_distance_km
    FROM users u, target_user t
    WHERE u.geohash LIKE LEFT(t.geohash, 6) || '%'  -- 6-char prefix ≈ 1.2km
    AND u.id != $1
)
SELECT 
    id, username, age, latitude, longitude,
    -- Calculate exact distance
    ST_Distance(
        ST_Point(longitude, latitude)::geography,
        ST_Point(target_lng, target_lat)::geography
    ) / 1000 as distance_km
FROM geohash_candidates
WHERE ST_Distance(
    ST_Point(longitude, latitude)::geography,
    ST_Point(target_lng, target_lat)::geography
) / 1000 <= max_distance_km
ORDER BY distance_km
LIMIT 50;
```

### Method 2: Multi-Level GeoHash Search
```sql
-- Progressive search with multiple geohash precisions
WITH target_user AS (
    SELECT latitude, longitude, geohash, max_distance_km
    FROM users WHERE id = $1
)
SELECT DISTINCT u.id, u.username, u.age, u.latitude, u.longitude,
    ST_Distance(
        ST_Point(u.longitude, u.latitude)::geography,
        ST_Point(t.longitude, t.latitude)::geography
    ) / 1000 as distance_km
FROM users u, target_user t
WHERE u.id != $1
AND (
    u.geohash LIKE LEFT(t.geohash, 5) || '%' OR  -- ~5km radius
    u.geohash LIKE LEFT(t.geohash, 4) || '%' OR  -- ~20km radius  
    u.geohash LIKE LEFT(t.geohash, 3) || '%'     -- ~156km radius
)
AND ST_Distance(
    ST_Point(u.longitude, u.latitude)::geography,
    ST_Point(t.longitude, t.latitude)::geography
) / 1000 <= t.max_distance_km
ORDER BY distance_km
LIMIT 50;
```

## Application Code Examples

### Go Implementation
```go
package main

import (
    "database/sql"
    "fmt"
    "math"
)

type User struct {
    ID        int64   `json:"id"`
    Username  string  `json:"username"`
    Latitude  float64 `json:"latitude"`
    Longitude float64 `json:"longitude"`
    GeoHash   string  `json:"geohash"`
    Distance  float64 `json:"distance_km"`
}

func FindNearbyUsers(db *sql.DB, userID int64, maxDistance int) ([]User, error) {
    query := `
        WITH target_user AS (
            SELECT latitude, longitude, geohash, max_distance_km
            FROM users WHERE id = $1
        )
        SELECT u.id, u.username, u.latitude, u.longitude,
            ST_Distance(
                ST_Point(u.longitude, u.latitude)::geography,
                ST_Point(t.longitude, t.latitude)::geography
            ) / 1000 as distance_km
        FROM users u, target_user t
        WHERE u.geohash LIKE LEFT(t.geohash, 6) || '%'
        AND u.id != $1
        AND ST_Distance(
            ST_Point(u.longitude, u.latitude)::geography,
            ST_Point(t.longitude, t.latitude)::geography
        ) / 1000 <= $2
        ORDER BY distance_km
        LIMIT 50`
    
    rows, err := db.Query(query, userID, maxDistance)
    if err != nil {
        return nil, err
    }
    defer rows.Close()
    
    var users []User
    for rows.Next() {
        var u User
        err := rows.Scan(&u.ID, &u.Username, &u.Latitude, &u.Longitude, &u.Distance)
        if err != nil {
            return nil, err
        }
        users = append(users, u)
    }
    
    return users, nil
}

func UpdateUserLocation(db *sql.DB, userID int64, lat, lng float64) error {
    query := `
        UPDATE users 
        SET latitude = $2, longitude = $3, 
            geohash = ST_GeoHash(ST_Point($3, $2), 7),
            location_updated_at = NOW()
        WHERE id = $1`
    
    _, err := db.Exec(query, userID, lat, lng)
    return err
}
```

### Redis Caching Layer
```go
// Cache active users in Redis for faster queries
func CacheActiveUsers(redis *redis.Client, users []User) error {
    pipe := redis.Pipeline()
    
    for _, user := range users {
        // Add to geospatial index
        pipe.GeoAdd("active_users", &redis.GeoLocation{
            Name:      fmt.Sprintf("user:%d", user.ID),
            Latitude:  user.Latitude,
            Longitude: user.Longitude,
        })
        
        // Set expiration for inactive users
        pipe.Expire("active_users", 30*time.Minute)
    }
    
    _, err := pipe.Exec()
    return err
}

func FindNearbyUsersRedis(redis *redis.Client, lat, lng float64, radiusKm int) ([]string, error) {
    return redis.GeoRadius("active_users", lng, lat, &redis.GeoRadiusQuery{
        Radius:    float64(radiusKm),
        Unit:      "km",
        WithCoord: true,
        WithDist:  true,
        Count:     50,
        Sort:      "ASC",
    }).Result()
}
```

## Performance Optimizations

### 1. Hybrid Approach
```
1. Quick filter with GeoHash prefix (index scan)
2. Exact distance calculation (smaller dataset)
3. Cache results in Redis for active users
```

### 2. Background Location Updates
```go
// Update location asynchronously
func UpdateLocationAsync(userID int64, lat, lng float64) {
    go func() {
        // Update database
        UpdateUserLocation(db, userID, lat, lng)
        
        // Update Redis cache
        redis.GeoAdd("active_users", &redis.GeoLocation{
            Name: fmt.Sprintf("user:%d", userID),
            Latitude: lat,
            Longitude: lng,
        })
    }()
}
```

### 3. Smart Precision Selection
```go
func getGeoHashPrecision(maxDistance int) int {
    switch {
    case maxDistance <= 5:
        return 7  // 153m precision
    case maxDistance <= 25:
        return 6  // 1.2km precision
    case maxDistance <= 100:
        return 5  // 4.9km precision
    default:
        return 4  // 20km precision
    }
}
```

## Key Benefits

1. **Fast Initial Filter**: GeoHash index scan eliminates 99% of users
2. **Exact Distance**: PostGIS calculates precise distances only for candidates
3. **Scalable**: Works with millions of users
4. **Real-time**: Sub-100ms query times with proper indexing
5. **Flexible**: Easy to adjust search radius and precision

## Hybrid Architecture: PostgreSQL + Elasticsearch

### When to Add Elasticsearch

**PostgreSQL-only (Simple apps):**
- Location-based matching only
- Basic filters (age, gender)
- Under 1M users

**Hybrid with Elasticsearch (Complex apps):**
- Text search + location (bio, interests)
- Multiple filters + faceted search
- Real-time analytics
- Advanced scoring algorithms

### Pattern 1: Search Index + Detail Fetch (Most Common)

```json
// Elasticsearch - Search optimized
{
  "user_id": 12345,
  "age": 28,
  "gender": "female",
  "location": {"lat": 37.7749, "lon": -122.4194},
  "interests": ["hiking", "photography"],
  "last_active": "2025-01-07T10:30:00Z",
  "profile_score": 8.5
}

// PostgreSQL - Complete profile
{
  "id": 12345,
  "username": "sarah_sf",
  "email": "sarah@example.com",
  "bio": "Love hiking and photography...",
  "photos": ["photo1.jpg", "photo2.jpg"],
  "verification_status": "verified",
  "subscription_type": "premium"
}
```

**Implementation:**
```go
// 1. Search in Elasticsearch
func SearchUsers(es *elasticsearch.Client, lat, lng float64, filters SearchFilters) ([]int64, error) {
    query := map[string]interface{}{
        "bool": map[string]interface{}{
            "filter": []map[string]interface{}{
                {"range": map[string]interface{}{"age": map[string]interface{}{"gte": filters.MinAge, "lte": filters.MaxAge}}},
                {"term": map[string]interface{}{"gender": filters.Gender}},
                {"geo_distance": map[string]interface{}{
                    "distance": fmt.Sprintf("%dkm", filters.MaxDistance),
                    "location": map[string]float64{"lat": lat, "lon": lng},
                }},
            },
        },
    }
    
    // Returns only user IDs
    return executeSearch(es, query)
}

// 2. Fetch full profiles from PostgreSQL
func GetUserProfiles(db *sql.DB, userIDs []int64) ([]UserProfile, error) {
    query := `
        SELECT id, username, bio, photos, verification_status
        FROM users 
        WHERE id = ANY($1)`
    
    return executeQuery(db, query, userIDs)
}
```

### Pattern 2: Denormalized ES (High-Performance)

```json
// Elasticsearch - Everything needed for display
{
  "user_id": 12345,
  "username": "sarah_sf",
  "age": 28,
  "gender": "female", 
  "location": {"lat": 37.7749, "lon": -122.4194},
  "bio": "Love hiking and photography...",
  "photos": ["photo1.jpg", "photo2.jpg"],
  "interests": ["hiking", "photography"],
  "last_active": "2025-01-07T10:30:00Z"
}
```

**Implementation:**
```go
// Everything in one ES query
func SearchProfiles(es *elasticsearch.Client, query string, location GeoPoint) ([]Profile, error) {
    esQuery := map[string]interface{}{
        "bool": map[string]interface{}{
            "must": []map[string]interface{}{
                {"multi_match": map[string]interface{}{
                    "query": query,
                    "fields": []string{"bio", "skills", "experience"},
                }},
            },
            "filter": []map[string]interface{}{
                {"geo_distance": map[string]interface{}{
                    "distance": "50km",
                    "location": location,
                }},
            },
        },
    }
    
    // Returns complete profiles
    return executeProfileSearch(es, esQuery)
}
```

### Architecture Comparison

| Feature | PostgreSQL Only | Pattern 1 (Search+Fetch) | Pattern 2 (Denormalized) |
|---------|-----------------|---------------------------|---------------------------|
| **Setup Complexity** | ✅ Simple | ❌ Complex | ❌ Complex |
| **Text Search** | 🐌 Slow | ⚡ Fast | ⚡ Fast |
| **Data Consistency** | ✅ Strong | ✅ Strong | ⚠️ Eventually |
| **Query Performance** | ⚡ Fast (geo only) | ⚡ Fast | ⚡ Fastest |
| **Storage Cost** | ✅ Low | ⚠️ Medium | ❌ High |
| **Operational Overhead** | ✅ Low | ❌ High | ❌ High |

### Recommended Hybrid Setup

```
Elasticsearch: user_id, age, gender, location, interests, last_active
PostgreSQL: full_profile, photos, messages, matches, payments
Redis: active_users_cache, recent_searches
```

**Data Flow:**
1. **Search** → Elasticsearch (fast filtering)
2. **Profile Details** → PostgreSQL (complete data)
3. **Active Users** → Redis (real-time cache)

## Production Considerations

- **Privacy**: Store geohash with reduced precision for privacy
- **Caching**: Use Redis for active users (last 30 minutes)
- **Sharding**: Partition by geohash prefix for horizontal scaling
- **Rate Limiting**: Limit location updates to prevent abuse
- **Data Sync**: Keep ES and PostgreSQL in sync with CDC or event streaming