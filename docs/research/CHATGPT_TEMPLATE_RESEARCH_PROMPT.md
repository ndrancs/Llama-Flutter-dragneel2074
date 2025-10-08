# ChatGPT Prompt: Research Accurate Chat Templates for LLM Models

Copy and paste this prompt to ChatGPT to get comprehensive, accurate chat template implementations:

---

## PROMPT START

I'm building a Flutter Android plugin for running GGUF models with llama.cpp. I need **production-ready, accurate chat templates** for modern LLM models, including reasoning models.

### Current Implementation Issues

My current chat template system has these problems:
1. **Not comprehensive enough** - Missing many popular models
2. **Not accurate** - May not match official implementations
3. **Missing reasoning models** - No support for o1, QwQ, DeepSeek-R1, etc.
4. **No special token handling** - BOS, EOS, thinking tokens
5. **No validation** - Can't verify templates are correct

### What I Need

Please provide **complete, accurate chat template implementations** for the following model families:

#### Standard Models (Instruction-tuned)
1. **Qwen 2.5 / Qwen 2** (latest format as of October 2024)
2. **Llama 3.2 / Llama 3.1 / Llama 3** (Meta's official format)
3. **Llama 2** (classic format for compatibility)
4. **Mistral / Mixtral** (v0.1, v0.2, v0.3 differences)
5. **Gemma 2** (Google's format)
6. **Phi-3** (Microsoft's latest)
7. **Command-R / Command-R+** (Cohere)
8. **Yi** (01.AI)
9. **DeepSeek-V2** (standard instruction mode)
10. **ChatGLM3 / ChatGLM4** (THUDM)

#### Reasoning Models (WITH Thinking Support)
11. **QwQ-32B-Preview** (Alibaba's reasoning model)
12. **DeepSeek-R1** (reasoning mode with <think> tags)
13. **o1-preview style** (if applicable to open models)
14. **Llama 3.3 reasoning** (if it has special reasoning format)

#### Code Models
15. **CodeLlama** (Meta)
16. **DeepSeek-Coder-V2**
17. **Qwen2.5-Coder**

#### Function Calling Models
18. **Hermes** (NousResearch - function calling format)
19. **Gorilla** (UC Berkeley)

### Requirements for Each Template

For each model family, provide:

#### 1. **Exact Format Specification**
```
<FORMAT_NAME> (e.g., "Qwen ChatML")

System Message Format:
[exact format with special tokens]

User Message Format:
[exact format]

Assistant Message Format:
[exact format]

Multi-turn Format:
[how to chain messages]

Special Tokens:
- BOS (Beginning of Sequence): <token or "none">
- EOS (End of Sequence): <token>
- Thinking Start (for reasoning models): <token>
- Thinking End (for reasoning models): <token>
- Any other special tokens

Example Full Conversation:
[show complete example with 2-3 turns including system prompt]
```

#### 2. **Model Detection Patterns**
How to auto-detect this format from GGUF filename? Examples:
- `qwen2.5-*-instruct` → Qwen format
- `llama-3-*-instruct` → Llama 3 format
- `qwq-*` → QwQ reasoning format

#### 3. **Reasoning Model Specifics**
For reasoning models (QwQ, DeepSeek-R1, etc.), explain:
- How does the model indicate it's "thinking"?
- What special tokens wrap the reasoning process?
- Should thinking be hidden from user or shown?
- How to stop generation at thinking vs. final answer?
- Example of full reasoning output

#### 4. **Edge Cases**
- How to handle empty system prompt?
- How to handle first message not being system?
- What if conversation starts with assistant message?
- Maximum recommended context for this format?

#### 5. **Official Sources**
Provide links to:
- Official model card / documentation
- Official tokenizer config (tokenizer_config.json)
- Official chat template (jinja2 template if available)
- Example implementations (HuggingFace, llama.cpp, etc.)

### Implementation Language

I need this in **Kotlin** (for Android). Please provide:

1. **Kotlin sealed class structure**:
```kotlin
sealed class ChatTemplate {
    abstract fun formatMessages(messages: List<Message>): String
    abstract fun getSupportedPatterns(): List<Regex>
    
    data class Message(val role: String, val content: String)
}
```

2. **Complete implementation for each template** including:
   - Proper escaping
   - Special token handling
   - Multi-turn conversation support
   - System prompt integration

3. **Template detection logic**:
```kotlin
object ChatTemplateDetector {
    fun detectFromFilename(filename: String): ChatTemplate?
    fun detectFromMetadata(metadata: Map<String, String>): ChatTemplate?
}
```

### Validation & Testing

For each template, provide:

1. **Test cases** - Input messages and expected formatted output
2. **Common mistakes** - What NOT to do
3. **Compatibility notes** - Which model versions use this format

### Priority Order

Please prioritize in this order:
1. **Reasoning models** (QwQ, DeepSeek-R1) - MOST IMPORTANT
2. **Latest popular models** (Qwen 2.5, Llama 3.2, Mistral)
3. **Code models** (DeepSeek-Coder, Qwen2.5-Coder)
4. **Everything else**

### Additional Context

- My plugin supports ARM64 Android with llama.cpp (October 2025 version)
- I need templates that work with GGUF quantized models
- Performance matters - templates should be efficient
- I need to support streaming token generation
- Users should be able to cancel generation mid-process

### Expected Output Format

Please structure your response like this:

```markdown
# LLM Chat Templates - Production Ready

## 1. Qwen 2.5 / Qwen 2
### Format Specification
[details]
### Kotlin Implementation
[code]
### Detection Patterns
[patterns]
### Test Cases
[tests]

## 2. QwQ-32B-Preview (REASONING MODEL)
### Format Specification
[details with thinking tokens]
### Kotlin Implementation
[code with thinking support]
### Reasoning Features
[how to handle thinking]
### Detection Patterns
[patterns]
### Test Cases
[tests]

[... continue for all models ...]
```

### Questions I Have

1. Do reasoning models like QwQ use standard ChatML with special thinking tokens, or completely different format?
2. Should thinking tokens be stripped from output or shown to user?
3. Are there any models that use tool/function calling in their chat template?
4. What's the difference between Mistral v0.1, v0.2, and v0.3 templates?
5. Do any models require special handling for empty messages or null system prompts?

Please be as detailed and accurate as possible. I want production-ready code that handles edge cases correctly. Include official documentation links for verification.

Thank you!

## PROMPT END

---

## How to Use This Prompt

1. **Copy the entire prompt** from "PROMPT START" to "PROMPT END"
2. **Paste into ChatGPT** (GPT-4 recommended for accuracy)
3. **Review the response** carefully
4. **Verify with official sources** - Check the provided links
5. **Test with real models** - Download GGUF models and test

## Follow-up Questions to Ask ChatGPT

After getting the initial response, ask:

1. "Can you provide the exact Jinja2 templates from official tokenizer_config.json for Qwen 2.5 and Llama 3.2?"
2. "Show me how DeepSeek-R1's reasoning format differs from QwQ's format with specific examples"
3. "What are the exact regex patterns to detect reasoning models vs standard chat models from filename?"
4. "Provide unit test cases for each template with expected input/output pairs"
5. "How do I handle models that DON'T have a chat template (base models)?"

## Alternative: Use Claude Instead

If ChatGPT doesn't provide accurate enough information, try Claude with this modified prompt:

```
I need you to research and provide EXACT, VERIFIED chat templates for modern LLM models.
Please access official documentation and provide:
1. Official Jinja2 templates from HuggingFace
2. Exact special tokens with their token IDs
3. Links to official model cards
4. Working Kotlin implementations
5. Test cases with expected outputs

Focus on accuracy over comprehensiveness. If you're not sure about a format, say so.
Priority: QwQ-32B, DeepSeek-R1, Qwen 2.5, Llama 3.2
```

## What to Do After Getting Response

1. **Create new file**: `android/src/main/kotlin/.../ChatTemplatesV2.kt`
2. **Implement the templates** based on ChatGPT's response
3. **Add unit tests** to verify correctness
4. **Test with real models** - Download and test each format
5. **Document** any deviations from official templates
6. **Update detection logic** to handle all model variants

## Resources to Verify Templates

After getting ChatGPT's response, verify against:

1. **HuggingFace Model Cards**: https://huggingface.co/models
2. **Official Model Repos**: Check each model's GitHub
3. **llama.cpp examples**: https://github.com/ggerganov/llama.cpp/tree/master/examples
4. **Tokenizer configs**: Look for `tokenizer_config.json` in model repos
5. **Community implementations**: Reddit r/LocalLLaMA, GitHub issues

## Expected Outcome

You should get:
- ✅ 15-20 accurate chat template implementations
- ✅ Reasoning model support (QwQ, DeepSeek-R1)
- ✅ Special token handling
- ✅ Auto-detection logic
- ✅ Test cases for validation
- ✅ Official documentation links

Good luck! 🚀
