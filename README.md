<div align="center">

<br>

# Speaking Rig

### A sparring partner for spoken Mandarin / Spoken Chinese

**No scores. No streaks. No corrections.**
**Somebody to talk to.**

<br>

`Runs on your Mac` · `Works with the wifi off` · `No account` · `Free forever!`

<br>

**[Say your first five lines →](CUES.md)**

<br>

</div>

---

<br>

## You can learn to read alone. You cannot learn to talk alone.

Every other part of learning a language, you can do by yourself. Flashcards
work. Grammar books work. Podcasts work. You can put in a year on your own and
come out able to read a menu.

Then you sit in a taxi and nothing comes out.

Talking needs another person. And another person, however kind, gets bored long
before you stop needing repetitions. Your teacher has forty minutes a week. Your
patient friend has about three conversations in them. Nobody has the twenty
minutes a day, every day, where you say the same clumsy sentence six times until
your mouth learns it.

That is the gap. This fills it.

<br>

## What it feels like

You write down where you are. One line, anything you like. This one is a waiter
at a noodle place who has decided you cannot handle spice.

Then you hold the space bar and talk.

```
   WAITER    Nínhǎo, xiǎng chī diǎn shénme?
             Hello, what would you like to eat?

   YOU       Wǒ yào yìwǎn jīdàn miàn
             I want a bowl of egg noodles.

   WAITER    Hǎo de, yào jiā là ma?
             Okay, would you like it spicy?

   YOU       Wǒ bù néng chī tài là de
             I cannot eat very spicy food.

   WAITER    Zhēn de? Nà wǒ gěi nín shǎo fàng yìdiǎnr.
             Really? Then I will go easy on it for you.
```

He does not tell you your tones were off. He does not award you a gem. He is a
waiter who thinks you are soft, and he is mildly disappointed in you, and the
conversation carries on.

That is the whole product.

<br>

## Three things you need. One person.

To practise speaking, you need three things.

**Somebody patient enough to hear you fail.** Not politely patient. Genuinely
indifferent to how long you take, because there is nowhere else they need to be.

**Somebody who talks back like a person.** Who has an opinion about the traffic.
Who remembers you said you were in a hurry. Who asks you something you actually
want to answer.

**Somewhere it costs nothing to be wrong.** No teacher's face. No stranger's
patience running out. No score at the end.

These are not three features. It is one person, and they are sitting on your
laptop, and they have all the time in the world.

