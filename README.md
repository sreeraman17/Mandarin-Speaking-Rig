# Speaking Rig

A local conversation partner for spoken Mandarin. You hold the space bar, say
something in imperfect Chinese, let go, and a person answers you. Everything
runs on your own machine. No account, no cloud, no metering, no network once a
session has started.

It exists because talking needs another person, and another person gets bored
long before a learner stops needing repetitions.

```
The other person speaks
        ↓ waits
You hold SPACE and talk
        ↓ release
It works out what you said, and shows it in pinyin and English
        ↓
They answer that
        ↓ waits
You talk again
```

That loop is the whole product.

---

## What it is not

- **Not a tutor.** It never corrects you, grades you, explains grammar, or
  praises your Chinese. The other person in the conversation is a person.
- **Not a drill app.** No streaks, no scores, no spaced repetition, no lessons.
- **Not a tone trainer.** Speech recognisers reconstruct tone from context
  rather than hearing it, so anything reported would be invented. See
  [Limitations](#limitations).
- **Not a reading app.** The interface is pinyin and English only. Chinese
  characters exist inside the machine and never appear on screen.

If you want to be corrected, this will frustrate you. It is built for the
twenty minutes a day where being slow and wrong costs nothing.

---

## Requirements

### System

| | |
|---|---|
| **macOS** | Required. Speech recognition uses `mlx-whisper` (Apple Silicon) and speech output uses the built-in `say` command. |
| **Apple Silicon** | M1 or later. It runs on Intel Macs and slowly. |
| **Memory** | 16GB works. 24GB or more is comfortable, because a speech model and a language model are both resident during a session. |
| **Disk** | About 10GB. Two Whisper models are roughly 4.5GB together, and a language model is 5GB or so. |
| **Homebrew** | For `ffmpeg`. |
| **ffmpeg** | Decodes and measures your recordings. Installed by `setup.sh`. |
| **Python** | 3.10 or later. `setup.sh` builds a virtual environment. |
| **Ollama** | From [ollama.com](https://ollama.com). This runs the language model that plays the other person. |
| **A language model** | `setup.sh` pulls `qwen3:8b`. A 14B model is noticeably better at holding a conversation, if you have the memory. See [Choosing a model](#choosing-a-model). |
| **Chinese voice** | System Settings → Accessibility → Spoken Content → System Voice → Manage Voices → Chinese (China) → Tingting. Without it the app runs and you hear nothing. |
| **A microphone in a quiet room** | This matters more than it sounds. A recording that is 40% low-frequency room noise fails where the same voice at the same volume in a quiet room succeeds. |
| **A browser** | Chrome, Safari or Edge. It runs at `http://127.0.0.1:8765`. |

Python packages, installed by `setup.sh`: `fastapi`, `uvicorn`,
`python-multipart`, `httpx`, `pypinyin`, `jieba`, `mlx-whisper`.

### Human

These are real requirements, not encouragement.

| | |
|---|---|
| **You can read pinyin with tone marks** | `Nǐ qù nǎr?` has to mean something to you on sight. The entire interface is pinyin. If you cannot read it, there is nothing here you can use. |
| **Roughly HSK 1 as a floor** | You need enough words to say something back. Below about 150 words there is no conversation to have, and you will spend the session typing. Start with a textbook, come back at HSK 1. |
| **HSK 4 is roughly the ceiling** | Above that the reply length becomes limiting and the speech recogniser becomes the bottleneck rather than your Mandarin. |
| **You will type sometimes** | When the recogniser cannot work out what you said, typing pinyin is the way through. This is designed in, not a failure mode. If having to type occasionally will annoy you out of using it, this is not for you. |
| **You can tolerate being misheard** | Whisper is English-first and its Mandarin is weaker. On short utterances from a non-native speaker it gets things wrong. Sometimes several times in a row. See [Limitations](#limitations). |
| **You want to talk, not to be taught** | Nothing here corrects you. If you say something wrong and it is understood, the conversation continues with your mistake in it. |
| **Twenty to thirty minutes** | That is the shape of a session. It is not built for five-minute bursts or two-hour marathons. |
| **A separate teacher, ideally** | This practises production. It does not teach grammar, fix your tones, or tell you when you are wrong. Those still need a person or a book. |

Not required, and worth having: another language with tones or without
articles. Learners coming from Tamil, Kannada, Vietnamese or Thai often find
structural bridges that English does not offer. Also worth knowing: if
aspiration works differently in your first language, the recogniser will
mishear the same pairs your ear finds hard, which is annoying and informative.

---

## Install

```bash
git clone https://github.com/YOUR-NAME/speaking-rig.git
cd speaking-rig
./setup.sh
```

`setup.sh` checks the machine, installs ffmpeg and the Python packages, starts
Ollama and pulls a model, checks for the Tingting voice, and downloads both
speech recognisers. Most of it is downloading. On a slow line the model
downloads take twenty minutes or more.

It stops with an explanation if anything essential fails. It does not warn and
carry on, because a setup that passes with no model on disk turns into a
fourteen-minute stall on your first turn.

Then:

```bash
./start.sh
```

That opens `http://127.0.0.1:8765`. Leave the terminal window open. Control-C
when you are done.

---

## Using it

**Start screen.** Pick or write a situation, choose a level, choose a model,
set the reply length. There is a health list showing what is working. The Start
button is never disabled by a health check, because a check that is wrong
should not lock you out of your own app. It is gated only on required model
files existing on disk, which is a file check and cannot be wrong in the same
way.

**A turn.** Hold SPACE, or hold the microphone button, and talk. Release. The
microphone is open only while you hold it. Escape throws a recording away.

**When it understands you**, your turn appears in pinyin and English and the
other person answers.

**When it is unsure**, your turn appears with two or three alternatives as
chips. Tap one and the conversation carries on from there.

**When it cannot work out what you said**, the turn is refused with the reason,
the audio measurements, and a typing box already open. Type pinyin. Tone marks,
tone numbers or nothing at all: `wo shi yindu ren`, `wo3 shi4 yin4du4 ren2` and
`wǒ shì yìndù rén` all work.

**Every turn of the other person's** has three controls. Replay. Slower. And
**take it somewhere else**, where you type an instruction in plain English:
`ask me about my work`, `talk about food`, `stop asking about the Great Wall`.
That replaces the turn. You steer.

**What it understood** appears on the other person's turns. It shows the one
line of English the model wrote about your turn before it replied. When a reply
lands flat, this tells you whether it misread you or read you and had nothing
to say.

---

## Features

- Push to talk, on the space bar or a button, with pointer capture so drifting
  off the button does not cut you off mid-sentence.
- Three outcomes per turn: understood, ambiguous with chips, or refused with a
  typing box. Never a dead end.
- Typed pinyin as a first-class input, in any tone notation, read into
  characters by a model whose answer is checked against your syllables.
- A conversation partner with a situation and no script.
- A record of what has been established, carried across turns and visible.
- Steer the conversation in plain English at any point.
- Reply length adjustable mid-session.
- Speech output through the system Chinese voice, with a slower replay.
- Playback of your own recording on every turn, which is the only unmediated
  record of what you said.
- Failures shown in full: the transcription, the recogniser score, the audio
  measurements, and what the machine considered.
- A per-turn diagnostic log you can read afterwards.
- A transcript export, with characters as an opt-in, so a teacher can read it.
- Everything downloaded before a session starts, and nothing downloaded during
  one.
- A one-button comparison of every language model you have installed, scored on
  real conversation turns.
- 374 regression checks, none of which need a microphone, a model or a network.

---

## Use cases

**The main one.** Twenty minutes a day of low-stakes speaking practice between
lessons, for a learner whose reading is ahead of their talking. This is what it
was built for and the only one it has been used for at length.

**Rehearsing a specific situation.** Write the situation yourself: `a landlord
asking why the rent is late`, `a nurse taking my blood pressure`. It is one
line of free text and there is nothing scene-specific in the code.

**Warming up before a lesson**, so the first ten minutes of a class are not
spent remembering how to form a sentence.

**Producing a transcript** of what you actually managed to say, for a teacher
to look at.

**Testing local models on a real task.** The comparison tool scores replies
from every model you have on turns taken from sessions that went wrong. It is a
reasonable small benchmark for conversational instruction-following in Chinese
even if you do not care about learning Mandarin.

**Not a use case.** Anything where being misunderstood matters. It is a
practice room.

---

## Architecture

Full detail, including the low-level design, is in
[ARCHITECTURE.md](ARCHITECTURE.md). The shape:

```
  your voice
      │
      ▼
  browser records ──────────► ffmpeg: 16kHz mono, silence trimmed,
                              then measured for duration, loudness,
                              peak and how much of it is speech
      │
      ▼
  Whisper (mlx) ────────────► what it thinks you said, plus a confidence
      │                       Rejected here: English inside Chinese,
      │                       repetition loops, ghost subtitles
      ▼
  candidate pool ───────────► readings of the same syllables, from a
      │                       dictionary. Nothing from the conversation.
      ▼
  resolver ─────────────────► an LLM picks an index. It has no field
      │                       it could write a sentence in.
      ▼
  ┌───────────────┬────────────────────┬───────────────────┐
  │ clear         │ ambiguous          │ nothing fits      │
  │ accept        │ chips, you pick    │ you type pinyin   │
  └───────────────┴────────────────────┴───────────────────┘
      │
      ▼
  ACCEPTED TURN ◄──── the only thing that crosses into the conversation
      │
      ▼
  conversation model ──────► reads your turn, then answers it
      │                      Checked for: repeating itself, echoing you,
      │                      two questions at once. One retry.
      ▼
  pinyin + English + speech
```

### The three things enforced in code

Everything else is the model's job. These are not.

1. **One reply per accepted turn.** Every turn takes a number and a reply for a
   superseded turn is discarded. Picking a chip while the first reply was in
   flight used to render both.
2. **Rejected recognition never enters the conversation.** What the recogniser
   produced is evidence. It becomes true in exactly three ways: it was credible
   on its own, you tapped a chip, or you typed it.
3. **Nothing downloads during a session.** Every model is fetched by
   `setup.sh` and checked on disk before a turn is transcribed.

### What is deliberately not in code

No list of what any scene means. No list of what a taxi driver knows or where a
conversation should go. No place names, no polarity word lists, no rules about
what a question should be about. All of those existed at one point, each one
added to fix something real, and together they turned the other person into a
machine that echoed your words and asked a question every single turn.

Code enforces turn-taking and truth boundaries. The model understands and
conducts the conversation. The reasoning, and the four builds it took to learn
it, are in [DECISIONS.md](DECISIONS.md).

---

## Limitations

Read this section before deciding whether to use it.

**Speech recognition is the weakest part.** Whisper is English-first and its
Mandarin corpus is much smaller. On short utterances from a non-native speaker
it fails often. Real examples from real sessions: `我不知道` came back as
`我不会爱你`; `慢一点儿吗` came back as the word "Boss" sixteen times; `鸡` was
heard as `纸` twice in a row. Four attempts at one sentence happens. The way
through is typing, which always works.

A Mandarin-first recogniser (SenseVoice or Paraformer via FunASR) is the known
next piece of work and is not done.

**Reply quality depends heavily on the model.** On an 8B model, replies are
grammatical, on topic, and often hollow: they acknowledge what you said without
engaging with it. A 14B model is meaningfully better. The comparison tool exists
because this is the single biggest variable and arguing about it is pointless.

**No tone feedback, on purpose.** Recognisers infer tone from context rather
than hearing it. Anything reported here would be invented. Real tone practice
needs pitch tracking against a reference contour, which is a separate project
of comparable size.

**The pinyin dictionary picks one word per sound.** `shi` is always `是`, so
`shi kuai qian` reads as `是快前` rather than `十块钱`. A model reads typed
pinyin in context and fixes most of this, but when the model is unavailable the
dictionary answers and it is wrong in this way. Documented cases are printed on
every test run.

**macOS only.** `mlx-whisper` needs Apple Silicon and the speech output uses the
`say` command. A Linux port needs a different recogniser and a different voice.

**Latency.** A clear turn is fast. An unclear one costs a resolver call, and a
reply that gets sent back for repeating itself costs another. Observed replies
have ranged from two to eleven seconds. Everything is warmed before the session
so no turn pays a cold start.

**No Chinese characters on screen.** This is a design decision, not an
oversight. If you can read characters and want them, the transcript export has
an opt-in and the interface does not.

**Tested by one learner.** One person, one accent, one level, over some weeks.
Everything in the regression suite came from something that actually broke, and
the failures of learners who are not that person have not been seen yet.

**Not accessible.** No screen reader support, no keyboard alternative to the
push-to-talk gesture beyond the space bar, no captions on the audio beyond the
pinyin and English already on screen.

---

## Configuration

Situations are a list of one-line strings in `seeds.json`. Add your own.

Levels and reply lengths are in `levels.json`. The reply length slider on the
start screen overrides it per session.

Words the recogniser is nudged towards are in `hotwords.json`. Add anything it
keeps mishearing, especially names and places. Keep it under about sixty
entries; a longer list dilutes the effect rather than strengthening it.

Everything else is an environment variable, and none of them need changing:

| variable | default | what it does |
|---|---|---|
| `WHISPER_REPO` | `mlx-community/whisper-large-v2-mlx` | the recogniser |
| `ASR_FALLBACK` | `mlx-community/whisper-medium-mlx` | tried when the first one loops |
| `OLLAMA_HOST` | `http://127.0.0.1:11434` | where Ollama is |
| `TTS_VOICE` | `Tingting` | the system voice |
| `ACCEPT_AT` | `0.55` | above this, a turn is accepted silently |
| `CHECK_AT` | `0.30` | below this, a turn is not accepted |
| `RESOLVE_ABOVE` | `0.62` | short turns above this skip the resolver |
| `RESOLVE_ABOVE_LONG` | `0.46` | longer turns above this skip the resolver |
| `LONG_ENOUGH` | `5` | characters, above which a turn counts as long |
| `PRIMER_WORDS` | `12` | words offered to the recogniser. 0 turns it off |
| `PRIMER_CHARS` | `150` | a hard ceiling on the primer's length |
| `ALT_MARGIN` | `2.5` | how far below the best reading an alternative may score |
| `CONFIRM_UNCERTAIN` | `0` | 1 makes an uncertain turn wait for you |

### Choosing a model

Press **Compare your models** on the start screen. It runs every model you have
installed over six turns taken from sessions that went wrong, and prints the
replies with what each turn is testing. It scores repeats, echoes, doubled
questions and length, and makes no claim about whether a reply was any good.
Read them and judge that yourself.

Two to three minutes per model.

---

## Tools

| | |
|---|---|
| `python tests/regression.py` | 374 checks. No microphone, no model, no network. Run before and after any change. |
| **Compare your models** button | every installed language model, on real turns |
| `python tests/bench_asr.py` | every recogniser on disk, over your own recordings in `tests/fixtures/` |
| `python tests/probe_audio.py <file.wav>` | one recording: where its energy sits, and what each recogniser makes of it before and after a high-pass filter |
| `python tests/label_session.py` | walks a finished session and asks two questions per turn, then prints how many turns moved on without you fighting the machine |

That last number is the only real measure of whether this works. A passing test
count says the parts work. It does not say the conversation did.

---

## Privacy

Everything is local. There is no account, no telemetry, and no network traffic
during a session.

Your recordings are written to a temporary folder so you can play them back and
so failures can be diagnosed. `End session` offers to delete them, and there is
a wipe that removes every recording of your voice. Synthesised speech is cached
separately and is not a recording of you.

The per-turn diagnostic log contains your transcriptions. It is a file on your
machine and nothing sends it anywhere.

---

## Contributing

The most useful contribution is a real failure. If a turn goes wrong:

1. Find the recording. The failure block on screen names the turn id, and the
   files are in the work folder.
2. Copy it into `tests/fixtures/` and add an entry to `fixtures.json` with what
   you were actually trying to say.
3. Open an issue with the audio measurements from the screen and the
   transcription.

Real failures beat synthetic audio and beat clean native-speaker samples.

If you are changing code: run `python tests/regression.py` before and after,
and do not reduce the pass count. Read [DECISIONS.md](DECISIONS.md) before
proposing anything structural. Most entries in it exist because the opposite was
tried first and failed in a way that took hours to find.

---

## Where this came from

It was built over some weeks for one learner, in conversation with an
assistant, by shipping a version, using it, and reporting exactly what went
wrong. Roughly seventy builds. The interesting record is not the code, it is
[DECISIONS.md](DECISIONS.md), which says why each part is shaped the way it is
and what was tried before it.

Several of the entries are about things that were built, measured, and thrown
away. Those are the useful ones.

---

## License

MIT. See [LICENSE](LICENSE).
