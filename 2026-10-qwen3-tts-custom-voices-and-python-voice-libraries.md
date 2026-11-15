tags:: [[tts]], [[voice-cloning]], [[python]], [[ai-model:Opus5.5]], [[ai-effort:high]]
file-created-at:: 2026-11-14

- # Qwen3-TTS custom voices, and the Python voice-library landscape
- **Sources**:
	- Armin Ronacher, *Custom Voices With Qwen3-TTS*, [Armin Ronacher's Thoughts and Writings](https://lucumr.pocoo.org/), October 29, 2026. User-supplied PDF, pp. 1–6.
	- [QwenLM/Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS) README (official model and `qwen-tts` API reference). Retrieved 2026-11-14.
	- Other libraries: 2026 comparison guides ([Codesota](https://codesota.com/guides/tts-models), [TTS.ai](https://tts.ai/blog/open-source-text-to-speech-guide-2026/)) plus general knowledge. Check licenses and install names against each project's repo before relying on them.
- **One-liner**: Qwen3-TTS represents a voice as a **2048-number speaker embedding (x-vector)**. Because it is just a vector, you can blend voices, move a voice along a gender direction, do rough speaker detection, and (with a small hack) give a *cloned* voice emotional instructions, which the official API does not support.

- ## Qwen3-TTS model family
	- Released by Alibaba's Qwen team on **January 22, 2026**, under the **Apache-2.0** license. All models use a **12 Hz speech tokenizer**: audio becomes discrete codes about 12 times per second, and the tokenizer decodes them back into a waveform.
	- | Model | Sizes | What it does | Accepts `instruct`? |
	  |---|---|---|---|
	  | **Base** | 1.7B, 0.6B | Clones a voice from a few seconds of reference audio, either in-context (audio + transcript) or from the speaker embedding alone. Also the starting point for fine-tuning. | ❌ (not documented) |
	  | **CustomVoice** | 1.7B, 0.6B | 9 built-in speakers: Vivian, Serena, Uncle_Fu, Dylan, Eric, Ryan, Aiden, Ono_Anna, Sohee. | ✅ e.g. "speak sad", "whisper" |
	  | **VoiceDesign** | 1.7B | Creates a new voice from a text description ("a man in his forties, warm, medium-low voice…"). | ✅ (the description *is* the instruction) |
	  | **Tokenizer-12Hz** | — | Speech codec: `encode` audio to codes, `decode` codes to audio. | — |
	- **Languages**: Chinese, English, Japanese, Korean, German, French, Russian, Portuguese, Spanish, and Italian, plus some dialect voices.
	- **Streaming**: one "dual-track" model handles both streaming and non-streaming generation. Claimed time to first audio is as low as **~97 ms**.
	- **Deployment**: day-0 support in **vLLM-Omni** (offline inference only at launch), a local Gradio demo (`qwen-tts-demo`), and hosted real-time APIs on Alibaba DashScope.

- ## The `qwen-tts` Python library
	- Install with `pip install qwen-tts` (Ronacher used **0.1.1**). The package is imported as `qwen_tts`. It runs on Apple Silicon with `device_map="mps", dtype=torch.bfloat16, attn_implementation="sdpa"`.
	- | API | Purpose |
	  |---|---|
	  | `Qwen3TTSModel.from_pretrained("Qwen/Qwen3-TTS-12Hz-1.7B-<Variant>", **kw)` | Load a model from a Hugging Face ID or local path. |
	  | `generate_custom_voice(text, language, speaker, instruct=None)` | Speak as a built-in speaker, optionally with a style instruction. |
	  | `generate_voice_design(text, language, instruct)` | Invent a voice from a description and speak the text. |
	  | `create_voice_clone_prompt(ref_audio, ref_text=None, x_vector_only_mode=...)` | Build a reusable clone prompt. `ref_text` is optional when `x_vector_only_mode=True`, at some cost in quality. The prompt holds `ref_spk_embedding`, the x-vector. |
	  | `generate_voice_clone(text, language, ref_audio / voice_clone_prompt, ...)` | Clone a voice. `ref_audio` can be a path, URL, base64 string, or `(np.ndarray, sr)`. |
	  | `get_supported_speakers()`, `get_supported_languages()` | List what the loaded model supports. |
	  | `Qwen3TTSTokenizer.from_pretrained(...).encode/decode` | Use the speech codec directly. |
	- Text, language, speaker, and instruct accept lists for **batching**. Extra kwargs (`max_new_tokens`, `top_p`, `temperature`, …) pass through to Hugging Face `generate`.
	- Generation calls return `(wavs, sample_rate)`. Write the audio with `soundfile.write(path, wav, sr)`.
	- **Fine-tuning** Base on a single speaker is supported (see the repo's `finetuning/` folder).

- ## The gap: cloned voices can't take instructions (officially)
	- You can clone a voice with Base, or direct a voice with CustomVoice/VoiceDesign, but there is **no documented way to do both**. A cloned voice can't be told to "speak sad" or "whisper".
	- Official options for a unique custom voice:
		- Fine-tune Base on a single speaker.
		- Design a clip with VoiceDesign, then clone it with Base. The instructions are lost in this step.
	- Why Ronacher picked Qwen3 over **Kokoro**, his first choice: Kokoro has no voice cloning, and he wanted voices that don't sound like everyone else's AI narrator. The project was a dungeon crawler he is building with his kids.

- ## What you can do with voice embeddings (Ronacher's experiments)
	- ### 1. Fingerprint a voice
		- In `x_vector_only_mode`, Base's **speaker encoder** reduces an entire recording to a single **2048-dim vector**. Similar voices give similar vectors.
		- ```python
		  base = Qwen3TTSModel.from_pretrained("Qwen/Qwen3-TTS-12Hz-1.7B-Base", **kw)
		  def fingerprint(path):
		      prompt = base.create_voice_clone_prompt(ref_audio=path, x_vector_only_mode=True)
		      return prompt[0].ref_spk_embedding
		  ```
		- Reference input: **28 s** clipped from a conference talk (lapel mic), cut between pauses so it starts and ends on full sentences, and transcribed with Whisper. A messy, short recording is enough for a "halfway convincing" clone, which is also the safety concern.
	- ### 2. Blend two voices
		- Take a weighted average of the two embeddings, then rescale it to the interpolated norm so the blend doesn't shrink toward zero:
		- ```python
		  def blend(a, b, t):
		      a, b = a.float(), b.float()
		      mixed = (1 - t) * a + t * b
		      return (mixed / mixed.norm() * ((1 - t) * a.norm() + t * b.norm())).to(torch.bfloat16)
		  ```
		- Example blend: Armin's real voice (median pitch 149 Hz) mixed with a VoiceDesign voice (114 Hz). At 50/50 the result is 130 Hz, and WavLM speaker-verification similarity is **0.82 to Armin** and **0.96 to the designed voice**. The blend stops sounding like the original quickly but keeps some of its traits, and it is always a *unique* voice.
		- Blends work better when the two references have **similar spectrograms**, meaning similar recording setups. The lapel-mic recording sounded hollow next to the clean synthetic one.
	- ### 3. "Bake" a voice into a file
		- Synthesize the transcript with the blended embedding, save it (`final.flac`), and use that file as the new reference. The round trip doesn't reproduce the blend exactly, but you end up with **one portable reference file** instead of a tensor.
	- ### 4. Give a cloned voice instructions (the main hack)
		- Insight: Base and CustomVoice condition on the speaker **the same way**, by inserting one vector into the talker's input sequence.
			- In **CustomVoice**, that vector is a row of the speaker-embedding table.
			- In **Base**, it is the x-vector from the speaker encoder, which only Base has.
			- The transformer weights of the two models are **nearly identical**, and `generate` already accepts `voice_clone_prompt`.
		- Method: extract the x-vector with Base, then pass it to **CustomVoice** in place of a built-in speaker, along with an `instruct`. This uses **private methods** (`_tokenize_texts`, `_build_assistant_text`, `_build_instruct_text`, `_merge_generate_kwargs`), so expect it to break between `qwen-tts` versions.
		- ```python
		  cv = Qwen3TTSModel.from_pretrained("Qwen/Qwen3-TTS-12Hz-1.7B-CustomVoice", **kw)
		  def say(text, xvec, instruct=None, language="English"):
		      ids = cv._tokenize_texts([cv._build_assistant_text(text)])
		      told = [cv._tokenize_texts([cv._build_instruct_text(instruct)])[0] if instruct else None]
		      voice = {"ref_code": [None], "ref_spk_embedding": [xvec],
		               "x_vector_only_mode": [True], "icl_mode": [False]}
		      codes, _ = cv.model.generate(
		          input_ids=ids, instruct_ids=told, voice_clone_prompt=voice,
		          languages=[language], non_streaming_mode=True,
		          **cv._merge_generate_kwargs(temperature=0.7, subtalker_temperature=0.7))
		      wavs, rate = cv.model.speech_tokenizer.decode([{"audio_codes": codes[0]}])
		      return wavs[0], rate
		  ```
		- Result: instructions **do work** on cloned voices, but less reliably than on built-in speakers, and reliability varies by emotion. In one "sad" sample, an acted-speech emotion classifier scored the clip as mostly *happy* (51%) or *angry* (35%), yet speaker similarity stayed at **0.89**. Ronacher found it good enough for game voice-overs.
	- ### 5. Vector math on voice attributes (gender axis)
		- He had VoiceDesign create **14 female and 14 male** voices, fingerprinted them, and took `axis = mean(women) − mean(men)`, normalized.
		- `move(xvec, t)` shifts a voice to position `t` along that axis and leaves its other components alone. This is the same idea as word-vector arithmetic.
		- Unlike simply pitch-shifting the recording (+4, +8, or +12 semitones), this changes perceived gender while keeping speaker identity (similarity to the original ≈0.85 at the start). It was judged with a wav2vec2 gender classifier and WavLM verification.
		- As a held-out check, he also plotted CustomVoice's 9 built-in speakers, which were not used to compute the axis, along the same direction.

- ## Speaker recognition: TTS embeddings vs. verification models
	- Test: a 20-minute podcast episode cut into **2-second windows, one per second**, each compared with a reference embedding of Armin.
	- | Reference clip | Qwen3 x-vector | ECAPA (dedicated speaker-verification model) |
	  |---|---|---|
	  | Conference-talk clip (different mic/setting) | **Fails completely** | Finds him in **97%** of windows |
	  | 20 s self-introduction from the same episode | **98%** correct | — |
	- Lesson: a TTS speaker encoder is tuned for *reproducing* a voice and is sensitive to the recording channel. For cross-recording identification, use a model trained for **speaker verification** (ECAPA or WavLM). On their own, ECAPA and WavLM clustered the episode and agreed on 98% of the speech.

- ## Practical takeaways
	- For narration with unique voices: **VoiceDesign** a base character, **blend** it with a real or other designed voice, **bake** the result to a reference file, then **voice it through CustomVoice** with instructions for emotion.
	- Use clean, matched recording setups for references. About 20–30 s of speech is plenty.
	- Treat the instructed-clone path as an **unofficial hack** tied to `qwen-tts` 0.1.1 internals.
	- Ethics: a short, low-quality clip is enough to fool someone on a phone call. Ronacher: "Use it for good, not evil."

- ## Other Python voice libraries
	- ### Text-to-speech / voice cloning
		- | Library / model | Python entry point | Strengths | License notes |
		  |---|---|---|---|
		  | **Qwen3-TTS** (Alibaba) | `pip install qwen-tts` | Cloning + built-in speakers + voice design, 10 languages, streaming. | Apache-2.0 |
		  | **Kokoro** (82M) | `pip install kokoro` (`KPipeline`) | Tiny, fast, runs on CPU, high naturalness for its size. Ronacher's first choice. | Apache-2.0. Fixed preset voices, **no cloning**. |
		  | **Chatterbox** (Resemble AI) | `pip install chatterbox-tts` | Zero-shot cloning from short audio, an emotion "exaggeration" control, built-in watermarking. | MIT |
		  | **F5-TTS** | `pip install f5-tts` | Flow-matching model with strong zero-shot cloning. | Code MIT, pretrained weights **CC-BY-NC** (no commercial use). |
		  | **CosyVoice 2/3** (Alibaba FunAudioLLM) | install from the GitHub repo | Multilingual, streaming, cloning. | Apache-2.0 |
		  | **Dia** (Nari Labs) | install from the GitHub repo / HF `transformers` | Multi-speaker **dialogue** in one pass (`[S1]`/`[S2]` tags), non-verbal sounds such as laughs. | Apache-2.0. Mainly English. |
		  | **Orpheus** (Canopy Labs) | `orpheus-speech` | Llama-3.2-based, emotive tags, low-latency streaming. | Llama license; requires "Built with Llama" attribution. |
		  | **Sesame CSM-1B** | HF `transformers` | Conversational speech that conditions on dialogue context. | Apache-2.0 |
		  | **Coqui TTS / XTTS-v2** | `pip install coqui-tts` (community fork) | Mature toolkit with many models. XTTS-v2 does multilingual cloning. | Coqui shut down in 2024. XTTS weights use the non-commercial CPML. |
		  | **Piper** | `pip install piper-tts` | Very fast offline TTS on CPU and Raspberry Pi. | MIT engine; per-voice licenses vary. |
	- ### Speech-to-text (used for reference transcripts)
		- **openai-whisper** / **faster-whisper** (CTranslate2, several times faster) / **whisperX** (adds word timestamps and diarization). Ronacher used Whisper both to transcribe the reference clip and to check what the TTS output actually said.
	- ### Speaker verification and diarization
		- **SpeechBrain**: `speechbrain/spkrec-ecapa-voxceleb` gives the ECAPA-TDNN embeddings that found Ronacher in 97% of windows.
		- **WavLM** (`microsoft/wavlm-base-plus-sv` via `transformers`): the speaker-verification "sounds like X" scores in the article.
		- **pyannote.audio**: complete "who spoke when" diarization pipelines. Its HF models are gated.
	- ### Audio classification and I/O
		- **wav2vec2** fine-tunes (via `transformers`) for emotion and gender classifiers like the ones in the article's charts.
		- **soundfile** (read/write WAV/FLAC), **librosa** (pitch, spectrograms, resampling), **torchaudio**.
