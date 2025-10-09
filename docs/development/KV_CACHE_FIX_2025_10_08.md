# KV Cache Position Tracking Fix - October 8, 2025

## Problem Summary

The app was crashing on the **second message** in a multi-turn conversation with the error:
```
java.lang.RuntimeException: Failed to decode prompt
```

### Root Cause

The native JNI code in `jni_wrapper.cpp` was resetting token positions to 0 for every generation, causing conflicts in the KV (Key-Value) cache:

1. **First message**: Tokens at positions 0-25 were decoded into KV cache ✅
2. **Second message**: Tokens at positions 0-48 tried to overwrite positions 0-25 ❌
3. **Result**: KV cache corruption → "Failed to decode prompt"

The issue was on line 283-289:
```cpp
batch.pos[i] = i;  // ❌ Always starts from 0!
```

## Solution

Implemented **persistent KV cache position tracking** across multiple generations by:

### 1. Added Global Position Tracker
```cpp
static int g_n_past = 0;  // Track the number of tokens already in KV cache
```

### 2. Updated Batch Position Assignment
```cpp
// OLD (wrong):
batch.pos[i] = i;

// NEW (correct):
batch.pos[i] = g_n_past + i;  // Continue from where previous generation left off
```

### 3. Increment Position Counter After Decoding
```cpp
// After decoding prompt:
g_n_past += tokens.size();

// After each generated token:
g_n_past++;
```

### 4. Reset Position Counter When Needed
```cpp
// When loading new model:
g_n_past = 0;

// When freeing model:
g_n_past = 0;
```

## Changes Made

### File: `android/src/main/cpp/jni_wrapper.cpp`

1. **Line 19**: Added `static int g_n_past = 0;`
2. **Line 209**: Reset `g_n_past = 0;` after loading model
3. **Line 285**: Changed `batch.pos[i] = g_n_past + i;`
4. **Line 294**: Added log: `LOGI("Decoding batch starting at position %d...")`
5. **Line 302**: Increment `g_n_past += tokens.size();` after prompt decode
6. **Line 419**: Changed `batch.pos[batch.n_tokens] = g_n_past;`
7. **Line 427**: Increment `g_n_past++;` after each token
8. **Line 463**: Reset `g_n_past = 0;` when freeing model

## How It Works Now

### Multi-Turn Conversation Example:

**Turn 1:**
- Prompt: "Hi" (26 tokens after template)
- Positions: 0-25 in KV cache
- Generated: "Hello! How can I assist you today?" (9 tokens)
- g_n_past after: 35

**Turn 2:**
- Prompt: "This is second message" (49 tokens after template)
- Positions: 35-83 in KV cache ✅ (continues from 35, no conflict!)
- Generated: Response tokens at positions 84+
- g_n_past after: 84+

**Turn 3+:**
- Continues seamlessly...

## Benefits

1. ✅ **Preserves conversation context** - Full chat history in KV cache
2. ✅ **Eliminates crashes** - No position conflicts
3. ✅ **Memory efficient** - Reuses KV cache instead of rebuilding
4. ✅ **Faster responses** - Doesn't re-process entire conversation each time

## Testing

After rebuilding the native library with these changes:

```bash
cd android
./gradlew assembleDebug
```

The app should now:
1. Successfully handle the first message ✅
2. Successfully handle the second message ✅
3. Continue multi-turn conversations indefinitely ✅

## Context Limits

Be aware that the model was loaded with `contextSize: 1024` tokens. Long conversations will eventually hit this limit. The plugin should handle this gracefully by:
- Monitoring `g_n_past` value
- Implementing context trimming when approaching limit
- Or resetting conversation when limit is reached

## Next Steps

Consider implementing:
1. **Context window monitoring** - Warn when approaching limit
2. **Smart context trimming** - Keep recent messages, summarize old ones
3. **Conversation reset API** - Allow manual KV cache clearing
4. **Context sliding window** - Automatically trim old messages

## Related Files

- `android/src/main/cpp/jni_wrapper.cpp` - Native JNI implementation
- `lib/src/llama_controller.dart` - Dart API wrapper
- `logs/chatapp_error.txt` - Error logs that revealed the issue

## References

- llama.cpp KV cache documentation
- Batch processing in transformer models
- Position encoding in LLMs
