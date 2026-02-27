# CSE6242 Codebase Guidelines for AI Agents

## Project Overview
This repository contains coursework for Georgia Tech's CSE6242 Data & Visual Analytics class, including **HW1: Graph Construction from TMDb Movie Data** and **HW2: Data Visualization with D3**.

## Core Architecture: Co-Actor Network Graph Building

The project builds an undirected co-actor graph starting from actor **Laurence Fishburne (ID: 2975)**, expanding through movie collaborations from 1999.

**Key Components:**
- **Graph Class**: Represents co-actor networks with nodes (actors) and edges (movie collaborations)
- **TMDBAPIUtils Class**: Interfaces with The Movie Database API to fetch movie/cast data
- **Data Output**: Generates `nodes.csv` (id, name) and `edges.csv` (source, target) for submission

## Critical Implementation Patterns

### Graph Representation
- **Nodes**: Tuples of `(actor_id: str, actor_name: str)` - note: IDs are strings, not integers
- **Edges**: Tuples `(source_id, target_id)` where both are strings and smaller ID sorts first for undirected graphs
- **No duplicates**: Both nodes and edges must be validated against existing data before insertion
- **CSV Format**: Commas in actor names must be removed to prevent parsing errors

### API Integration Pattern
1. Always include `language=en-US` parameter in all API calls (prevents encoding issues)
2. Use `http.client.HTTPSConnection` with proper exception handling for timeout/connection errors
3. Cast limit of 5: Use the 'order' attribute (0-4) to select top-billed actors only
4. Handle missing data gracefully: Skip movies/credits without cast information

### Graph Building Algorithm (2-Phase Expansion)
```
Phase 1: Initialize with Laurence Fishburne, get his 1999 movie credits
Phase 2a: Add all co-actors (order 0-4) from those movies as nodes + edges to root
Phase 2b: For each new node, repeat: get their 1999 credits → add new co-actors
Phase 3: Repeat Phase 2b one more time (2 iterations total)
```
Expected graph size: ~307-707 nodes, ~-200 to +500 edges (due to live API variation)

## Developer Workflows

### Running & Testing
```bash
cd /workspaces/cse6242/HW1
python q1.py  # Executes graph building and test suite
```

**Tests include**: Graph method validation, duplicate prevention, degree calculation, API integration

### Data Validation
- Load generated CSVs: `Graph(with_nodes_file="nodes.csv", with_edges_file="edges.csv")`
- Verify no duplicate nodes/edges, correct format, actor names cleaned
- Check edge normalization: all edges follow `(smaller_id, larger_id)` format

## Key Gotchas & Best Practices

1. **Type consistency**: All IDs must be strings (TMDB returns integers; convert with `str()`)
2. **Edge normalization**: Always sort `(source, target)` tuple for undirected graph representation
3. **Rate limiting**: TMDB doesn't enforce strict limits, but may timeout on high-volume requests—add `time.sleep()` if needed
4. **Missing data**: Movies may lack cast info; wrap API calls in try-except and skip silently
5. **CSV output**: Use `csv.writer` or manual string concatenation; handle encoding as UTF-8

## File Structure
```
HW1/
  q1.py          # Graph class + TMDBAPIUtils class + graph building logic
  nodes.csv      # Generated: actor data (id, name)
  edges.csv      # Generated: co-actor relationships (source, target)
```

## Key API Endpoints
- `GET /3/movie/{id}/credits` - Movie cast list (use `?api_key=` + `&language=en-US`)
- `GET /3/person/{id}/movie_credits` - Actor's filmography (same params)

## Testing Strategy
When modifying graph building or API methods:
1. Test Graph methods independently (add_node, add_edge, max_degree_nodes)
2. Validate API responses with mock data before full integration
3. Compare output node/edge counts against expected ranges
4. Verify no duplicate nodes/edges in final CSV output

---

## HW2: Data Visualization with D3

### Q2: Force-Directed Graph Layout
**Goal**: Create an interactive network graph visualization of board game relationships using D3 version 5 with node pinning capabilities.

**Key Requirements:**
- **D3 Version**: Must use D3 v5 (available in `lib/d3.v5.min.js`)
- **Data Source**: `board_games.csv` - undirected graph with source, target, value (0=similar, 1=not similar)
- **Browser**: Chrome v131.0.0 or higher for grading
- **Local Testing**: Use Python HTTP server

**Implementation Components:**

1. **Node Labels** [2 points]
   - Display node name at top-right of each node in bold
   - Labels must move with dragged nodes
   - Update label position in tick function

2. **Edge Styling** [3 points]
   - value=0 (similar): gray, thick, solid line
   - value=1 (not similar): green, thin, dashed line
   - Style based on data value field in links array

3. **Node Scaling and Coloring** [3 points]
   - Radius: Scale based on node degree (linear or squared scale acceptable)
   - Color: Use gradient with ≥3 color gradations
   - Higher degree = darker/deeper color
   - Lower degree = lighter color
   - **Note**: D3 v5 doesn't support `d.weight`; calculate degrees manually from links array

4. **Node Pinning Interaction** [6 points]
   - Drag a node to pin (fix) its position
   - Pinned nodes can still be dragged, but don't move with layout algorithm
   - Visually distinguish pinned nodes with different color
   - Double-click pinned node to unpin and restore free movement
   - **Note**: Use `d.fixed` Boolean flag (deprecated from D3 v3) to track pin state
   - **Important**: Ensure autograder can detect pinned nodes; increase radius for highly-weighted nodes

5. **Credit Text** [1 point]
   - Add GT username in top-right corner
   - Must be `<text>` element with id="credit"

**Critical Gotchas:**
- D3 v5 doesn't support `d.weight` or `d.fixed` directly; implement manually
- Calculate node degrees by counting source/target occurrences in links
- For autograder compatibility: larger node radii and readable labels improve detection
- Use double-click handler to avoid timeout errors
- Use correct element IDs for circle nodes (e.g., "dorktower" for "Dork Tower")

**File Structure:**
```
HW2/
  Q2/
    Q2.html           # Complete solution (HTML + CSS + JS)
    board_games.csv   # Dataset (not submitted to Gradescope)
    Q2.css            # (Optional) Separate stylesheet
    Q2.js             # (Optional) Separate JavaScript
  lib/
    d3.v5.min.js      # D3 library (provided, not submitted)
```

**Testing Workflow:**
```bash
cd /workspaces/cse6242/HW2/Q2
python3 -m http.server 8000
# Open http://localhost:8000 in Chrome
```

**Color Scheme Recommendation:**
Use ColorBrewer palettes (e.g., YlOrRd, Blues, Greens) available at https://colorbrewer2.org for meaningful gradients.
