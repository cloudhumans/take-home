# Welcome to the Cloud Humans Take-Home Assignment!

Hey there! We're super excited that you're interested in joining our all-star team at Cloud Humans. We've got some of the top engineers in Brazil, and we're looking forward to seeing what you can bring to the table. We've put together this fun little challenge to give you a taste of what we do — no complicated-to-understand tasks or hidden gotchas here. Just a straightforward project to let your skills shine. Let's get started! 🚀

## About Us

Cloud Humans is an innovative AI platform specializing in customer support through text channels. We're leading the revolution in customer service by providing advanced, AI-driven Customer Experience solutions for SaaS, Fintech, and tech-driven companies like Nuvemshop, CRM&BONUS, Hotmart, Zippi, and Insider.

By integrating generative AI with Reinforcement Learning from Human Feedback (RLHF), our platform enables brands to respond to 50% to 85% of customer messages without human assistance (averaging 65%), all while maintaining human-like quality. We manage around 400,000 support tickets per month and hit the 3 million ticket mark in August 2024. Our founders are former execs from Uber, Rappi, Creditas, and Loft, backed by investors like Canary VC, Grão, and Y Combinator.

## The Challenge

Your task is to develop an API with an endpoint that returns appropriate chatbot responses based on content for a specific company. In this challenge, our mock customer is Tesla Motors, but your solution should be designed to accommodate others as well. Imagine you're building this foundational component as the first step in a larger architecture that doesn't exist yet, but may look like this.

![image](https://github.com/user-attachments/assets/6b2a826c-6630-4424-b08c-e295d6232770)

### Sample Request

```http
POST /conversations/completions
{
    "helpdeskId": 123456,
    "projectName": "tesla_motors",
    "messages": [
        {
            "role": "USER",
            "content": "Hello! How long does a Tesla battery last before it needs to be replaced?"
        }
    ]
}
```

### Sample Response

```http
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
    "handoverToHumanNeeded": false,
    "sectionsRetrieved": [
        { "score": 0.6085123, "content": "How do I know if my Tesla battery needs replacement? Tesla batteries are designed to last many years; the vehicle will notify you if maintenance is needed." },
        { "score": 0.5785547, "content": "What is Tesla's battery warranty? Tesla’s battery warranty typically lasts for 8 years or about 150,000 miles, depending on the model." },
        ...
    ]    
}
```

*Feel free to adjust the field names and formatting (snake_case, camelCase, etc.) to suit your preference—no tricks here! Our goal is to make it as straightforward as possible for you, regardless of the language or framework you choose.*


### What You'll Need to Do

- **Use RAG (Retrieval Augmented Generation)**: Implement this technique in your endpoint (simple explanation [here](https://lucvandonkersgoed.com/2023/12/11/retrieval-augmented-generation-rag-simply-explained/)).
- **Use Provided Resources**: We've supplied sample HTTP calls via CURL that you can import into Postman to speed things up—we don't want to waste your time reading docs.
- **Focus on Code and Solution**: We want to see your code—simplicity, readability, and maintainability are key. Please include explanations of the decisions you made and how it could evolve. For that purpose, provide a README explaining your ideas. 

  *In your README file, please include:*

  1. **Instructions to Run the Code**: Step-by-step guide on how to set up and run your project (docker would be awesome).
  2. **Main Technical Decisions**: Explain the key choices you made during development.
  3. **Relevant Comments About Your Project**: Any additional insights or considerations.

- **Use English**: Please use English in your code and documentation.

## Rules for the AI Response

1. **Use Only IDS Content**: The AI should not use any information outside of the provided Improved Data Set (IDS) when generating responses.

### Choose one of the following features to implement:

#### Option 1: Clarification Feature

2. **Clarify When Unsure**: If the AI doesn't have enough info to provide a solid answer, it should ask the user for more details.

3. **Limit Clarifications**: The AI can make up to **2 clarifications** per conversation. If a 3rd is needed, it should inform the user that the ticket will be escalated to a human specialist and set `handoverToHumanNeeded: true` in the response.

#### Option 2: Smart Handover Feature

4. **Automatic escalation for N2 content**: Whenever a user asks about content labeled with type **N2** by the Vector DB, the API should return `handoverToHumanNeeded: true` along with the answer. Note: N1 content types are those which the AI should be able to answer by itself entirely; N2 are the content that should be redirected to a human after being replied to the user.

**Note**: Your solution doesn't need to be rocket science. It can be as simple as some IF statements if you think it would be a good start. Think of what you're building as an MVP; it doesn't have to be perfect. If you have some cool ideas that would be hard to implement or test, please share them in the README.

# What We Provide

- **OpenAI API Key**: We'll provide you with an API key to use OpenAI's `text-embedding-ada-002` model for embeddings and the `gpt-4` model for chat completions.
- **Vector DB Key**: We refer to our FAQs as the "Improved Data Set" (IDS) — think of it as our beefed-up FAQ on steroids. For this challenge, we've prepared an [IDS](https://docs.google.com/spreadsheets/d/1SbLV3OA6m3dYery6AqgPruxYTjEZ5TJlrtxK7bn8pEA/edit?usp=sharing) that's already populated in our vector database (hosted on Azure AI Search), so you don't need to create or maintain any content for this challenge.
  
Here are the request samples you will need in your application (tip: import them on postman to test and understand how they work):

### Embed the User Question

Use the `$OPEN_AI_KEY` we sent you.

```bash
curl https://api.openai.com/v1/embeddings \
  -H "Authorization: Bearer $OPEN_AI_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "text-embedding-3-large",
    "input": "How long does a Tesla battery last before it needs to be replaced?"
  }'
```

### Semantic Search on Vector DB

Replace `[...embeddings here...]` with the vector returned by the embeddings model. Use the `$AZURE_AI_SEARCH_KEY` we sent you.

```bash
curl --location --request POST 'https://claudia-db.search.windows.net/indexes/claudia-ids-index-large/docs/search?api-version=2023-11-01' \
--header 'Content-Type: application/json' \
--header 'api-key: $AZURE_AI_SEARCH_KEY' \
--data-raw '{
    "count": true,
    "select": "content, type",
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
  -H "Authorization: Bearer $OPEN_AI_KEY" \
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
