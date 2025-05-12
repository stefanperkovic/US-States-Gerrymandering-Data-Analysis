## Time Complexity Analysis of Gerrymandering Analysis Script

This document outlines the estimated time complexity of the major computational steps in the generic Python script used for gerrymandering analysis. Time complexity is expressed using Big O notation, which describes the limiting behavior of a function when the argument tends towards a particular value or infinity.

**Variables Used:**

* `B_vf`: Number of rows (blocks) in the voter file.
* `B_cvap`: Number of rows (blocks) in the CVAP file.
* `T`: Number of unique Census Tracts.
* `D`: Number of Congressional Districts.
* `V_tracts`: Total number of vertices in all tract geometries.
* `V_districts`: Total number of vertices in all district geometries.
* `L`: Average length of string identifiers (like GEOIDs), assumed small and constant for many operations.

---

### 1. Data Loading

* **Loading CSV Files (Voter File, CVAP File) with Pandas:**
    * `pd.read_csv()`: Approximately **O(Number of cells)**. If we consider `B_vf` rows and `C_vf` columns for the voter file, it's O(B_vf * C_vf). Assuming `C_vf` is relatively constant, this simplifies to **O(B_vf)** for the voter file and **O(B_cvap)** for the CVAP file.
* **Loading Shapefiles (Tracts, Districts) with GeoPandas:**
    * `gpd.read_file()`: The complexity depends on the file format and the number and complexity of geometries. For shapefiles, it involves reading attribute data and geometric data. It's roughly proportional to the file size, which is influenced by `T` and `V_tracts` for tracts, and `D` and `V_districts` for districts. A loose approximation might be **O(T + V_tracts)** for tracts and **O(D + V_districts)** for districts.

---

### 2. Data Cleaning and Preprocessing (on Block-Level Data)

* **String Operations (e.g., `zfill`, slicing for `census_tract`):**
    * Applied row-wise. For `B_vf` rows, this is **O(B_vf * L)**, which simplifies to **O(B_vf)** if `L` is constant. Similar for `B_cvap`.
* **Arithmetic Operations (e.g., calculating proportions, estimated votes):**
    * Applied row-wise. For `B_vf` rows, this is **O(B_vf)**.
* **Type Conversions (e.g., `.astype(str)`):**
    * Applied row-wise: **O(B_vf)** or **O(B_cvap)**.

---

### 3. Groupby Aggregations (Pandas)

* **Aggregating Voter Data from Blocks to Tracts:**
    * `df_active.groupby('census_tract').agg(...)`: The complexity is typically **O(B_vf)** if using hash-based aggregation, or **O(B_vf log B_vf)** if sorting of group keys is involved. The number of resulting groups is `T`.
* **Aggregating CVAP Data from Blocks to Tracts:**
    * `df_cvap.groupby('census_tract').agg(...)`: Similarly, **O(B_cvap)** or **O(B_cvap log B_cvap)**.
* **Aggregating Tract Data to Districts (after spatial join):**
    * `gdf_tracts_with_district.groupby(DISTRICT_ID_COL).agg(...)`: If there are `T_assigned` tracts successfully assigned to districts, this is **O(T_assigned)** or **O(T_assigned log T_assigned)**. The number of resulting groups is `D`.

---

### 4. Merges (Pandas)

* **Merging `tract_voter_agg` with `tract_cvap` (on `census_tract`):**
    * `pd.merge()`: If both DataFrames have `T` rows (approximately), and `census_tract` is indexed or sorted, it can be close to **O(T)**. If sorting is required, it's **O(T log T)**.
* **Merging aggregated data with `gdf_tract_shapes` (on `census_tract`):**
    * Similar to above, approximately **O(T)** or **O(T log T)**.
* **Merging `district_aggregates` with `gdf_districts` (on `DISTRICT_ID_COL`):**
    * If both have `D` rows, approximately **O(D)** or **O(D log D)**.

---

### 5. Spatial Join (GeoPandas)

* **`gpd.sjoin(gdf_tracts_merged, gdf_districts, ...)`:**
    * This is a computationally intensive step. GeoPandas uses spatial indexing (e.g., R-tree) by default if the `rtree` library is installed, which significantly improves performance over a naive pairwise comparison.
    * With spatial indexing, the average-case complexity is often cited as roughly **O((N+M)log(N+M))** or **O(N log N + M log M + K)**, where `N` is the number of geometries in the left GeoDataFrame (tracts, so `T`), `M` is the number of geometries in the right GeoDataFrame (districts, so `D`), and `K` is the number of intersecting pairs found.
    * In the worst case (e.g., all geometries overlapping in a complex way), it could degrade. Without spatial indexing, it would be closer to O(T*D * Avg_Vertex_Comparison_Cost).

---

### 6. Geometric Operations (GeoPandas) for Polsby-Popper

* **Coordinate Reference System (CRS) Transformation (`.to_crs()`):**
    * Applied to `D` district geometries. The complexity for each geometry depends on its number of vertices. Overall: **O(V_districts)**.
* **Area Calculation (`.geometry.area`):**
    * Applied to `D` district geometries. Complexity for each depends on vertices. Overall: **O(V_districts)**.
* **Perimeter Calculation (`.geometry.length`):**
    * Applied to `D` district geometries. Complexity for each depends on vertices. Overall: **O(V_districts)**.
* The subsequent arithmetic operations to calculate the Polsby-Popper score for each district are **O(D)**.

---

### 7. Efficiency Gap Calculation

* This involves arithmetic operations on the `D` district-level aggregated vote counts (Dem_Votes, Rep_Votes, Total_Votes, Wasted_Votes).
* Complexity: **O(D)**.

---

### 8. Community Concentration Score Calculation

* **Identifying High-Concentration Tracts:**
    * Iterating through `T` tracts to check if `pct_minority > threshold`: **O(T)** for each minority group.
* **Grouping High-Concentration Tracts by District:**
    * `gdf_tracts_with_district[gdf_tracts_with_district[high_conc_col]].groupby(...).size()`: This operates on a subset of tracts (those above the threshold). If `T_high_conc` is the number of such tracts, this is roughly **O(T_high_conc)**.
* **Calculating Shares and Sum of Squared Shares:**
    * Operations on `D` districts: **O(D)**.
* Overall, for `M_groups` minority groups, the dominant part is `M_groups * O(T)`.

---

### Summary of Dominant Steps

For a typical state:
* `B_vf` (blocks in voter file) and `B_cvap` (blocks in CVAP) are usually the largest numbers.
* `T` (tracts) is smaller than `B_vf` or `B_cvap`.
* `D` (districts) is much smaller than `T`.

The most computationally intensive steps are likely to be:
1.  **Initial Data Loading and Block-Level Processing:** O(B_vf + B_cvap).
2.  **Block-to-Tract Aggregations:** O(B_vf + B_cvap) or O(B_vf log B_vf + B_cvap log B_cvap).
3.  **Spatial Join:** O((T+D)log(T+D) + K) on average with spatial indexing. This can be significant if `T` is large.
4.  **Geometric Operations for Polsby-Popper (CRS, area, perimeter):** O(V_districts).

The efficiency of merges and groupby operations also contributes, but they are often very optimized in Pandas/GeoPandas. The spatial join is often a bottleneck for large numbers of detailed geometries.
