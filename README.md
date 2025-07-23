# Knowledge Graph MCP Server

A Model Context Protocol (MCP) server that implements a knowledge graph-based memory system with AI-powered entity and relation extraction. The system uses structured ontologies to extract meaningful relationships from unstructured text.

## Features

- **Ontology-driven extraction** - Uses predefined ontologies to structure knowledge extraction
- **AI-powered relation extraction** - Leverages LLMs to extract entities and relationships from text
- **JSON persistence** - Automatic saving and loading from JSON files with configurable paths
- **MCP compatible** - Works with any MCP-enabled AI assistant
- **Multiple ontologies** - Ships with 6 predefined ontologies for different domains
- **Custom ontologies** - Support for user-defined ontologies
- **Local storage** - All data stored locally in JSON format

## Installation

```bash
git clone https://github.com/belindamo/kg_mcp
cd kg_mcp
pip install -e .
```

## MCP Configuration

Add to your MCP configuration:

```json
{
  "mcps": {
    "kg-memory": {
      "command": "fastmcp",
      "args": ["run", "server.py"],
      "env": {
        "ONTOLOGY": "basic",
        "KG_STORAGE_PATH": "./my_knowledge.json"
      }
    }
  }
}
```

### Available Ontologies

- **basic** (default) - Person, Organization, Location, Concept
- **messaging** - Sessions, Messages, Semantic Entities  
- **github** - Developers, Repositories, PRs, Issues, Commits
- **scientific** - Researchers, Experiments, Results, Publications
- **simple_person_project** - Simple Person-Project relationships
- **employment** - Employment relationships

### Custom Ontologies

You can provide a path to a custom ontology file:

```json
{
  "env": {
    "ONTOLOGY": "/path/to/my_ontology.py"
  }
}
```

The file must define an `ontology` variable of type `Ontology`.

### Storage Configuration

Control where the knowledge graph is stored:

```json
{
  "env": {
    "KG_STORAGE_PATH": "/path/to/my_knowledge.json"
  }
}
```

Default storage path is `./kg_memory.json` in the current directory.

## MCP Tools

The server provides four MCP tools:

### `add_memories(text_chunk: str) -> str`

Extracts structured knowledge from unstructured text and stores it in the knowledge graph.

```python
# Example usage through MCP
add_memories("John Smith works for Google. Google is located in Mountain View.")
# Returns: "Successfully extracted 2 memories:
# - John Smith (Person) works_for Google (Organization)  
# - Google (Organization) located_in Mountain View (Location)"
```

### `retrieve_relevant_context(query: str) -> str`

Retrieves relevant context from the knowledge graph based on a query.

```python
# Example usage through MCP
retrieve_relevant_context("Who works at Google?")
# Returns relevant relations and entities related to the query
```

### `save_knowledge_graph() -> str`

Manually saves the current knowledge graph to the JSON file.

```python
# Example usage through MCP
save_knowledge_graph()
# Returns: "Knowledge graph saved successfully to ./kg_memory.json. Contains 15 entities and 8 relations."
```

### `get_knowledge_stats() -> str`

Returns statistics about the current knowledge graph including entity/relation counts and ontology information.

```python
# Example usage through MCP
get_knowledge_stats()
# Returns detailed statistics about the knowledge graph
```

## Direct API Usage

You can also use the library directly in Python:

```python
from kg_mcp import KGMem, basic_ontology

# Initialize with an ontology and custom storage path
kg = KGMem(basic_ontology, storage_path="./my_custom_kg.json")

# Add unstructured text (automatically saves to JSON)
relations = kg.add_unstructured("Alice works for Microsoft. Microsoft is in Seattle.")

# Query the knowledge graph  
results = kg.retrieve_str("Who works at Microsoft?")

# Manually save if needed
kg.save_to_json()

# The data persists and will be loaded automatically next time
kg2 = KGMem(basic_ontology, storage_path="./my_custom_kg.json")
# kg2 now contains the previously saved data
```

## Ontology Structure

Each ontology defines:

- **EntityTypes** - Types of entities (Person, Organization, etc.)
- **RelationTypes** - Types of relationships with constraints
- **QueryTypes** - Predefined query patterns for retrieval

### Example: Basic Ontology

```python
from kg_mcp import EntityType, RelationType, Ontology

# Entity types
person_type = EntityType(name="Person")
organization_type = EntityType(name="Organization")

# Relation types with constraints
works_for = RelationType(
    name="works_for",
    entity_type0=person_type,      # Person
    entity_type1=organization_type, # works for Organization
    context="Employment relationships"
)

# Complete ontology
ontology = Ontology(
    entity_types=[person_type, organization_type],
    relation_types=[works_for],
    query_types=[]
)
```

## AI Configuration

The system uses DSPy for LLM integration. Default model is `gemini/gemini-2.5-flash`. You can configure AI settings:

```python
kg = KGMem(
    ontology=basic_ontology,
    ai_config={
        "model": "gpt-4",
        "temperature": 0.1,
        "api_key": "your-key"
    }
)
```

## Examples

### Example 1: Employment Tracking
```python
from kg_mcp import KGMem, employment_ontology

kg = KGMem(employment_ontology)

# Add employment information
kg.add_unstructured("""
Sarah Johnson is the new CTO at TechStart Inc. 
Bob Wilson also joined TechStart as a Senior Engineer.
TechStart Inc. was recently acquired by MegaCorp.
""")

# Query employment relationships
results = kg.retrieve_str("Who works at TechStart?")
```

### Example 2: GitHub Project Tracking
```python
from kg_mcp import KGMem, github_ontology

kg = KGMem(github_ontology)

# Add development activity
kg.add_unstructured("""
Alice opened a pull request to fix the authentication bug.
The PR targets the main branch and fixes issue #123.
Bob reviewed the pull request and approved it.
""")

# Query development workflow
results = kg.retrieve_str("What pull requests are open?")
```

### Example 3: Scientific Research
```python
from kg_mcp import KGMem, scientific_ontology

kg = KGMem(scientific_ontology)

# Add research information
kg.add_unstructured("""
Dr. Smith conducted experiments on quantum entanglement.
The experiments produced breakthrough results on coherence time.
Dr. Smith collaborated with Dr. Johnson on the theoretical analysis.
""")

# Query research activities
results = kg.retrieve_str("What experiments did Dr. Smith conduct?")
```

## Running the Server

Start the MCP server:

```bash
# With default basic ontology
fastmcp run server.py

# With specific ontology and custom storage
ONTOLOGY=github KG_STORAGE_PATH=./github_knowledge.json fastmcp run server.py

# With custom ontology
ONTOLOGY=/path/to/custom_ontology.py fastmcp run server.py
```

## Testing

Run the test suite:

```bash
cd tests
python run_all_tests.py
```

Individual test files:
- `test_mcp.py` - MCP server integration tests
- `test_add_unstructured.py` - Text extraction tests
- `test_retrieve.py` - Knowledge retrieval tests
- `test_*_ontology.py` - Ontology-specific tests

## Architecture

The system consists of:

1. **KGMem** - Core knowledge graph memory class
2. **AI** - DSPy-based extraction and retrieval engine
3. **Ontologies** - Structured type definitions
4. **MCP Server** - FastMCP-based server implementation

## Future Enhancements

- Visualizations for the graph
- Persistent storage backends (currently in-memory)

## Future experiments
- Better retrieval methods than in-context retrieval such as cosine similarity over vector embeddings and BM-25 over text chunks
- Add a function to discover a new ontology from unstructured text


## License

MIT
