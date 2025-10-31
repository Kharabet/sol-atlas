# Lazy Thread Creation - User Flows

## Flow Diagrams

### Flow 1: Brand New User (No Threads)

```
┌─────────────────────────────────────────────────────────────┐
│ USER STARTS BOT                                             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    User types: /start
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ BOT RESPONSE                                                 │
│                                                              │
│ "👋 Welcome, John!                                          │
│                                                              │
│  What would you like to explore today? 🤔"                  │
│                                                              │
│  ┌────────────────────────┐                                │
│  │ ➕ Start New Chat       │  ← Only button shown           │
│  └────────────────────────┘                                │
│                                                              │
│  FSM State: waiting_for_first_message                       │
│  Active Thread: None                                        │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                User types: "how do I learn python?"
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ THREAD CREATION (Background)                                │
│                                                              │
│ 1. Detect: user in waiting_for_first_message state         │
│ 2. Generate name: "how do I learn python?"                 │
│    → LLM summarizes → "Learning Python" ✅                  │
│ 3. Create thread:                                           │
│    - thread_id: "abc-123"                                   │
│    - name: "Learning Python"                                │
│    - user_id: 922705                                        │
│ 4. Set as active thread                                     │
│ 5. Clear FSM state                                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ BOT RESPONSE                                                 │
│                                                              │
│ "✨ Started new chat!"                                      │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │ ➕ New Thread                                       │    │
│  ├────────────────────────────────────────────────────┤    │
│  │ 💬 Learning Python (1) │ ✏️ │ 🗑️                  │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
│ [Streaming LLM Response about learning Python...]          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                   User continues chatting
                   Messages go to "Learning Python" thread
```

---

### Flow 2: User Creates New Thread (Has Existing Threads)

```
┌─────────────────────────────────────────────────────────────┐
│ CURRENT STATE                                                │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │ ➕ New Thread                                       │    │
│  ├────────────────────────────────────────────────────┤    │
│  │ 💬 Learning Python (5) │ ✏️ │ 🗑️                  │    │
│  │    Research (12)        │ ✏️ │ 🗑️                  │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
│  Active Thread: "Learning Python"                           │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                User taps: "➕ New Thread"
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ BOT RESPONSE                                                 │
│                                                              │
│ "Let's dive in! What would you like to discuss? 🌟"        │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │ ➕ New Thread                                       │    │
│  ├────────────────────────────────────────────────────┤    │
│  │    Learning Python (5) │ ✏️ │ 🗑️                   │    │
│  │    Research (12)        │ ✏️ │ 🗑️                   │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
│  FSM State: waiting_for_first_message                       │
│  Active Thread: None (cleared)                              │
│  Old threads still visible ✅                               │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                User types: "best pizza recipes"
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ THREAD CREATION                                              │
│                                                              │
│ Generate name: "best pizza recipes"                        │
│ → "Best Pizza Recipes" ✅                                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ BOT RESPONSE                                                 │
│                                                              │
│ "✨ Started new chat!"                                      │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │ ➕ New Thread                                       │    │
│  ├────────────────────────────────────────────────────┤    │
│  │ 💬 Best Pizza Recipes (1) │ ✏️ │ 🗑️                │    │
│  │    Learning Python (5)     │ ✏️ │ 🗑️                │    │
│  │    Research (12)            │ ✏️ │ 🗑️                │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
│ [Streaming response about pizza recipes...]                │
└─────────────────────────────────────────────────────────────┘
```

---

### Flow 3: User Presses "Start New Chat" (Empty State Button)

```
┌─────────────────────────────────────────────────────────────┐
│ CURRENT STATE (No threads, just did /reset)                │
│                                                              │
│  ┌────────────────────────┐                                │
│  │ ➕ Start New Chat       │                                │
│  └────────────────────────┘                                │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                User taps: "➕ Start New Chat"
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ BOT RESPONSE                                                 │
│                                                              │
│ "I'm here to help! What's on your mind? 💭"                │
│                                                              │
│  ┌────────────────────────┐                                │
│  │ ➕ Start New Chat       │  ← Same button                 │
│  └────────────────────────┘                                │
│                                                              │
│  FSM State: waiting_for_first_message (refreshed)          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                User types: "hello"
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ THREAD CREATION                                              │
│                                                              │
│ Generate name: "hello"                                      │
│ → Too short → Fallback: "General Chat" ✅                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│ BOT RESPONSE                                                 │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │ ➕ New Thread                                       │    │
│  ├────────────────────────────────────────────────────┤    │
│  │ 💬 General Chat (1)    │ ✏️ │ 🗑️                   │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
│ "Hello! How can I help you today?"                         │
└─────────────────────────────────────────────────────────────┘
```

