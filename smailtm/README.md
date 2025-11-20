# SMailTM - Mail.tm API Wrapper for Android

[![](https://jitpack.io/v/samyak2403/SMailTM.svg)](https://jitpack.io/#samyak2403/SMailTM)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![API](https://img.shields.io/badge/API-24%2B-brightgreen.svg?style=flat)](https://android-arsenal.com/api?level=24)

A powerful and easy-to-use Kotlin library for Android that provides a complete wrapper around the [Mail.tm](https://mail.tm) API. Create temporary email addresses, receive messages, and manage accounts programmatically.

## Features

- 📧 Create temporary email accounts
- 🔐 Secure authentication with JWT tokens
- 📨 Fetch and manage messages
- 🔔 Real-time message notifications via Server-Sent Events (SSE)
- 🚀 Asynchronous operations support
- 🎯 Simple and intuitive API
- 🔄 Automatic domain management
- 📱 Android-optimized with Kotlin

## Installation

### Step 1: Add JitPack repository

Add the JitPack repository to your root `build.gradle` or `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://jitpack.io") }
    }
}
```

### Step 2: Add the dependency

Add the dependency to your module's `build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.github.samyak2403:SMailTM:1.0.0")
}
```

## Quick Start

### 1. Create a New Account

```kotlin
import com.samyak.smailtm.util.SMailBuilder
import javax.security.auth.login.LoginException

try {
    // Create account with random email
    val mailTM = SMailBuilder.createDefault("your_password")
    mailTM.init()
    
    // Get account details
    val account = mailTM.getSelf()
    println("Email: ${account.address}")
    println("ID: ${account.id}")
} catch (e: LoginException) {
    e.printStackTrace()
}
```

### 2. Login to Existing Account

```kotlin
try {
    val mailTM = SMailBuilder.login("your_email@domain.com", "your_password")
    println("Logged in successfully!")
} catch (e: LoginException) {
    e.printStackTrace()
}
```

### 3. Login with Token

```kotlin
try {
    val mailTM = SMailBuilder.loginWithToken("your_jwt_token")
    println("Authenticated with token!")
} catch (e: LoginException) {
    e.printStackTrace()
}
```

## Usage Examples

### Fetch Messages

```kotlin
// Fetch all messages
mailTM.asyncFetchMessages(object : MessageFetchedCallback {
    override fun onMessagesFetched(messages: List<Message>) {
        messages.forEach { message ->
            println("From: ${message.from.address}")
            println("Subject: ${message.subject}")
            println("Body: ${message.text}")
        }
    }
    
    override fun onError(response: Response) {
        println("Error: ${response.response}")
    }
})

// Fetch limited number of messages
mailTM.asyncFetchMessages(5, object : MessageFetchedCallback {
    override fun onMessagesFetched(messages: List<Message>) {
        println("Fetched ${messages.size} messages")
    }
    
    override fun onError(response: Response) {
        println("Error: ${response.response}")
    }
})
```

### Get Message by ID

```kotlin
try {
    val message = mailTM.getMessageById("message_id")
    println("Subject: ${message.subject}")
    println("From: ${message.from.address}")
    println("Body: ${message.text}")
} catch (e: MessageFetchException) {
    e.printStackTrace()
}
```

### Real-time Message Listener

```kotlin
// Open event listener for real-time notifications
mailTM.openEventListener(object : EventListener {
    override fun onReady() {
        println("Event listener is ready!")
    }
    
    override fun onMessageReceived(message: Message) {
        println("New message received!")
        println("From: ${message.from.address}")
        println("Subject: ${message.subject}")
    }
    
    override fun onError(error: String) {
        println("Error: $error")
    }
    
    override fun onClose() {
        println("Event listener closed")
    }
})

// Close the listener when done
mailTM.closeMessageListener()
```

### Account Management

```kotlin
// Get account information
val account = mailTM.getSelf()
println("Email: ${account.address}")
println("Created: ${account.createdAt}")

// Get total messages count
val totalMessages = mailTM.getTotalMessages()
println("Total messages: $totalMessages")

// Delete account
mailTM.asyncDelete(object : WorkCallback {
    override fun workStatus(status: Boolean) {
        if (status) {
            println("Account deleted successfully")
        } else {
            println("Failed to delete account")
        }
    }
})
```

### Create Custom Account

```kotlin
try {
    // Create account with specific email
    val created = SMailBuilder.create("myemail@domain.com", "password123")
    if (created) {
        println("Account created successfully!")
        
        // Now login
        val mailTM = SMailBuilder.login("myemail@domain.com", "password123")
    }
} catch (e: LoginException) {
    e.printStackTrace()
}

// Or create and login in one step
try {
    val mailTM = SMailBuilder.createAndLogin("myemail@domain.com", "password123")
    println("Account created and logged in!")
} catch (e: LoginException) {
    e.printStackTrace()
}
```

## API Reference

### SMailBuilder

Static methods for account creation and authentication:

- `login(email: String, password: String): SMailTM` - Login to existing account
- `loginWithToken(token: String): SMailTM` - Login with JWT token
- `create(email: String, password: String): Boolean` - Create new account
- `createAndLogin(email: String, password: String): SMailTM` - Create and login
- `createDefault(password: String): SMailTM` - Create account with random email

### SMailTM

Main class for API operations:

#### Account Operations
- `init()` - Initialize the instance (required for createDefault)
- `getSelf(): Account` - Get current account details
- `getAccountById(id: String): Account` - Get account by ID
- `delete(): Boolean` - Delete account synchronously
- `asyncDelete()` - Delete account asynchronously
- `asyncDelete(callback: WorkCallback)` - Delete with callback

#### Message Operations
- `getTotalMessages(): Int` - Get total message count
- `getMessageById(id: String): Message` - Get specific message
- `fetchMessages(callback: MessageFetchedCallback)` - Fetch all messages
- `fetchMessages(limit: Int, callback: MessageFetchedCallback)` - Fetch limited messages
- `asyncFetchMessages(callback: MessageFetchedCallback)` - Async fetch all
- `asyncFetchMessages(limit: Int, callback: MessageFetchedCallback)` - Async fetch limited

#### Event Handling
- `openEventListener(eventListener: EventListener)` - Open SSE listener
- `openEventListener(eventListener: EventListener, retryInterval: Long)` - Open with custom retry
- `closeMessageListener()` - Close event listener

## Data Models

### Account
```kotlin
data class Account(
    val id: String,
    val address: String,
    val quota: Int,
    val used: Int,
    val isDisabled: Boolean,
    val isDeleted: Boolean,
    val createdAt: String,
    val updatedAt: String
)
```

### Message
```kotlin
data class Message(
    val id: String,
    val accountId: String,
    val msgid: String,
    val from: Sender,
    val to: List<Receiver>,
    val subject: String,
    val intro: String,
    val seen: Boolean,
    val isDeleted: Boolean,
    val hasAttachments: Boolean,
    val size: Int,
    val downloadUrl: String,
    val createdAt: String,
    val updatedAt: String,
    val text: String,
    val html: List<String>,
    val attachments: List<Attachment>
)
```

## Callbacks

### EventListener
```kotlin
interface EventListener {
    fun onReady()
    fun onMessageReceived(message: Message)
    fun onError(error: String)
    fun onClose()
}
```

### MessageFetchedCallback
```kotlin
interface MessageFetchedCallback {
    fun onMessagesFetched(messages: List<Message>)
    fun onError(response: Response)
}
```

### WorkCallback
```kotlin
interface WorkCallback {
    fun workStatus(status: Boolean)
}
```

## Requirements

- Android API 24+
- Kotlin 1.9+
- Java 11+

## Dependencies

This library uses:
- OkHttp 4.12.0 - HTTP client
- Gson 2.10.1 - JSON parsing
- EventSource 2.5.0 - Server-Sent Events
- SLF4J & Logback - Logging

## Permissions

Add to your `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## ProGuard

If you're using ProGuard, add these rules:

```proguard
-keep class com.samyak.smailtm.** { *; }
-keepclassmembers class com.samyak.smailtm.** { *; }
```

## Error Handling

The library throws specific exceptions:

- `LoginException` - Authentication failures
- `AccountNotFoundException` - Account not found
- `MessageFetchException` - Message retrieval errors
- `DomainNotFoundException` - Domain issues
- `DateTimeParserException` - Date parsing errors

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

```
MIT License

Copyright (c) 2024 Samyak

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## Credits

Based on the original [JMailTM](https://github.com/shivam1608/JMailTM) library.

## Support

For issues, questions, or contributions, please visit the [GitHub repository](https://github.com/samyak2403/SMailTM).

---

Made with ❤️ by [Samyak](https://github.com/samyak2403)

