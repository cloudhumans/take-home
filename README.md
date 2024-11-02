# Welcome to the Cloud Humans Take-Home Assignment!

Hey there! We're super excited that you're interested in joining our all-star team at Cloud Humans. We've got some of the top engineers in Brazil, and we're looking forward to seeing what you can bring to the table. We've put together this fun little challenge to give you a taste of what we do—no complicated tasks or hidden gotchas here. Just a straightforward project to let your skills shine. Let's get started!

## About Us

Cloud Humans is an innovative AI platform specializing in customer support through text channels. We're leading the revolution in customer service by providing advanced, AI-driven Customer Experience solutions for SaaS, Fintech, and tech-driven companies like Nuvemshop, CRM&BONUS, Hotmart, Zippi, and Insider.

By integrating generative AI with Reinforcement Learning from Human Feedback (RLHF), our platform enables brands to respond to 50% to 85% of customer messages without human assistance (averaging 65%), all while maintaining human-like quality. We manage around 400,000 support tickets per month and hit the 3 million ticket mark in August 2024. Our founders are former execs from Uber, Rappi, Creditas, and Loft, backed by investors like Canary VC, Grão, and Y Combinator.

## The Challenge

Your task is to create an API with an endpoint that returns the appropriate chatbot response based on the content for a specific company. Our fake customer in this challenge will be Tesla Motors but your solution must accept others as well. At cloud humans

[image]

### What You'll Need to Do

- **Use [RAG (Retrieval Augmented Generation)](https://lucvandonkersgoed.com/2023/12/11/retrieval-augmented-generation-rag-simply-explained/)**: Implement this technique in your endpoint.
- **Leverage Provided Resources**: We've supplied sample HTTP calls via CURL that you can import into Postman to speed things up—we don't want to waste your time reading docs.
- **Focus on Code and Solution**: We want to see your code—simplicity, readability, and maintainability are key. Please include explanations of the decisions you made. For that purpose, provide a README explaining your ideas.

  *In your README file, please include:*

  1. **Instructions to Run the Code**: Step-by-step guide on how to set up and run your project (docker would be awesome).
  2. **Main Technical Decisions**: Explain the key choices you made during development.
  3. **Relevant Comments About Your Project**: Any additional insights or considerations.

- **Use English**: Please use English in your code and documentation.

### What We Provide

- **OpenAI API Key**: You'll get an API key to use OpenAI's `embeddings-large` and `GPT-4o` (chat completions) models.
- **Improved Data Set (IDS)**: We call our "FAQs" as "IDS". Think of it as our beefed-up FAQ. For this challenge, we've created an [IDS](https://docs.google.com/spreadsheets/d/1SbLV3OA6m3dYery6AqgPruxYTjEZ5TJlrtxK7bn8pEA/edit?usp=sharing) that's already populated in our Vector DB (hosted on Azure AI Search) so you don't need to create or mainting it for this challenge.

## Rules for the AI Response

1. **Clarify When Unsure**: If the AI doesn't have enough info to provide a solid answer, it should ask the user for more details.
2. **Limit Clarifications**: The AI can make up to **2 clarifications** per conversation. If a 3rd is needed, it should inform the user that the ticket will be escalated to a human specialist and set `handoverToHumanNeeded: true` in the response.

## Sample of expected request payload

```http
POST /conversations/completions
{    
    "projectName": "tesla_motors",
    "messages": [
        {
            "role": "USER",
            "content": "Hello! How long does a Tesla battery last before it needs to be replaced?"
        }
    ]
}
```

## Sample response

```http
HTTP/1.1 201 Created
{
    "messages": [
        {
            "role": "USER",
            "content": "Hello! How long does a Tesla battery last before it needs to be replaced?"
        },
        {
            "role": "AGENT",
            "content": "Hello! How can I assist you today? I'm Claudia, your Tesla support assistant 😊\nTesla batteries are designed to last many years; the vehicle will notify you if maintenance is needed! Let me know if you have more questions! 🚗⚡"
        }
    ],
    "sectionsUsed": [
        {
            "content": "How do I know if my Tesla battery needs replacement? Tesla batteries are designed to last many years; the vehicle will notify you if maintenance is needed."
        },
        {
            "content": "What is Tesla's battery warranty? Tesla’s battery warranty typically lasts for 8 years or about 150,000 miles, depending on the model."
        },
        {
            "content": "What happens if the Tesla battery completely drains? The car will need to be towed to a charging station."
        }
    ],
    "handoverToHumanNeeded": false
}
```

*Feel free to adjust the field names and formatting (snake_case, camelCase, etc.) to suit your preference—no tricks here! Our goal is to make it as straightforward as possible for you, regardless of the language or framework you choose.*

## Sample requests you will need to use in your endpoints

### Embed the User Question

Replace `OPEN_AI_KEY` with the private key we sent you.

```bash
curl https://api.openai.com/v1/embeddings \
  -H "Authorization: Bearer OPEN_AI_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "text-embedding-3-large",
    "input": "How long does a Tesla battery last before it needs to be replaced?"
  }'
```

### Semantic Search on Vector DB

Replace `[...embeddings here...]` with the vector returned by the embeddings model. Replace `AZURE_AI_SEARCH_KEY` with the private key we sent you.

```bash
curl --location --request POST 'https://claudia-db.search.windows.net/indexes/claudia-ids-index-large/docs/search?api-version=2023-11-01' \
--header 'Content-Type: application/json' \
--header 'api-key: AZURE_AI_SEARCH_KEY' \
--data-raw '{
    "count": true,
    "select": "content",
    "top": 10,
    "filter": "projectName eq '\''tesla_motors'\''",
    "vectorQueries": [
        {
            "vector": [...embeddings here...],
            "k": 3,
            "fields": "embeddings",
            "kind": "vector"
        }
    ]
}'
```

### Request to Chat Completions model

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer OPEN_AI_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      {"role": "system", "content": "You are a dumb assistant. Always say \"Hello!\" and nothing more"},
      {"role": "user", "content": "Hello! How long does a Tesla battery last before it needs to be replaced?"}
    ]
  }'
```

---

Have fun, and don't hesitate to reach out if you have any questions. We can't wait to see what you come up with! 😍