---

### Flow 4: Edge Case - Multiple Quick Messages

```
User State: waiting_for_first_message
Active Thread: None

User types quickly:
├─ Message 1: "what is"     (t=0ms)
├─ Message 2: "quantum"     (t=100ms)
└─ Message 3: "computing"   (t=200ms)

┌─────────────────────────────────────────────────────────────┐
│ HANDLER LOGIC (Race Condition Prevention)                   │
│                                                              │
│ Message 1 arrives:                                          │
│  ├─ Check state: waiting_for_first_message ✅               │
│  ├─ Acquire Redis lock: "thread_creation_lock:922705"      │
│  ├─ Create thread with name from "what is"                 │
│  ├─ Set thread_id = "xyz-789"                              │
│  ├─ Clear FSM state                                         │
│  ├─ Release lock                                            │
│  └─ Process message                                         │
│                                                              │
│ Message 2 arrives (100ms later):                           │
│  ├─ Check state: None (cleared by Msg 1) ❌                │
│  ├─ Get active thread: "xyz-789" ✅                         │
│  └─ Process message to existing thread                      │
│                                                              │
│ Message 3 arrives (200ms later):                           │
│  ├─ Check state: None ❌                                    │
│  ├─ Get active thread: "xyz-789" ✅                         │
│  └─ Process message to existing thread                      │
└─────────────────────────────────────────────────────────────┘

Result: Only ONE thread created ✅
All 3 messages in same thread ✅
Thread name based on first message ✅
```

---

## State Transitions

### FSM State Machine

```
┌─────────────────────────────────────────────────────────────┐
│                     NO STATE (Normal)                        │
│                                                              │
│  - User has active thread                                   │
│  - Messages go to active thread                             │
│  - Normal chat behavior                                     │
└─────────────────────────────────────────────────────────────┘
              ▲                                    │
              │                                    │
              │ Clear state after                  │ User presses
              │ thread created                     │ "New Thread"
              │                                    │ or /start with
              │                                    │ no threads
              │                                    ▼
┌─────────────────────────────────────────────────────────────┐
│          waiting_for_first_message                          │
│                                                              │
│  - No active thread                                         │
│  - Waiting for user's first message                         │
│  - Next message triggers thread creation                    │
│  - Welcome prompt shown                                     │
└─────────────────────────────────────────────────────────────┘
              ▲                                    
              │                                    
              │ /reset command or                  
              │ delete last thread                 
              │                                    
```

---

## Data Flow

### Thread Creation Pipeline

```
User Message: "how do I learn python?"
      │
      ▼
┌─────────────────────────────────────┐
│ streaming_dm.py Handler             │
│  - Check FSM state                  │
│  - Detect: waiting_for_first_message│
└─────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────┐
│ thread_name_generator.py            │
│  - Input: "how do I learn python?"  │
│  - Call LLM with prompt:            │
│    "Summarize in 3-5 words"        │
│  - Output: "Learning Python"        │
└─────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────┐
│ thread_service.py                   │
│  - create_thread(user_id, name)     │
│  - Save to Redis                    │
│  - Set as active                    │
│  - Return thread object             │
└─────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────┐
│ Clear FSM State                     │
│  - state.clear()                    │
└─────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────┐
│ Update Keyboard                     │
│  - Add new thread to menu           │
│  - Mark as active with 💬           │
│  - Send to user                     │
└─────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────┐
│ Continue Normal Streaming           │
│  - Process message                  │
│  - Stream LLM response              │
└─────────────────────────────────────┘
```

---

## Redis Keys Used

```python
# FSM state (Aiogram manages this)
"fsm:{bot_id}:{chat_id}:{user_id}:state" 
→ "ThreadCreationStates:waiting_for_first_message"

# Active thread pointer
"user_active_thread:{user_id}" 
→ "{thread_id}" or None

# Thread creation lock (prevent race conditions)
"thread_creation_lock:{user_id}" 
→ "locked" (TTL: 5 seconds)

# Thread data
"thread:{thread_id}" 
→ Thread object (hash)

# User's threads set
"user_threads:{user_id}" 
→ Set of thread_ids

# Thread conversation history
"thread_history:{thread_id}" 
→ List of messages
```

This comprehensive flow documentation should help with implementation!

