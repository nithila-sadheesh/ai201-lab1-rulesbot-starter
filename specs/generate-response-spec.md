# Spec: `generate_response()`

**File:** `generator.py`
**Status:** Spec incomplete — fill in all blank fields before implementing

---

## Purpose

Given a user query and a list of retrieved rule chunks, generate a response that directly answers the question using only the retrieved text as context. The response must be grounded — it should not draw on the model's general knowledge of board games, only on what was retrieved.

---

## Input / Output Contract

**Inputs:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | `str` | The user's original question |
| `retrieved_chunks` | `list[dict]` | Ranked list of chunks from `retrieve()`, each with `"text"`, `"game"`, and `"distance"` |

**Output:** `str`

A plain string containing the response to show the user. The response should:
- Answer the question using only the retrieved rule text
- Identify which game the answer comes from
- Acknowledge clearly when the answer is not found in the loaded rules

Returns a fallback string (not an error) when `retrieved_chunks` is empty.

---

## Design Decisions

*Complete the fields below before writing any code. Use your AI tool in Plan or Ask mode to help you reason through what belongs here — but the decisions are yours.*

---

### Context formatting

*How will you format the retrieved chunks before passing them to the LLM? Describe the structure — not the code. Consider: will you label chunks by game? Include distance scores? Separate chunks with delimiters?*

```
Each chunk is formatted with a labeled header showing its rank and source game, wrapped in XML-style tags, and separated by a blank line. Chunks are sorted most-to-least relevant as it has been shown that LLMs are best at using context from the beginning and end of text. Distance scores are not shown as the order itself shows relevance. 
```

---

### System prompt — grounding instruction

*Write the exact system prompt instruction you will use to prevent the model from answering beyond the retrieved text. This is the most important design decision in this function.*

```
 Provide answers from the retrieved context below alone. If the retrieved context does not contain a direct, specific answer, do not use  your general knowledge base or general game rulebooks at all. Do not supplement the retrieved text with any outside knowledge, even if you believe it to be correct. Instead, say "I couldn't find that in the loaded rule books."
```

---

### System prompt — citation instruction

*Write the exact instruction you will use to tell the model to identify which game its answer comes from.*

```
Provide the name of the game that the answer is coming from as a citation. 
```

---

### Fallback behavior

*What should the response say when the answer isn't found in the loaded rule books? Write the exact fallback message.*

```
Instead, say "I couldn't find that in the loaded rule books."
```

---

### Handling low-relevance chunks

*`retrieved_chunks` may include chunks with high distance scores (weak relevance). Will you filter these out before building context, pass them all in, or handle them another way? What are the tradeoffs?*

```
Currently pass them anyway, so there is always an answer provided. Add a threshold value of 0.75.
```

---

### Message structure

*Describe how you will structure the messages list for the API call — what goes in the system message vs. the user message?*

```
System message is all the prompt instructions while the user message is just their inputted query.
```

---

## Implementation Notes

*Fill this in after implementing and testing.*

**Test query and response:**

```
Query: Can two players claim the same route in Ticket to Ride?
Response: No, according to the retrieved context, the same player may not claim both parallel routes between two cities, but two different players may each claim one of the parallel routes.
Correctly grounded? Yes
Cited the right game? Yes
```

**One thing you changed from your original spec after seeing the actual output:**

```
Threshold. Do not say anything like "according to the retrieved context". 
```
