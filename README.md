# OptimizedChatBotAI

This project demonstrates how to create chat bot in Unreal Engine with LLM (locally executed or OpenAI) and ElevenLabs text-to-speech service. The goal is simple, but there are some nuances.

The project doesn't work as is. It requires a few plugins:
- Simple HTTP Client (free, download [here](https://github.com/AntiAnti/SimpleHttpClient-UE5/releases))
- Ynnk Voice Lipsync (buy on [Fab](https://www.fab.com/listings/c0c3315b-0d1d-4351-b091-299c437565ba))
- Eleven Voice (buy on [Fab](https://www.fab.com/listings/ecd17f81-8a90-4f69-bf3a-9b7cbd5a564c))
- Ynnk Facial Animation (free, download [here](https://vrmocapstudio.com/plugins.html))

For Eleven Labs, paste your API key to Edit --> Project Settings --> Eleven Voice --> API Key.

To optimize latency, LLM should work in streaming mode, i. e. it returns generated text in chunks. For example:
- How
- How old
- How do are
- How do you you?
  
ElevenLabs supports incomplete requests via websocket to speed up processing and decrease latency. But it returns audio in chunks, and any chunk can end inside of a word. It's not a problem in a common case with USoundWaveProcedural, but it's not suitable in this case since we also want to generate lip-sync and facial animation synchronized with the audio. If we play incomplete voice chunks as is, we'd get noticeable seams between them. So, I don't use incomplete audio chunks. I still send all incoming text tokens to ElevenLabs as I get them, but at the end of each sentence I request to complete generation and return finalized audio to speak it. In other words, we get audios corresponding to sentences. For every finalized audio we generate facial animation and play it at MetaHuman character model.

Default setup in the project uses language model executed locally via [LM Studio](https://lmstudio.ai/). You may try very-very lightweight model **llama-3.2-1b-instruct** or better model like **Meta-Llama-3.1-8B-Instruct-Q4_K_M**, depending on your PC. There is unused event **SubmitNewPrompt_2** in BP_PlayerController blueprint, which demonstrates how to use OpenAI service instead of locally executed model. Either way, text deltas should go to **ProcessGeneratedTextDelta** event. Note: deltas are differences between streamed texts, current and previous. For example, delta between "How old are you?" and "How old are " is "you?".