Built to solve the longing for a speaking buddy to practise Mandarin, by yours truly — [sreeraman.in](https://www.sreeraman.in)

<br>

## What the apps built instead

Every language app on your phone is optimised for one thing, and it is not your
Mandarin. It is your return visit tomorrow.

So they correct you, because a correction is a moment of feedback and feedback
is engagement. They score you, because a number can go up. They give you a
streak, because loss aversion is the cheapest retention mechanic ever
discovered. They gamify a green owl into your notifications.

None of that is talking.

Being corrected is the opposite of practising. A taxi driver in Chengdu will not
stop the car to tell you your third tone collapsed. He will squint, guess what
you meant, and answer the guess. Learning to survive that squint is the actual
skill, and no app will let you have it, because letting you be wrong feels like
a bad product.

Here you get to be wrong. Nobody is watching. Nothing is counting.




<br>

## It will mishear you. That is not the bug.

Sometimes you will say something and the machine will get it wrong. It will
show you what it thought you said, offer a couple of guesses, and if it is
completely lost it will hold up its hands and let you type.

This is the part people expect us to apologise for, and we are not going to.

Being misunderstood is what happens when you speak a language badly. Working
out how to get your meaning across anyway, saying it slower, saying it another
way, pointing at the thing, is not a workaround. It is the skill. A learner who
has never been misunderstood in practice has never practised.

What we did promise is that you can never get stuck. Every turn has a way
forward. Say it again, tap the right guess, or type it. The conversation always
continues.

<br>

## Say something. Right now.

You do not have to know what to say. That is the hardest part of a blank
conversation and we took it away.

**[CUES.md](CUES.md)** is one page. Ten lines, five in a taxi, five in a noodle
shop. Every one is something you would really say. Every one has the English, and
what to type if the machine misses you, and what a good answer looks like so you
know whether it is working.

Start here:

```
Wǒ yào qù jīchǎng            I want to go to the airport
Wǒ shíjiān bù duō le         I do not have much time
Qǐng kāi màn yìdiǎnr         Please drive a bit slower
```

Tell a driver you are in a hurry and then ask him to slow down, and see whether
he notices you just contradicted yourself. A good one does.

<br>

## Get it running

```bash
git clone https://github.com/YOUR-NAME/speaking-rig.git
cd speaking-rig
./setup.sh          # mostly downloading. Go and make tea.
./start.sh
```

That is it. It opens in your browser, on your own machine. Turn the wifi off if
you like; it will not notice.

<br>

## Is this for you?

**Yes, if** you can read pinyin with the tone marks, you know somewhere around
150 Chinese words, and what you want is twenty minutes a day of talking with
nobody marking you.

**Not yet, if** you are starting from zero. You need enough words to say
something back. Go and do HSK 1 in a textbook and come back; we will still be
here.

**No, if** what you want is to be corrected. Nothing here will tell you when you
are wrong. Some people need that and there is no shame in it, but this is not
that tool.

**Also no, if** you do not have a Mac with Apple Silicon. The speech recognition
needs it. Sorry.

<br>

<div align="center">

**[Say your first five lines →](CUES.md)**

</div>

<br>

---
---

<br>

# For developers

<br>

Everything below is for people who want to read the code, run it, or take it
apart. Learners can stop at the line above.

The short version: a turn-taking conversation system in which speech
recognition is one noisy sensor. Code enforces turn-taking and truth
boundaries. A language model understands and conducts the conversation. Nothing
in code decides what a conversation means or where it should go.

<br>

## Battle scars

Four builds and about seventy iterations of a person using it and reporting
exactly what broke. These are the findings that cost the most and would be
easiest to get wrong again.

<details open>
<summary><b>A confidence score means different things at different lengths</b></summary>
<br>

Whisper heard `我的行李不重` at 0.50 confidence, in direct answer to a question
about luggage. Correct sentence, sensible answer, thrown away. Twice.

The fix that finally worked has nothing to do with tuning a threshold. A decoder
that has lost the thread invents **short** things. `Boss`. `我估计到`. `你告诉我`.
It does not invent `我住在孟买你呢` and hold it together for seven characters.

So the acceptance bar is 0.62 at four characters or fewer and 0.46 above that.
Measured against every turn of three real sessions: thirteen of fourteen land
where they should, and the fourteenth costs one extra model call rather than an
error.

Two earlier fixes for the same symptom did not work, and both are documented in
`DECISIONS.md` as failures.

</details>

<details>
<summary><b>A model that can only pick cannot lie</b></summary>
<br>

There is a language model choosing between readings of what you said. Its
entire output is one integer and two booleans. There is nowhere in its schema a
sentence could go.

This constraint was learned the expensive way. An earlier version of the same
layer could return text, and it did: `你好,去啦吗?` came back as
`你好，要去机场了吗？` and the conversation carried on as though that had been
said. Roughly a dozen builds were spent adding guards before the layer was
removed entirely, and it only came back once the output had nowhere to hide a
sentence.

`candidate_pool()` takes what was heard and a count. It has no parameter for the
conversation. That is the mechanism by which context ranks candidates rather
than adding to them.

</details>

<details>
<summary><b>Typed pinyin is a different problem to speech, and safer</b></summary>
<br>

The dictionary keeps one word per sound, so every measure word in the language
loses: `杯` to `北`, `碗` to `玩`, `份` to `分`, `瓶` to `平`, `鸡` to `及`. A
learner ordering a cup of tea got `一北茶`. Widening the index to the runners-up
was tried three times and failed three times, because `鸡` is only the fifteenth
commonest word for the sound `ji` and no beam width reaches it.

A model reads it instead. What makes that safe is that the syllables are not
evidence of anything: the learner typed them. So the answer is read back into
pinyin and rejected unless it spells out to exactly those syllables.
`我想要一杯热茶谢谢` does not spell out to `yi bei cha` and is thrown away.

Three outcomes, all tested: it spells out and is used, it writes something else
and the dictionary answers, or Ollama is down and the dictionary answers. Typing
degrades to being wrong in a documented way rather than failing.

</details>

<details>
<summary><b>Fixing symptoms one at a time built a machine</b></summary>
<br>

This is the most useful scar and the least technical.

A driver asked his passenger which exit to take, so a check went in requiring
every question to contain 你. He invented a city, so a list of thirty place
names went in. He asked the same thing three times, so `seeds.json` grew fields
describing what fifteen different characters knew and where fifteen different
conversations could travel. He ignored a no, so lists of negative and
affirmative characters went in.

Every one of those fixed the thing in front of it. Together they turned the
other person into a machine that echoed the learner's words and then asked a
question with 你 in it, **every single turn**. The learner's phrase for it was
"hard coded logic". The brief had reached 490 words, most of it about the Great
Wall and airport terminals.

All of it was deleted. The brief is 167 words now and contains four rules about
conversation and nothing about any scene. A model that knows what a waiter is
does not need to be told what a waiter knows.

</details>

<details>
<summary><b>There is no cheap test for "is this real Mandarin"</b></summary>
<br>

Two were built and measured against real failures, and neither works.

Segmentation statistics do not separate: `我到了` and `我到楼` have identical
token counts, single-character fractions and longest-word lengths. A lattice
round trip does not separate either: `不是有面在洗手间` round trips perfectly and
`洗手间在哪里` does not, because `哪` and `那` swap.

The judgement needs a language model, so the resolver's prompt now opens by
telling it that most of what it is about to see is strings of real Chinese words
nobody would say.

</details>

<details>
<summary><b>Every check in code compares text against text</b></summary>
<br>

Five survive, and not one makes a claim about meaning.

| check | what it compares |
|---|---|
| `is_noise` | Latin letters inside a Chinese turn, a repeated fragment, a known subtitle artefact |
| `repeats_itself` | a four-character run, or a question's subject words, reused from that speaker's own earlier turn |
| `echoes_learner` | the reply's opening clause against the learner's own words |
| `questions_in` | how questions are marked in Chinese, which is grammar |
| `spells_out` | characters read back as exactly these syllables |

Three things are hard-coded and nothing else is: one reply per accepted turn,
rejected recognition never entering the conversation, and nothing downloading
during a session.

</details>

<details>
<summary><b>Small operational scars, each of which cost an evening</b></summary>
<br>

- Whisper returns `-inf` for a degenerate segment, Python writes that into JSON
  as a bare `NaN`, and the browser rejects the whole response with an error
  explaining nothing. Everything numeric is scrubbed and a middleware turns any
  unhandled exception into JSON.
- A stale server running old code alongside a fresh page cost an entire evening
  of diagnosing bugs that had already been fixed. The page and the server carry
  the same build number and compare them.
- One badly heard turn used to trigger a 1.5GB download of the fallback
  recogniser, mid conversation, with nothing on screen. Nothing downloads during
  a session now, and `setup.sh` fails loudly rather than warning and carrying on.
- A health check that was wrong once locked the learner out of their own app.
  The Start button is gated only on files existing on disk.
- A suspended `AudioContext` reported silence and swallowed every recording. The
  microphone meter informs and never blocks a turn.
- Picking a chip while the first reply was still in flight rendered two turns
  from the other person. Every turn takes a number and a superseded reply is
  discarded.

</details>

<br>

Full design in **[ARCHITECTURE.md](ARCHITECTURE.md)**, high level and low level.
The complete record, including everything built and thrown away, in
**[DECISIONS.md](DECISIONS.md)**. That second file is the interesting one, and
several entries exist because the opposite was tried first.

<br>

## Requirements

### System

| | |
|---|---|
| **macOS, Apple Silicon** | M1 or later. Speech recognition is `mlx-whisper`, speech output is the built-in `say`. Runs on Intel and slowly. |
| **Memory** | 16GB works, 24GB is comfortable. A speech model and a language model are both resident during a session. |
| **Disk** | About 10GB. Two Whisper models are roughly 4.5GB, a language model another 5GB. |
| **Homebrew and ffmpeg** | ffmpeg decodes and measures recordings. `setup.sh` installs it. |
| **Python 3.10+** | `setup.sh` builds a virtual environment. |
| **[Ollama](https://ollama.com)** | Runs the language model that plays the other person. |
| **A language model** | `setup.sh` pulls `qwen3:8b`. A 14B is noticeably better at conversation if you have the memory. |
| **The Tingting voice** | System Settings → Accessibility → Spoken Content → System Voice → Manage Voices → Chinese (China). Without it the app runs and you hear nothing. |
| **A microphone in a quiet room** | This matters more than it sounds. A recording that is 40% low-frequency room noise fails where the same voice at the same volume in a quiet room succeeds. |

Python packages, installed for you: `fastapi`, `uvicorn`, `python-multipart`,
`httpx`, `pypinyin`, `jieba`, `mlx-whisper`.

### Human

| | |
|---|---|
| **Reads pinyin with tone marks** | The whole interface is pinyin. Non-negotiable. |
| **HSK 1 floor** | Around 150 words. Below that there is no conversation to have. |
| **HSK 4 ceiling, roughly** | Above it the recogniser becomes the bottleneck rather than your Mandarin. |
| **Willing to type sometimes** | Designed in, not a failure mode. |
| **Tolerates being misheard** | Whisper is English-first and its Mandarin is weaker. |
| **Wants to talk, not be taught** | Nothing here corrects you. |
| **Twenty to thirty minutes** | The shape of a session. |

Worth having and not required: another language with tones, or without articles.
Learners coming from Tamil, Kannada, Vietnamese or Thai often find structural
bridges English does not offer. And if aspiration works differently in your
first language, the recogniser will mishear the same pairs your ear finds hard.
Annoying, and informative.

<br>

## Limitations

**Speech recognition is the weakest part.** Real examples from real sessions:
`我不知道` came back as `我不会爱你`. `慢一点儿吗` came back as the word "Boss"
sixteen times. `鸡` was heard as `纸` twice running. Four attempts at one sentence
happens. A Mandarin-first recogniser (SenseVoice or Paraformer via FunASR) is the
known next piece of work and is not done.

**Reply quality depends heavily on the model.** On 8B, replies are grammatical,
on topic, and often hollow. 14B is meaningfully better. Press **Compare your
models** on the start screen rather than taking anyone's word for it.

**No tone feedback, on purpose.** Recognisers infer tone from context rather
than hearing it, so anything reported would be invented. Real tone work needs
pitch tracking against a reference contour, which is a separate project of
comparable size.

**The dictionary picks one word per sound.** Mitigated by the spelling layer.
When Ollama is unavailable the dictionary answers and the documented cases print
on every test run.

**macOS only.** A Linux port needs a different recogniser and a different voice.

**Latency.** Two to eleven seconds for a reply, depending on how much repair a
turn needed. Everything is warmed before the session so no turn pays a cold
start.

**Tested by one learner.** One person, one accent, one level. Everything in the
regression suite came from something that actually broke, and the failures of
other learners have not been seen yet.

**Not accessible.** No screen reader support, and no keyboard alternative to the
push-to-talk gesture beyond the space bar.

<br>

## Tools

| | |
|---|---|
| `python tests/regression.py` | 421 checks. No microphone, no model, no network. Seconds. |
| **Compare your models** | every installed language model, over turns taken from sessions that went wrong |
| `python tests/bench_asr.py` | every recogniser on disk, over real recordings in `tests/fixtures/` |
| `python tests/probe_audio.py <file.wav>` | one recording in detail: where its energy sits, and what each recogniser makes of it before and after a high-pass filter |
| `python tests/label_session.py` | walks a finished session, two questions per turn, then prints how many turns moved on without the learner fighting the machine |

That last number is the only real measure. A passing test count says the parts
work. It does not say the conversation did.

<br>

## Configuration

Situations are one-line strings in `seeds.json`. Levels and reply lengths are in
`levels.json`, overridden by the slider on the start screen. Words the recogniser
is nudged towards are in `hotwords.json`; keep it under about sixty entries,
because a longer list dilutes the effect rather than strengthening it.

<details>
<summary>Every environment variable, none of which need changing</summary>
<br>

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

</details>

<br>

## Privacy

Everything is local. No account, no telemetry, no network traffic during a
session.

Recordings are written to a temporary folder so you can play them back and so
failures can be diagnosed. `End session` offers to delete them, and there is a
wipe that removes every recording of your voice. Synthesised speech is cached
separately and is not a recording of you.

<br>

## Contributing

**The most useful contribution is a real failure.** Find the recording (the
failure block on screen names the turn id), copy it into `tests/fixtures/`, add
an entry to `fixtures.json` with what you were actually trying to say, and open
an issue with the audio measurements from the screen.

Real failures beat synthetic audio and beat clean native-speaker samples.

Changing code: run `python tests/regression.py` before and after and do not
reduce the pass count. Read `DECISIONS.md` before proposing anything structural.

<br>

## Where this came from

Built over some weeks for one learner, in conversation with an assistant, by
shipping a version, using it, and reporting exactly what went wrong.

The code is not the interesting artefact. `DECISIONS.md` is.

<br>

---

<div align="center">
<br>

MIT. See [LICENSE](LICENSE).

<br>
</div>
