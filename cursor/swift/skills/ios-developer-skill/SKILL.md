---
name: ios-swiftui-development
description: Implement features in SwiftUI apps including views, view models, networking, persistence and architecture decisions for iOS apps.
---

# SwiftUI Development Skill

Use this skill when implementing or modifying iOS application features.

This includes:
- SwiftUI views
- ViewModels
- Networking
- Persistence
- Refactoring iOS architecture

Default target: **iOS 17+**

Use modern APIs:
- NavigationStack
- Observation (@Observable)
- SwiftData
- async/await

---

# Implementation workflow

When implementing a feature follow these steps:

1. **Define the model**

Create or update the model in `/Models`.

Example:

```swift
struct TodoItem: Identifiable {
    let id: UUID
    var title: String
    var isCompleted: Bool
}
