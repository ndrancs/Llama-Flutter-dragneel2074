# Chat Template Integration

## Overview

This app uses the **automatic chat template** feature from `llama_flutter_android` plugin. Templates are automatically detected from model filenames and applied correctly for 11+ model families.

## How It Works

### 1. Automatic Detection

When you load a model, the template is auto-detected from the filename:

```
Qwen2-0.5B-Instruct-Q4_K_M.gguf  → ChatML template
Llama-3-8B-Instruct.gguf         → Llama-3 template
Mistral-7B-Instruct.gguf         → Mistral template
```

No manual configuration needed!

### 2. Message Structure

The app uses `ChatMessage` objects with proper roles:

```dart
// System message (optional, set once)
ChatMessage(role: 'system', content: 'You are a helpful assistant.')

// User messages
ChatMessage(role: 'user', content: 'Hello!')

// Assistant responses
ChatMessage(role: 'assistant', content: 'Hi there!')
```

### 3. Conversation History

The app maintains full conversation context automatically:

- System message persists across the conversation
- User and assistant messages are added to history
- History is passed to `generateChat()` for context-aware responses
- Clear chat keeps system message but removes conversation

## Usage in the App

### Sending Messages

```dart
// In chat_service.dart
void sendMessage(String message) {
  // Adds user message to history
  _conversationHistory.add(ChatMessage(role: 'user', content: message));
  
  // Generates with auto-detected template
  final stream = _llama!.generateChat(
    messages: _conversationHistory,  // Full context
    maxTokens: 512,
    temperature: 0.7,
    topP: 0.9,
  );
  
  // Streams tokens, then adds assistant response to history
}
```

### Clearing History

```dart
// Keeps system message, clears conversation
chatService.clearHistory();
```

### Custom System Message

```dart
// Set during initialization
await chatService.initialize(
  systemMessage: 'You are a coding assistant expert in Dart and Flutter.'
);

// Or update later
chatService.setSystemMessage('You are a creative writing assistant.');
```

## Supported Models

The app works with these model families (auto-detected):

| Model Family | Template | Example Filename |
|-------------|----------|------------------|
| **Qwen / Qwen2** | ChatML | `Qwen2-0.5B-Instruct-Q4_K_M.gguf` |
| **Llama-3** | Llama-3 | `Llama-3-8B-Instruct.gguf` |
| **Llama-2** | Llama-2 | `Llama-2-7B-chat.gguf` |
| **QwQ-32B** | QwQ | `QwQ-32B-Preview-Q4_K_M.gguf` |
| **Mistral** | Mistral | `Mistral-7B-Instruct-v0.3.gguf` |
| **DeepSeek** | DeepSeek | `DeepSeek-Coder-6.7B-Instruct.gguf` |
| **Phi-2/3** | Phi | `Phi-3-mini-4k-instruct.gguf` |
| **Gemma** | Gemma | `gemma-2b-it.gguf` |
| **Alpaca** | Alpaca | `alpaca-7B.gguf` |
| **Vicuna** | Vicuna | `vicuna-7B-v1.5.gguf` |

## Advanced Usage

### Manual Template Override

If auto-detection doesn't work:

```dart
// In chat_service.dart, update sendMessage signature:
void sendMessage(String message, {String? template}) {
  final stream = _llama!.generateChat(
    messages: _conversationHistory,
    template: template,  // e.g., 'chatml', 'llama3'
    maxTokens: 512,
  );
}
```

### Get Supported Templates

```dart
final templates = await chatService.getSupportedTemplates();
print(templates);
// ['chatml', 'qwen', 'llama3', 'llama2', 'phi', 'gemma', 'alpaca', 'vicuna']
```

## Benefits

✅ **Zero manual formatting** - No template syntax to remember  
✅ **Multi-model support** - Works with 11+ model families  
✅ **Type-safe** - Compile-time validation with `ChatMessage`  
✅ **Context-aware** - Full conversation history automatically managed  
✅ **Stream responses** - Real-time token generation with proper formatting  
✅ **Stop generation** - Cancel mid-response while preserving partial output  

## Implementation Details

### File: `lib/services/chat_service.dart`

**Key Features:**
- `List<ChatMessage> _conversationHistory` - Maintains full conversation
- `generateChat()` - Uses automatic template formatting
- `sendMessage()` - Adds user message, generates response, adds assistant reply
- `clearHistory()` - Resets conversation while keeping system message
- `stopGeneration()` - Cancels stream and stops native generation
- `setSystemMessage()` - Updates system prompt

### File: `lib/main.dart`

**UI Features:**
- Displays messages in chat bubbles (user on right, AI on left)
- Shows typing indicator during generation
- Stop button to cancel generation
- Clear chat with confirmation dialog
- Timestamps for all messages

## Migration Notes

### Before (Manual Templates)

```dart
// OLD: Manual template formatting
final prompt = '''
<|im_start|>system
You are helpful.<|im_end|>
<|im_start|>user
$userInput<|im_end|>
<|im_start|>assistant
''';
controller.generate(prompt: prompt);
```

### After (Automatic)

```dart
// NEW: Automatic template formatting
final messages = [
  ChatMessage(role: 'system', content: 'You are helpful.'),
  ChatMessage(role: 'user', content: userInput),
];
controller.generateChat(messages: messages);
```

## Troubleshooting

### Model Generates Gibberish

**Cause:** Wrong template or corrupted model  
**Solution:** Try manual template override:

```dart
chatService.sendMessage('Hello', template: 'chatml');
```

### Model Doesn't Respond

**Cause:** Missing system message  
**Solution:** Always initialize with system message:

```dart
await chatService.initialize(
  systemMessage: 'You are a helpful AI assistant.'
);
```

### Template Not Detected

**Cause:** Unusual model filename  
**Solution:** Use manual override or check supported templates:

```dart
final templates = await chatService.getSupportedTemplates();
// Pick the right one and pass to sendMessage
```

## References

- [Plugin Documentation](../llama_flutter_android/CHAT_TEMPLATE_QUICK_START.md)
- [Complete Template Guide](../llama_flutter_android/CHAT_TEMPLATES.md)
- [Supported Models List](../llama_flutter_android/CHAT_TEMPLATES.md#supported-templates)

## Latest Updates (Oct 8, 2025)

- ✅ ChatML now uses correct `<|im_start|>` tokens
- ✅ Llama-3 uses proper header format (not ChatML)
- ✅ Added QwQ-32B, Mistral, DeepSeek support
- ✅ QwQ auto-strips thinking blocks from history
- ✅ 100% accuracy based on official model docs
