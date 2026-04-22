# Observation

## 1. What A2A messages were exchanged between agents (copy from logs)
```{json}
{
  "jsonrpc": "2.0",
  "method": "message/send",
  "params": {
    "message": {
      "role": "user",
      "parts": [
        {
          "kind": "text",
          "text": "I need to book flight ID 1 (United UA101, SF to NY on 2025-11-15) for 2 seats. Please reserve the seats, confirm the reservation, and process the payment."
        }
      ],
      "messageId": "afecfa225c454f08a200eb1f969d2a07"
    }
  }
}
```

## 2. How the Travel Assistant discovered the Flight Booking Agent

The Travel Assistant discovered the Flight Booking Agent through the Registry Stub. It called discover_remote_agents with the query "flight booking reservation confirmation payment processing", which sent a semantic search request to the Registry at http://127.0.0.1:7861. The Registry returned the Flight Booking Agent's info. The Travel Assistant then fetched its agent card from http://127.0.0.1:10002/.well-known/agent-card.json to get the endpoint and skills, and used that to send the A2A message.

## 3. The JSON-RPC request/response format you observed

The A2A protocol uses JSON-RPC 2.0 as its envelope format.The response follows the same JSON-RPC structure.

- **`jsonrpc`**: Always `"2.0"`.
- **`method`**: `"message/send"` — the standard A2A method for sending a message to an agent.
- **`params.message`**: Contains the actual message with:
  - `role`: `"user"` — indicating this is a request from the caller.
  - `parts`: An array of content parts, each with `kind` (e.g., `"text"`) and the content itself.
  - `messageId`: A unique UUID for tracking the message.


## 4. What information was in the agent card and how it was used

- `capabilities`: {"streaming": true}
- `defaultInputModes`: text
- `defaultOutputModes`: text
- `description`: Flight booking and reservation management agent
- `name`: Flight Booking Agent
- `preferredTransport`: JSONRPC
- `protocolVersion`: 0.3.0
- `skills`:
    - `check_availability`: Check seat availability for a specific flight
    - `reserve_flight`: Reserve seats on a flight for passengers
    - `confirm_booking`: Confirm and finalize a flight booking
    - `process_payment`: Process payment for a booking (simulated)
    - `manage_reservation`: Update, view, or cancel existing reservations
- `url`: http://127.0.0.1:10002/
- `version`: 0.0.1

**How it was used**: After discovery, the Travel Assistant fetched this agent card to learn the agent's endpoint URL, supported protocol, and available skills before sending the A2A message.

## 5. Your observations about the benefits and limitations of this approach

**Benefits:**
- Agents don't need to know each other at build time. The Travel Assistant discovers the Flight Booking Agent dynamically through the registry at runtime.
- JSON-RPC 2.0 provides a standardized, language-agnostic message format, so any agent implementing the A2A spec can participate.
- Each agent handles its own domain independently, making the system easier to develop and maintain.

**Limitations:**
- The registry is a stub that returns canned responses, not real semantic search.
- There is no authentication, which means any agent can invoke any other agent without verification.
- Both agents logged a streaming compatibility warning, indicating the Strands SDK A2A integration is still experimental.