# System, User & Assistant Messages

## 1. What is it?

An LLM chat request contains a list of messages, each with a role:

- **system**: instructions from the APP (behavior, rules, persona). Usually hidden from the user.
- **user**: what the end user says.
- **assistant**: the model's previous replies (sent back in later requests).

## 2. Why does it exist?

One flat text blob can't separate "app rules" from "user input". Roles give the model a clear frame: whose voice is speaking, and which words are rules vs data.

## 3. Simple Mental Model

System = the manager's briefing note pinned above the chat. User = the customer talking. Assistant = the reply. The model sees the whole transcript every time.

## 4. How it Works

```
messages = [
  {role:"system",    content:"You are a banking support agent. Never give investment advice."},
  {role:"user",      content:"What is a fixed deposit?"},
  {role:"assistant", content:"A fixed deposit is..."},   <- from a previous call
  {role:"user",      content:"And is it better than mutual funds?"}
]
```

Each new request re-sends the whole history — the model has NO memory between API calls.

## 5. ASCII: Stateless memory illusion

```
Call 1: [system, user1]            -> assistant1
Call 2: [system, user1, asst1, user2] -> assistant2
Call 3: [system, ...everything..., user3] -> assistant3

Memory? NO. The app re-sends history every time.
Context window fills up -> must trim/summarize old messages.
```

## 6. Important Terms

- **Stateless**: the API remembers nothing between calls; your app owns the history.
- **Conversation history**: the messages array your backend stores (DB/cache).
- **Context management**: trimming/summarizing history as it approaches the token limit.

## 7. What happens internally?

All messages are concatenated into one token sequence with role markers. Attention treats system instructions as the frame; the model generates the next "assistant" turn.

## 8. How is it different?

### Prompt vs System Prompt (common confusion!)

| | Prompt (general) | System Prompt |
|---|---|---|
| Who writes | Anyone, any message | The app developer |
| When | Per request | Fixed, app-level |
| User sees it | Yes | Usually no |
| Purpose | One specific task | Persistent behavior/rules |

### Memory vs Context

| Memory | Context |
|---|---|
| Stored across sessions (DB, vector store) | Only within one request |
| App-managed | Sent each call |
| Long-term | Short-term window |

## 9. Common Misunderstandings

- ❌ "The model remembers our chat." It doesn't — your app re-sends history.
- ❌ "System prompt is a special hidden channel." It's just text with a special role — and it can leak or be overridden by clever attacks, so put hard rules in code too.
- ❌ "User messages and system have equal authority." System usually dominates, but defense-in-depth matters (see prompt injection later).

## 10. Connection to Previous

Previous: the request/response lifecycle. Messages are the actual content structure inside that request.

## 11. Connection to Next

Which model you pick, what it costs per token, and how you stream output — the practical choices of integration.

## 12. One-Minute Explanation

"Every LLM request carries a messages array with roles: system for app-level rules, user for the end user, assistant for previous replies. The API is stateless — my backend stores the history and re-sends it every call, which is why the model 'seems' to remember. The system prompt differs from a normal prompt: it's written by the developer, fixed at app level, and shapes behavior across all conversations."
