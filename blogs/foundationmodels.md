# GPT-1:

# BERT:

# GPT-2:

# T5:

# GPT-3:

# Claude:

# LLaMA Family:

## Llama 2 vs Llama 3 Model Progression

**Context Window** :
- Llama 2.0 starts at 4K tokens, while all Llama 3.x versions move up to at least 8K (3.0) and then 128K (3.1, 3.2), showing a major leap in handling longer contexts.

**Vocabulary Size** :
  - Llama2.0 uses a 32K token vocabulary.
  - All later versions (3.x) expand this to a much larger 128K, enabling more efficient and diverse tokenization.

**Multilingual Capabilities** :
  - Llama 2.0 and 3.0 are "English Only."
  - With Llama 3.1 onwards, the models are "Official Multilingual" in 8 languages, showing a significant expansion in language support.
**Tool Calling** :
  - Neither Llama 2.0 nor Llama 3.0 support tool calling.
  - With Llama 3.1, 3.2 Multimodal, and 3.2 Lightweight, tool calling is supported.

**Llama 3.1** models come in both base and instruct (finetuned) versions, with model sizes of 8B, 70B, and 405B parameters. All models offer multilingual support (8 languages) and were released in July 2024.Only the Instruct models are finetuned and support tool use, making them better suited for instruction following and external tool integration. None of the Llama 3.1 models are multimodal (they handle text only).

**Llama 3.2** models (released September 2024) include both Base and Instruct versions, with model sizes ranging from 1B to 90B parameters. All models support multilingual capabilities across 8 languages. Only the Instruct models are finetuned and support tool use, enabling more sophisticated interactions. The MM (multimodal) models—both Base and Instruct (11B and 90B)—can process both images and text as input, while the smaller 1B and 3B Instruct models handle text only. This release brings advanced
multimodal abilities and retains strong multilingual and tool integration features.

### Prompt Format:

Llama supports roles to indentify the information in prompts.Llama 3.1 & 3.2 have 4 roles:

- system
- user
- ipython
- assistant

**Special tokens for single-turn and multi-turn chat**:

1. **<|begin_of_text|>** : Start of a prompt

2. **<|start_header_id|>** : Start of a role for perticular message.

3. **<|end_header_id|>**: End of the role for a perticular message.

4. **<|eot_id|>** : End of a turn.

5. **<|eom_id|>** : End od the message. A message represents a possible stopping point where the model can inform the execution environment that a tool call need to be made.

6. **<|python_tag|>** : A special tah used in model's response to signify a tool call.

7. **<|finetune_right_pad_id|>**: Used for padding text sequences in a batch of the same length.

8. **<|end_of_token|>**: Model will cease to generate more tokens after this. This token is generated only by the base model.

The Llama3 family uses the open source tiktoken3 tokenizer. The family has a vocablary of 128K tokens.This is 4X more than llama2. 

# Bloomberg GPT:

# GPT-4:

# Claude-2:

# Mistral 7B :

# Grok-1:

# Gemini 1.0 & 1.5:

# Phi-2 & Phi-3:

# Claude-3:
