# WhalyticsSession Usage Guide

## Overview

`WhalyticsSession` provides a convenient way to manage session-level data (user_id, session_id, and user properties) and automatically include them in all events created from that session.

## Benefits

- **Less repetition**: Set user_id, session_id, and user properties once per session
- **Consistency**: All events from the same session automatically share the same properties
- **Convenience**: Simpler API for creating multiple events with shared context

## Basic Usage

### Creating a Session

```rust
use whalytics_sdk::WhalyticsSession;

// Simple session creation
let session = WhalyticsSession::new("user_123", "session_456");
```

### Creating a Session with User Properties

```rust
use whalytics_sdk::WhalyticsSessionBuilder;
use serde_json::json;
use std::collections::HashMap;

let mut user_props = HashMap::new();
user_props.insert("platform".to_string(), json!("rust"));
user_props.insert("version".to_string(), json!("1.0"));

let session = WhalyticsSessionBuilder::default()
    .user_id("user_123")
    .session_id("session_456")
    .user_properties(user_props)
    .build()
    .unwrap();
```

### Adding User Properties Dynamically

```rust
let mut session = WhalyticsSession::new("user_123", "session_456");

// Add user properties one at a time
session.set_user_property("platform", json!("rust"));
session.set_user_property("subscription_type", json!("premium"));
session.set_user_property("level", json!(10));
```

### Creating Events from a Session

```rust
// Simple event - user_id, session_id, and user_properties are automatically included
let event = session.event("level_started").build().unwrap();

// Event with additional event properties
let mut event_props = HashMap::new();
event_props.insert("level_id".to_string(), json!(5));
event_props.insert("score".to_string(), json!(1500));

let event = session.event("level_completed")
    .event_properties(event_props)
    .build()
    .unwrap();
```

## Complete Example

```rust
use whalytics_sdk::{WhalyticsClient, WhalyticsSession};
use serde_json::json;
use std::collections::HashMap;

fn main() {
    // Initialize client
    let mut client = WhalyticsClient::new("YOUR_API_KEY");
    
    // Create a session
    let mut session = WhalyticsSession::new("user_123", "session_456");
    
    // Set user properties for the session
    session.set_user_property("platform", json!("rust"));
    session.set_user_property("subscription_type", json!("premium"));
    
    // Log multiple events - all will include user_id, session_id, and user_properties
    client.log_event(session.event("app_start").build().unwrap());
    
    let mut level_props = HashMap::new();
    level_props.insert("level_id".to_string(), json!(5));
    client.log_event(
        session.event("level_completed")
            .event_properties(level_props)
            .build()
            .unwrap()
    );
    
    // Send all events
    client.flush().unwrap();
}
```

## Comparison: Old vs New Way

### Old Way (without Session)

```rust
// Have to repeat user_id, session_id, and user_properties for each event
let event1 = WhalyticsEventBuilder::default()
    .event("level_started")
    .user_id("user_123")
    .session_id("session_456")
    .user_properties(user_props.clone())
    .build()
    .unwrap();

let event2 = WhalyticsEventBuilder::default()
    .event("level_completed")
    .user_id("user_123")
    .session_id("session_456")
    .user_properties(user_props.clone())
    .build()
    .unwrap();
```

### New Way (with Session)

```rust
// Set once, use many times
let session = WhalyticsSession::new("user_123", "session_456");
session.set_user_property("platform", json!("rust"));

let event1 = session.event("level_started").build().unwrap();
let event2 = session.event("level_completed").build().unwrap();
```

## API Reference

### WhalyticsSession Methods

- `new(user_id, session_id)` - Create a new session
- `event(event_name)` - Create an event builder with session properties pre-filled
- `set_user_property(key, value)` - Add or update a user property
- `user_id()` - Get the user_id
- `session_id()` - Get the session_id
- `user_properties()` - Get all user properties

### WhalyticsSessionBuilder

Use the builder pattern for more control:

```rust
WhalyticsSessionBuilder::default()
    .user_id("user_123")
    .session_id("session_456")
    .user_properties(props_map)
    .build()
    .unwrap()
```
