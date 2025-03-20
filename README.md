# GUTTENTAG



### **Overview**
The `ElevationService` class provides functionalities for:
1. Retrieving the elevation at a specific geographic coordinate (longitude, latitude) from a raster database.
2. Determining whether there is a clear line of sight between two geographic points, considering terrain elevation and Earth's curvature.

### **Components Explained**

#### 1. **Dependencies & Setup**
- **Spring Framework Annotations**:
  - `@Service`: Marks this class as a service component in Spring.
  - `@Autowired`: Injects dependencies (e.g., `JdbcTemplate`).

- **GeometryFactory**:
  - Used to create geographic objects (points, lines) with a specified coordinate system (EPSG: 4326 for WGS 84).

- **JdbcTemplate**:
  - Facilitates interaction with a relational database.

#### 2. **Elevation Retrieval**
The `getElevation` method:
- Queries a database table `elevation_data` to fetch the elevation at a given point.
- SQL highlights:
  - `ST_Value`: Retrieves the elevation value from the raster data at the specified point.
  - `ST_SetSRID(ST_MakePoint(?, ?), 4326)`: Creates a geographic point with the given longitude and latitude in the WGS 84 coordinate system.
  - `ST_Intersects`: Ensures the point intersects with the raster area.
  - `LIMIT 1`: Fetches the first matching raster value.

#### 3. **Line of Sight Calculation**
The `hasLineOfSight` method determines if there is a clear view from one point to another:
1. **Constructing the Line**:
   - Creates two points using `createPoint` (starting and ending locations).
   - Forms a line (`LineString`) connecting these points.

2. **Terrain Elevation Query**:
   - SQL Query:
     - Uses `ST_Densify` to interpolate intermediate points along the line at 100-meter intervals.
     - Converts these points to their respective elevations using raster data.
     - Outputs the elevation (`ST_Z`), longitude, and latitude.

   - The query result is stored in a list of `TerrainPoint` objects (representing intermediate points).

3. **Earth's Curvature**:
   - `calculateEarthCurvature` computes the effect of Earth's curvature on visibility:
     - Formula: \( \text{curvature} = \frac{\text{distance}^2}{2 \times \text{Earth's Radius}} \).

4. **Visibility Check**:
   - Calculates the distance between points using the **Haversine formula** (accounts for the spherical shape of the Earth).
   - For each terrain point:
     - Computes the expected elevation of the line at that point, factoring in Earth's curvature.
     - Checks if the terrain elevation exceeds the line's height. If yes, the terrain blocks the line of sight.

5. **Return Value**:
   - Returns `true` if there's a clear view; otherwise, `false`.

#### 4. **Utility Methods**
- **`createPoint`**:
  - Creates a `Point` object (longitude, latitude) using `GeometryFactory`.

- **`calculateDistance`**:
  - Implements the **Haversine formula** to calculate the great-circle distance between two geographic points.

#### 5. **Helper Class**
`TerrainPoint`:
- Represents a single point along the line with its longitude, latitude, and elevation.

---

### **How it All Comes Together**
1. You can call `getElevation` with a coordinate to retrieve its elevation from the database.
2. You can use `hasLineOfSight` to check if two points are directly visible to each other without terrain obstruction:
   - It considers terrain elevations along the line between the points.
   - It accounts for Earth's curvature for accurate visibility assessment.

