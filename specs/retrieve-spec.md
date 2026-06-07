# Spec: `retrieve()`

**File:** `retriever.py`
**Status:** Spec incomplete — fill in all blank fields before implementing

---

## Purpose

Given a user's natural language query, find the most relevant chunks from the vector store using semantic similarity search. Return them ranked by relevance so that `generate_response()` can use them as context.

---

## Input / Output Contract

**Inputs:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | `str` | The user's natural language question |
| `n_results` | `int` | Maximum number of chunks to return (default: `N_RESULTS` from `config.py`) |

**Output:** `list[dict]`

Each dict in the returned list must contain exactly these keys:

| Key | Type | Description |
|-----|------|-------------|
| `"text"` | `str` | The chunk text |
| `"game"` | `str` | The game name this chunk came from |
| `"distance"` | `float` | Cosine distance score — lower means more similar to the query |

Results should be ordered from most to least relevant (lowest to highest distance). Returns an empty list `[]` if the collection contains no documents.

---

## Design Decisions

*Complete the fields below before writing any code. Use your AI tool in Plan or Ask mode to help you reason through what belongs here — but the decisions are yours.*

---

### Query approach

*Describe how you will use `_collection.query()` to find relevant chunks. What arguments will you pass, and why?*

```
I will pass in the query, the n_results, and include as ["documents", "metadatas", "distances]. This will provide the top n chunks and with the necessary corresponding information.
```

---

### Return structure

*Sketch out what one item in your return list looks like as a concrete example. Where does each field come from in the query results?*

```
The return list has the chunk text itself, the game it came from, and the cosine similarity with the query. A concrete example of this would be: 
{
    "text": "Players take turns rolling two dice and moving their token...",
    "game": "Monopoly",
    "distance": 0.12
}
"text" - results["documents"][0][i]
"game" - results["documents"][0][i]
"distance" - results["documents"][0][i]
```

---

### Handling the nested result structure

*`_collection.query()` returns nested lists. Describe what index you need to access to get the actual list of results for a single query, and why the nesting exists.*

```
_collection.query() provides a dictionary with several keys. They keys are "documents", "metadatas", "distances", and "ids". The value for each of these keys is a list with the length corresponding to the n values. So to find the document of the first query for example you would do results["documents][0].
```

---

### Relevance threshold

*Will you filter out results above a certain distance score, or return all `n_results` regardless of how relevant they are? What are the tradeoffs of each approach?*

```
I will filter out certain results, so they will have to meet a certain threshold to be outputed. I believe this is important because the user's question may not have an answer found in the rulebooks and outputing an answer regardless will likely not be relevant. The tradeoff to this is that an answer may not be given or if the threshold is set too high, you return too few results for valid questions. 
```

---

### Edge cases

*How does your implementation behave when: (a) the collection is empty, (b) the query matches no chunks well, (c) the query matches chunks from multiple games?*

```
a) Returns an empty list (no chunks). This happens before.query() is called as to not cause an error. 
b) Currently still provides the top n chunks
c) Currently still provides the multiple chunks, regardless of which game they are form. But each dict has a "game" field so generate_reponse() can distinguish where each of them came from. 
```

---

## Implementation Notes

*Fill this in after implementing, before moving to Milestone 3.*

**Test query and top result returned:**

```
Query: [your test query]
Top result game: Catan
Distance score: 0.471
Does it make sense? Yes
```

**One thing about the query results that surprised you:**

```
The last result had a pretty high similarity score (0.625) and is from the wrong game, indicating that there may be a better chunking strategy available. 
```
