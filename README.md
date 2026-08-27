# Movie Recommendation System

A production-style command-line GraphRAG application that answers movie questions and generates recommendations by combining structured relationships, semantic search, and an LLM.

## Why This Project

Most recommendation demos use only keyword matching or vector similarity. This project combines two complementary retrieval strategies:

- **Neo4j** stores movies, directors, actors, genres, themes, awards, and their relationships.
- **Pinecone** stores Gemini embeddings of movie descriptions for semantic retrieval.
- **Gemini** extracts entities, classifies intent, creates structured query plans, and explains the final answer.

The result is a system that can answer both precise graph questions and natural-language recommendation requests.

## Capabilities

- Ask factual questions such as: `Which movies did Christopher Nolan direct?`
- Explore relationships such as actors, directors, genres, themes, and awards.
- Find similar movies from natural-language descriptions.
- Resolve partial entity names such as `DiCaprio` to known graph entities.
- Route each question automatically to graph retrieval or similarity retrieval.
- Combine Pinecone candidates with Neo4j genre relationships before Gemini ranks and explains recommendations.
- Index a movie PDF into both the graph and vector store through one repeatable pipeline.

## Architecture

```text
                         +----------------+
Movie PDF -------------->| Gemini         |
                         | entity extract |
                         +-------+--------+
                                 |
                                 v
                         +----------------+
                         | Neo4j graph    |
                         | entities +     |
                         | relationships  |
                         +----------------+

Movie PDF --> parse/chunk --> Gemini embeddings --> Pinecone

User question --> entity resolution --> intent classification
                                      |                 |
                                      v                 v
                              Neo4j graph        Pinecone search
                                      |                 |
                                      +--------+--------+
                                               v
                                      Gemini grounded answer
```

## Project Structure

| File | Responsibility |
| --- | --- |
| `2_config.js` | Loads environment variables and initializes Neo4j, Pinecone, and Gemini clients |
| `3_pdfParser.js` | Parses the source PDF into movie blocks |
| `4_entityExtractor.js` | Extracts structured movie entities with Gemini |
| `5_graphBuilder.js` | Creates Neo4j nodes, relationships, and indexes using `MERGE` |
| `6_vectorStore.js` | Chunks the PDF, creates embeddings, retries failures, and upserts vectors |
| `9_entityResolver.js` | Resolves names and concepts against graph entities |
| `10_queryClassifier.js` | Classifies questions as graph or similarity queries |
| `11_graphHandler.js` | Handles factual and relationship queries in Neo4j |
| `12_similarityHandler.js` | Retrieves and ranks semantically similar movies |
| `13_runQuery.js` | Runs the interactive query experience |

## Prerequisites

- Node.js 20 or later
- A Neo4j database, such as Neo4j Aura
- A Pinecone index configured for 3072-dimensional vectors
- A Google Gemini API key with access to Gemini chat, embeddings, and the Files API
- The input dataset at `data/movies.pdf`

## Setup

1. Install dependencies:

   ```bash
   npm install
   ```

2. Create a local environment file:

   ```bash
   copy .env.example .env
   ```

3. Fill in the values in `.env`:

   ```text
   NEO4J_URI=neo4j+s://your-instance.databases.neo4j.io
   NEO4J_USERNAME=neo4j
   NEO4J_PASSWORD=your-neo4j-password
   PINECONE_API_KEY=your-pinecone-api-key
   PINECONE_INDEX_NAME=your-pinecone-index
   GEMINI_API_KEY=your-gemini-api-key
   ```

## Run It

Check all external services:

```bash
npm run test
```

Build the Neo4j graph and Pinecone vector store from the included PDF:

```bash
npm run index
```

Start the interactive query CLI:

```bash
npm run query
```

Type a question at the prompt. Type `exit` to close the application.

Example questions:

```text
Which movies were directed by Christopher Nolan?
Recommend movies similar to a mind-bending science-fiction thriller.
What awards did The Godfather win?
Which actors appear in movies with Leonardo DiCaprio?
```

## Engineering Highlights

- Uses Neo4j transactions and `MERGE` to avoid duplicate entities during indexing.
- Creates graph indexes for faster entity matching and writes.
- Uses deterministic vector IDs so repeated upserts update the same chunks.
- Adds retry handling and controlled concurrency for embedding requests.
- Uses an explicit query-classification and routing layer instead of sending every request to one retrieval method.
- Keeps connection setup centralized and closes external service connections after each CLI workflow.

## Current Scope

This is a portfolio and learning project designed to demonstrate GraphRAG architecture. The current indexer expects the included PDF format, approximately 1,000 movies, and movie blocks separated by dashed lines. The `npm run index` command currently uses `data/movies.pdf` directly.

`npm run test` is a live-service smoke test rather than a fully isolated automated test suite. Running the project requires valid credentials and provisioned Neo4j and Pinecone resources.

## Security

Credentials belong only in `.env`, which is ignored by Git. Use `.env.example` as the configuration template and never commit API keys or database passwords.

## Technologies

Node.js, JavaScript ES modules, Gemini, LangChain.js, Neo4j, Pinecone, `pdf-parse`, and dotenv.
