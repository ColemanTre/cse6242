# CSE6242 Codebase Guidelines for AI Agents

## Project Overview
This repository contains coursework for Georgia Tech's CSE6242 Data & Visual Analytics class. Currently, it focuses on **HW1: Graph Construction from TMDb Movie Data**.

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
