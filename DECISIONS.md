# Decisions

Why this thing is shaped the way it is. Read before changing anything
structural. Most entries exist because the opposite was tried first and failed
in a way that took hours to find.

If you are an assistant picking this project up: run `python tests/regression.py`
before and after every change. Do not reduce the pass count. Several rounds of
this project were lost to fixing one thing and silently breaking another.

---

## What this is

A local conversation partner for spoken Mandarin, for one learner: an adult at
HSK 2 to 3, studying with a private teacher, learning
out of curiosity. Target register is taxi, restaurant, small talk. Native
languages English, Tamil, Kannada.

It is not a tutor, a drill app, or a grading system. The other person in the
conversation is a person, never a teacher.

Everything runs on the machine. No account, no cloud, no metering.

---

## The architecture, and the version that failed

**There is a model between the learner and the conversation. It chooses. It
cannot write.**

Reversed on 16 August 2026, deliberately, and this entry is the record of why.

The first design put a model there and asked it to "work out what the learner
meant". It had no access to the audio and every incentive to write a better
sentence than the one that was said, so it did: `你好,去啦吗?` became
`你好，要去机场了吗？` and the conversation carried on as though that had been
said. Roughly a dozen builds were spent adding guards before it was pulled out.

It was pulled out for two years of session time and the mishearing did not stop.
`我到了` arrived as `我到楼`. `公园` arrived as `共游`. Five turns in one session
came back as some arrangement of the word Boss, on two different Whisper models,
with and without the primer. The dictionary can generate the right reading. It
cannot tell which of its readings belongs here.

So the layer is back, under one constraint that the first version did not have.
The resolver is handed a numbered list of utterances that the sounds already
support and it returns a number. `RESOLVE_SCHEMA` has two fields, an integer and
a boolean. There is nowhere in its output that a sentence could go. It can pick
wrong, and there is no arrangement of tokens it can emit that puts a word into
the conversation which was not already a reading of the syllables heard.

`candidate_pool()` takes what was heard and a count. It has no parameter for the
conversation, which is how context is kept to ranking the list rather than
adding to it. Every clause of that is a regression test.

Two other rules follow from the same reasoning:

- A turn heard clearly enough makes no resolver call at all. Most turns are
  clear, and a clear turn costs one recogniser pass. Ambiguity work is paid for
  only where there is ambiguity.
- A resolver that answers nonsense, or cannot be reached, does not eat the turn.
  The recogniser's own first choice stands, marked uncertain. An explicit -1 is
  the only thing that means none of these, and -1 sends you to typing.

**Confidence comes from the recogniser, not from a model's opinion of itself.**
`exp(mean(avg_logprob))` over segments. An earlier version asked the language
model to rate its own certainty, which measures nothing. Three bands, from the
spoken dialogue literature (Bohus 2007, Skantze 2007):

| band | behaviour |
|---|---|
| above `ACCEPT_AT` | accepted silently |
| between | accepted, marked not certain |
| below `CHECK_AT` | not accepted; the other person asks about it |

The number is never shown as "confidence" in the normal interface. It reads as
a percentage and is not one.

**Uncertain turns are not gated behind a confirmation step.** This was asked
for twice by reviewers and declined twice. The learner explicitly rejected a
friction layer, wanting the behaviour of a search engine: pick the likely
reading, answer, and mention the correction quietly. The dialogue research
agrees for low-purposiveness tasks, where implicit confirmation is preferred.
`CONFIRM_UNCERTAIN=1` exists for anyone who disagrees. Off by default.

**Clarifications are targeted, not "please repeat".** When a turn is not
caught, the other person re-asks their own question in simpler words. The
literature is consistent that this beats a bare repeat request. Those turns are
tagged `kind: clarification` and only the most recent survives in history,
because otherwise a run of bad audio fills the model's context with its own
re-asks.

---

## The recogniser

**large-v2 did not fix the looping.** The switch away from turbo was right and
it was not sufficient. On 16 August 2026, all on large-v2: `Boss` at 0.43,
`Boss Boss` at 0.48, `蒙` eighty times at 0.84, `蛮 Boss` sixteen times at 0.60.
The medium fallback independently produced `Boss Boss` on two of the same clips.
Whisper is English first and falls back into English when it loses the thread.
Everything downstream of it is load bearing, not belt and braces.

`tests/bench_asr.py` scores every recogniser on disk against real recordings in
`tests/fixtures/`. The score is two points for hearing it outright and one for
leaving it reachable from the chips, which is the number that matters. Add a
fixture every time a turn goes wrong.

**Default is `whisper-large-v2-mlx`. It must not be turbo.**

The project shipped with `large-v3-turbo` for months of session time because it
was fast. Turbo has four decoder layers instead of thirty-two and is the most
repetition-prone Whisper variant, with the worst degradation on non-English. On
learner Mandarin it produced `yine yine yine` and `各位各位各位` on recordings
measured at 1.8 seconds and 72% speech. The audio was fine. The model was
wrong, and everything else was tuned around that mistake for days.

The recogniser is now selectable from the start screen. If turns come back as
repeated nonsense, that dropdown is the first thing to change.

**Audio is decoded and measured on the server.** ffmpeg to 16kHz mono, with
silence trimmed from both ends, then duration, loudness, peak and the fraction
that is actually speech. Push-to-talk naturally produces a burst of speech
surrounded by silence, which is the exact shape that makes Whisper hallucinate.
Trimming turned a test case from 2.65s at 5% voiced into 0.2s at 78% voiced.

Those numbers appear on screen when a turn fails. Without them there is no way
to tell a bad recording from a bad recogniser, and weeks were spent guessing.

**The recogniser prompt must stay short.** Around twelve words. At roughly
twenty-eight it began echoing the prompt back and emitting English repetition
loops. `PRIMER_WORDS` controls it; 0 turns priming off for comparison. Words
are chosen for relevance to the current topic, plus a fixed core of short
answers, because two-syllable replies give the decoder nothing to work with.

**Failures escalate rather than give up.** A looping transcription is retried
without the prompt, then on a different recogniser, then salvaged by keeping
whatever came before the repetition. `我去公司wardwardward…` is a real turn
wearing a broken tail.

---

## Spelling out what was typed

**A model reads typed pinyin into characters, and code checks that it did.**

The dictionary keeps one word per sound and cannot do this. Every measure word
in the language loses: 杯 to 北, 碗 to 玩, 份 to 分, 瓶 to 平, and 鸡 to 及. A
whole session went on ordering chicken noodles, a water and a tea, and produced
一北茶, 一玩面条, 三平水 and 记得面条, remember the noodles. A learner in a
restaurant cannot order anything.

Widening the index to the runners up was tried three times and failed three
times. 鸡 is the fifteenth commonest word for the sound `ji`, so no beam width
reaches it, and every widening floods the readings with 握 窝 卧 for 我. Which
character goes with which is a language question and frequency is the wrong
instrument.

This is not the layer that was pulled out of the project. That one was asked
what the learner meant, had no access to the audio, and wrote whole sentences
nobody had said. This one is handed syllables the learner typed with their own
fingers, which are not evidence of anything: they are the thing itself.

The safety is one comparison. `spells_out()` reads the answer back into pinyin
and requires it to equal exactly the syllables typed, erhua either way round.
我想要一杯热茶谢谢 does not spell out to `yi bei cha` and is thrown away. There is
no string it can return that the learner did not type.

Three ways it ends, all tested:

| | |
|---|---|
| it spells out | used |
| it writes something else | rejected, the dictionary answers |
| ollama is not running | the dictionary answers |

Typing has always been the escape hatch that must never fail. It now degrades to
exactly the behaviour it had before, which is wrong but honest, rather than
failing.

`SPELL_SCHEMA` has one field and it holds characters. The reader is told the
conversation and the situation, because in a noodle shop `ji de miantiao` is
鸡的面条 and not 记得面条, and that is a fact about this conversation rather than
an ontology anyone had to write down.

## The dictionary

Turning typed pinyin into characters is a dictionary lookup, not a job for a
model. A model was tried and failed on `wo zai jia`, which is the point at
which the escape hatch stopped working. It now uses a reverse index built from
the segmenter's word list with best-path segmentation over log probabilities.

Three things went wrong on the way, all worth not repeating:

- **jieba loads its dictionary lazily.** `jieba.dt.FREQ` is empty until
  `jieba.initialize()` is called. The index was built at startup from zero
  words, silently, and behaved differently depending on what had run first.
- **Greedy longest-match is wrong.** It gave 我区级常 for `wo qu jichang`,
  because 区级 is a real word. Best-path over probabilities fixes it.
- **Summing log frequencies rewards using more words.** Log probabilities do
  not. And the normaliser must be computed from real corpus frequencies, before
  the artificial boosts, or rare words win again.

**Erhua is expanded before lookup.** `dianr` is written the way people write
it and the index is keyed `dian|er`, because that is what the segmenter produces
for 一点儿. Without expansion `man yidianr ma` parses into perfectly good
syllables, matches nothing, and comes back as could not read that. Only `er`
legitimately ends in r, so anything else that does is erhua and expanding it
risks nothing.

This was found from a real recording. `man yidianr ma`, a bit slower, said to a
driver who had just announced he would have to speed up. Whisper caught the
first syllable as 蛮 and collapsed into `Boss` sixteen times. Typing it back was
supposed to be the way out and was not.

**The core-character boost is decided by frequency, not by string order.** Two
core characters sharing a sound used to be resolved by whichever came last in
the string. 以 is only in the set because it sits inside 可以, and it took `yi`
from 一, which the segmenter ranks at 217830 against 136106. `man yidian ma`
came out as 慢以点吗. 一 was missing from the set entirely and is now in it.

Two things were tried here and reverted, both measured:

- **Making the boost a floor instead of an overwrite.** Worse across the board.
  想 became 向, 慢 became 满, 一杯 became 以北, 请 became 轻. The boost is doing
  real work.
- **Widening the set to the numbers and measure words.** Every character added
  at that weight takes its sound from whatever held it. 二 took 儿, 杯 took 北,
  万 took 玩, 前 took 钱, 时 took 是, and 半个小时 became 半个小是. Multi-syllable
  words are already reachable through the dictionary, which is where 三个人 and
  两杯茶 come from.

**What the one-word-per-sound index still gets wrong.** Listed rather than
asserted, because it is a known limit and not a regression. The suite prints
them on every run so they stay visible:

    wo yao yi bei cha       我要一北茶      wanted 我要一杯茶
    shi kuai qian           是快前         wanted 十块钱
    ban ge xiaoshi          半个小是       wanted 半个小时
    wo de xingli bu zhong   我的行里不中     wanted 我的行李不重

All four are the same shape: a frequent single character beating the right one
for a sound where no multi-syllable dictionary word covers the pair. Fixing it
means more than one word per key and a way to choose between them, which needs
a check for whether a character sequence is grammatical. See the note above on
why the cheap versions of that do not work.

Syllable boundaries are kept in the key, so `xi'an` (西安) and `xian` (咸) are
different lookups.

The same index generates possible readings: alternative segmentations of the
same syllables, syllable swaps across the confusions learners actually make
(including aspiration, which Tamil and Kannada handle differently), sentence
final particles, and a fuzzy match against the learner's own word list. That
last one is how 共游 `gongyou` reaches 公园 `gongyuan`, three letters apart.

Readings are shown on uncertain turns and on not-caught turns. They were hidden
on not-caught turns for two builds, which was wrong: that is exactly when they
are the only thing that can rescue the turn.

---

## Reading before answering

**A reply says what it understood before it writes anything.** `TURN_SCHEMA`
opens with `reading`, one line of English on what the learner meant and whether
anything about it is worth remarking on. It is filled first, on purpose: a field
written after the sentence is a caption rather than a reading.

Four turns in one session were correct and empty:

| the learner said | the reply | what a person says |
|---|---|---|
| 中辣, to a waiter who doubts they can take spice | he offered to add spicy chicken | 中辣？行，试试看 |
| the noodles are delicious, before they arrived | thank you, we always take care | you have not eaten yet |
| three bottles of water, for one person | certainly, one moment | that is a lot |
| slow down, while running late | be careful | not in a hurry any more? |

Nothing is wrong in any of those. They are service-desk acknowledgements, and
the interesting thing in every one is the gap between what was said and what was
expected. A model asked for a sentence writes a sentence. Asked first what the
turn means, it has somewhere to reply from.

The reading is also on screen, folded away behind a link on the turn. When a
reply lands flat it says whether the model misread the learner or read them and
had nothing to say. Those are different faults and were indistinguishable from
outside.

**The reply budget is settable from the start screen.** Twenty-six characters at
HSK 2 had no room to notice something and also answer it, so the model answered
and dropped the noticing: 那给您加辣的鸡。 is eight characters and it is a receipt.
Defaults raised to 22, 40, 60 and 90, and there is a slider, because the right
number depends on how the learner's Mandarin is on the day and a server cannot
reason about that. Clamped at both ends.

**Comparing models is in the app.** `tests/bench_reply.py` shipped two builds
ago and the learner's response was "what benchmark?", which is a fair answer to
a script in a directory nobody opens. There is a button on the start screen now.
It runs every installed model over six turns taken from sessions that went
wrong, and scores each reply on repeats, echoes, doubled questions and length.
Nothing in the scoring judges meaning: the replies are printed and the learner
reads them. Each case carries a line about what it is for, so the point of it is
on screen rather than in a comment here.

## Where the line is

Code enforces turn taking and truth boundaries. The model understands and
conducts the conversation. Nothing in code decides what a conversation means or
where it should go.

This was written down on 16 August 2026 after four builds of drift, and the
drift is worth describing because it happened one reasonable step at a time.

A driver asked his passenger which exit to take, so a check went in requiring
every question to contain 你. A driver invented Beijing, so a list of thirty
place names went in. A driver asked the same thing three times, so seeds.json
grew `wants`, `knows` and `moves_on`, and an assistant wrote out for fifteen
scenes what each person knew and where each conversation could travel. A driver
ignored a no, so lists of negative and affirmative characters went in.

Every one of those fixed the thing in front of it. Together they turned the
other person into a machine that echoed the learner's words and then asked a
question with 你 in it, every single turn, and the learner's word for it was
hard coded logic. The brief had reached 490 words, most of it about the Great
Wall and airport terminals.

**Removed.** The place list, the polarity word lists, the rule that a question
must be about 你, and every `wants`, `knows` and `moves_on` in seeds.json. A
scene is one line the learner writes. A model that knows what a waiter is does
not need an assistant to tell it what a waiter knows.

**The brief is the four rules and nothing else**, 167 words:

> Respond naturally to the latest thing they said. Do not repeat a question
> whose answer is already evident from the conversation. Do not force the
> conversation back onto an earlier thread. You may react, answer, comment,
> volunteer information, ask something relevant, or naturally move the
> conversation forward.

**Kept, and why.** Three invariants and two structural checks.

| | |
|---|---|
| one reply per accepted turn | a turn token; two replies rendered for one turn |
| rejected ASR never enters history | evidence is not truth |
| nothing downloads during a session | a seven minute stall mid conversation |
| a person does not say the same sentence twice | text against text, no claim about meaning |
| a person does not open by repeating your words | text against text, same |

The repetition check stays in code rather than moving to the model. A loop is
the most damaging failure a learner can hit, because it ends the session. The
check is deterministic, costs nothing on a clean turn, and makes no claim about
what anything means. Detection is code; the replacement turn is the model's, one
retry, no forced shape.

How a question is marked in Chinese is grammar rather than ontology, so
`questions_in()` stays. What a question ought to be about was ontology, and it
is gone.

The recognition side is a different job. The dictionary, the syllable
confusions, the core characters and the hotwords exist to get the sounds right,
not to decide what conversations mean, and they are defended by tests. They stay.

**`tests/bench_reply.py`** answers whether a model can carry this without the
scaffolding. Real conversation states from real sessions, every installed model,
scored on properties of text against text: repeats, echoes, two questions, all
one shape, over length. Run it before arguing about whether a rule belongs in
code or in the prompt.

## What is evidence and what is true

Four different things, and conflating any two of them is where the bad turns
come from:

| | |
|---|---|
| what the microphone caught | the recording, kept |
| what the recogniser thinks | `raw_asr`, evidence, never truth |
| what the learner meant | the accepted turn, and only this crosses into the conversation |
| what the other person says | generated from the accepted turn |

**Only an accepted turn reaches the conversation.** A turn becomes accepted in
exactly three ways: the sounds were credible enough on their own, the learner
tapped a chip, or the learner typed it. Everything else stays evidence. History
carries accepted turns and real partner turns, and nothing else: no raw ASR, no
rejected candidates, no resolver guesses that were turned down.

**When the resolver rejects the pool, there are no chips.** Offering them anyway
contradicts the only judgement in the system qualified to make the call, and it
happened: `Yào miǎn xiǎo qiàn zài nǎlǐ` was offered under "did you mean" twice
more with the syllables shuffled. The pool is built by walking a dictionary over
syllables, so most of it is strings of real Chinese words that nobody would say.
Three lines of that costs something to read and none of it is the answer. The
pool is kept in `considered` for diagnostics and not shown.

**There is no cheap test for whether a string is real Mandarin.** Two were tried
and measured on real failures. Segmentation statistics do not separate: 我到了 and
我到楼 have identical token counts, single-character fractions and longest-word
lengths. A lattice round trip does not separate either: 不是有面在洗手间 round trips
perfectly and 洗手间在哪里 does not, because 哪 and 那 swap. The judgement needs a
model, the resolver is the model, and its prompt now leads with the fact that
most of what it will be shown is rubbish.

**How much was said changes what the score means.** A decoder that has lost
the thread invents short things: `Boss`, 我估计到, 你告诉我. It does not invent
我住在孟买你呢 and hold it together for seven characters.

The free pass is 0.62 at four characters or fewer and 0.46 above that. Measured
against every turn of three real sessions: thirteen of fourteen land where they
should, and the fourteenth costs a resolver call rather than an error.

This was the second fix for the same failure. The first, splitting `natural`
out of `pick` so a resolver could say "nothing beats what was heard" without
meaning "throw it away", was necessary and not sufficient: 我住在孟买你呢 at 0.58,
in direct answer to a question about Mumbai, was sent to the resolver and the
resolver called it unnatural. It is plainly natural. A model's opinion is now
not the only thing standing between a correct transcription and the bin.

**A place nobody has mentioned was invented.** A driver taking someone to the
airport decided they had arrived in Beijing that morning and asked which city
they had set out from. Twice, while driving them out of the country.

`invented_places()` checks the reply against everything anyone has actually
said plus the scene itself. It is a list of the places that come up in this
register, so it is narrow on purpose. The seed also now states which direction
the journey goes, since the model had it backwards.

**An echo is not a question, and neither is a tag, but a question word beats
both.** 去印度啊？ is someone repeating what they heard. 是吗？ is a reaction.
从哪座城市出发呀？ ends on the same particle as the first and is plainly a
question, because 哪 is in it. 吗 and 呢 are deliberately left out of that test,
because they turn up inside quotes of what the other person just said, and
你说呢就是不知道呀 is not a question either.

**A mediocre decoder score is not a veto.** `我的行李不重`, heard correctly in
direct answer to `你行李重不重`, was thrown away at 0.50 and again at 0.58. The
recogniser had done its job. The layer above refused to believe it.

The cause was a two-field resolver that could not tell these apart:

- nothing in the pool beats what was heard
- what was heard is not a thing anyone would say

Both came back as -1 and -1 meant a dead end. The resolver now answers three
questions. `natural` asks whether what was heard is real spoken Mandarin, and it
is the only route to a dead end. `pick` returning -1 means what was heard
stands. What the recogniser produced does not need permission to survive.

The prompt says so as well: option 0 starts ahead, and a sentence that directly
answers the question just asked is strong evidence even when the decoder was
unsure of itself. The .42 prompt had been over-corrected the other way, told
that most of what it would see was rubbish and that -1 was usually right, and it
duly rejected a correct transcription twice. Both directions are now in the
prompt with a worked example each way.

**Saying it once is checked, not requested.** Read as a conversation rather
than as turns, one session went: the new terminal is nice, the new terminal is
very big, the new airport is indeed big. And: is this your first time here, have
you flown this flight before, have you taken other flights before. Three
tellings of one fact and three askings of one question, in five turns.

A line in the brief saying never ask the same thing twice does not survive a
model rebuilding the scene each turn. `repeats_itself()` compares a generated
reply against everything that person has already said: a four character run
reused anywhere, or two questions sharing content words once function words are
stripped. When it fires, one regeneration, told what it repeated and to go
somewhere else. Only when it fires, so a clean turn costs nothing. Seven cases
from the real transcript are in the suite.

The reply prompt also gets the last six of that person's own lines with an
explicit instruction not to reuse them, and the note that a question they have
answered is finished.

**The turn shape is a set of moves, not one move.** Two whole sessions came
back in a single shape: the learner's words echoed, then a question with 你 in
it.

    喜欢长城啊，      那您去过吗？
    想起长城啊，      那您去过吗？
    朋友告诉您的啊，   那您现在去哪里呢？
    请你告诉我？      你有什么想分享的吗？

That last one echoed please tell me and then asked what they would like to
share. The learner's word for it was hard coded logic, and he was right.

It was written in. The brief said a short reaction to what they just said, then
one question about them, and the model read short reaction as repeat their words
with a particle. Then the check added two builds earlier, rejecting any question
without 你 in it, guaranteed that every turn ended in one. Fixing the driver
interviewing the passenger about his own driving turned him into an interviewer.

Three changes, and the first is the important one:

- The brief lists moves rather than a shape: say what you think, tell them
  something about yourself, agree and add, disagree, say what is happening
  around you, ask them something. Not the same one twice running.
- Most turns end in a question, not every turn. Deterministic and free:
  `asked_recently()`, and a third question in a row is refused.
- `echoes_learner()` catches the opener. Longest common substring against what
  the learner said: half the opening clause, or two characters plus a particle
  on the end. A shared place name is two characters and is a subject rather
  than a parrot, so the ratio rule needs three. Nine openers from the real
  sessions, no false positives.

**A reply that has been flagged does not go out.** 你对历史感兴趣吗 was asked
twice in one session and 那您去过吗 twice in another, both detected, both sent
anyway, because the retry was no better and the first attempt was kept.
`drop_question()` now takes the question off and keeps the reaction, which is a
smaller turn and an honest one. What is still wrong goes on the record.

**One reply per turn.** Picking a chip called `replace()`, which fired its own
reply while the original turn's reply was still in flight, and both rendered.
Two of the other person's turns in a row, in both sessions, both times right
after a chip. Every turn takes a number now and a superseded reply is discarded.
Chips are also inert while a reply is out.

**You can take the conversation somewhere else.** A button on the newest turn,
a box, plain English: ask me about my work, talk about food. It replaces that
turn rather than adding one. The rig cannot know what the learner wants to
practise today, and the learner can just say. This came from the learner, as the
answer to what should happen when a reply is flagged twice, and it turned out to
be worth having on every turn rather than only on failure.

**The question has to be one only they could answer.** A driver asked his
passenger whether the flight would be caught, how long the terminal was from
here, and whether the two of them would make it. Three times, about his own
driving, to the person in the back seat.

The tell is cheap and it held on every line of that session: a real question
with no 你 in it is almost never about them. 能赶上飞机吗, 现在去航站楼还要多久 and
能到吗 all lack it. 你赶时间吗, 你在哪个城市住 and 你还有行李要拿吗 all have it.
Eleven lines from the transcript, no false positives.

Two things are not questions and both had to be excluded before this worked.
An echo, someone repeating what they just heard: 去印度啊？ and 你说呢就是不知道呀。
A tag, a reaction with a question mark attached: 是吗？ 真的吗？ 好吗？ The clause
splitter also had to split on the question mark itself, or 去印度啊？那肯定很热闹
stays in one piece and the echo at the front is never seen for what it is.

`reply_problems()` returns all of it in words the model can act on, and there is
one retry carrying every problem at once. The second attempt only replaces the
first if it has fewer problems, so a worse retry is not an improvement. A clean
turn costs nothing.

The turn shape at the top of the brief now leads with both halves: a short
reaction, then one question about them. And it names what is the other person's
own business rather than the learner's, which is where every one of those three
questions came from.

**Two questions in a turn is caught too.** 你飞出去过吗？新航站楼很大，你坐哪趟航班？
is an interview, and the rule saying one question a turn had been in the brief
for three builds without holding.

**Each scenario says what that person knows and where the conversation can go.**
`knows` and `moves_on` in seeds.json, alongside `wants`. The driver asked which
exit to take and said he wanted to see it too, and asked which flight, having
been told neither. He knows the roads and the terminals and nothing about your
trip, and that is now written down.

`moves_on` exists because a driver with nowhere else to be asks the same
question in new words. Traffic, whether you will make it, where you are flying,
how long you have been in the city, the weather, his own hours. When a thread
closes there is somewhere to go.

**The brief was consolidated while being fixed.** It had reached twelve rules
and started repeating itself, which is the thing it warns against. Seven now,
and a test asserts it stays under 460 words.

**A reply may not assume the opposite of the answer just given.** "Have you
flown from here before" answered `不，我没有` came back as "which gate do you like
best". Good Mandarin, on topic, and only a sentence if the answer had been yes.

`answer_polarity()` reads the accepted turn against the question it answered and
returns no, yes or open. Deterministic, no model call, no latency. A no is
spelled out to the reply along with what it rules out. It handles the set
phrases that contain 不 without being denials, since 不好意思 and 差不多 turn up
constantly. Fourteen cases in the suite.

The reply instruction also works through it in order: what were you told
literally, what does that rule out, then react and ask. A question that assumes
the opposite is wrong even when the Mandarin is good and the topic is right.

**A turn that was not heard does not move the conversation.** It is an
interruption to where you are. The clarification is handed the exact question it
must re-ask and told not to ask anything else. Asked to "ask your own last
question again in simpler words" without the question in front of it, the model
wrote a new one: a question about the exit came back as a question about the
restroom and the thread was gone.

**The line just said outranks anything remembered.** The record of what has been
established is background. It is not a subject. `Nǐ gāngcái shuō qù jīchǎng,
cèsuǒ zài nǎ?` is what happens when a fact from five turns ago is treated as the
topic, and it also asked the passenger for something the driver would know
better, which the brief now forbids.

**The record holds facts, not what they imply.** Going to the airport is going
to the airport. It is not travelling for work and it is not being in a hurry.
A wrong record is worse than an empty one, because every turn after it is built
on it. State updates run while a reply is being spoken and carry the turn they
were computed from, so an older one landing last cannot undo a newer one.

**Every turn is on the record.** `session.jsonl` in the work folder, one line
per turn, with the layers kept apart: recording, raw ASR, candidate pool,
resolver verdict, accepted turn, what the reply was given, what it produced.
Without that separation a bug gets fixed in the wrong layer. The chips looked
like a chip problem and were a resolver problem. The driver changing the subject
looked like a persona problem and was one line in the clarification prompt.

**A passing test count is not the product metric.** 214 checks say the parts
work. They do not say the conversation did. `tests/label_session.py` walks a
finished session and asks two questions per turn, and prints the only number
that matters: how many turns moved on without the learner fighting the machine.

## Interface rules

**Pinyin and English only.** No Chinese characters anywhere in the interface.
The learner cannot read them yet. Characters exist internally for speech
synthesis and are available as an opt-in on the transcript export, because a
transcript a teacher can read is a different object from the screen.

**The raw transcription is never overwritten.** `raw_asr` and the accepted text
are separate fields. When they differ, the screen shows what was heard.

**Playback of your own voice is the only unmediated record.** Everything else
has been through a recogniser.

**Nothing is gated on a self-check.** The health list informs; the Start button
is never disabled. An earlier version locked the learner out of the app on the
strength of a check that was wrong.

**Failures are visible.** Every rejected turn appears as a block in the
conversation with the reason, the transcription, and the audio measurements.
Not a small grey line.

**Only the newest turn is editable.** Editing an older one discards everything
after it, which is correct but was confusing when any turn could be edited.

**Latin letters inside a Chinese turn mean the recogniser wandered, whatever
it scored.** 蒙 Boss came back at 0.45, above the line for accepting a turn with
a small mark against it, so the word Boss went into the conversation as a thing
that had been said out loud. Whisper is English first and falls back into
English when it loses the thread rather than admitting it has nothing. This
register contains no English, so any of it is a miss. Five turns in one session
came back as some arrangement of the word Boss, on two different Whisper models,
with and without the primer.

**A failed turn shows chips first, then an open typing box.** Chips because
tapping one takes a second and typing does not. The box open already because
getting past a bad turn should never cost two clicks. The box does not take the
caret when it opens by itself: the space bar starts a recording, and saying it
again is usually the faster way through. Tapping a chip goes down the typing
path rather than a second path beside it, so there is one commit route to keep
working instead of two. When nothing survives the wreckage there are no chips,
and the box is the whole answer, which is honest.

**Typed pinyin is read word by word.** `syllables()` is all or nothing by
design and returns nothing if any part of the string is not pinyin. Running a
whole line through it meant one unreadable word killed everything and the screen
said "could not read that" without saying which word. Typing back what the
screen had shown, Meng Boss, failed for exactly this reason. The line now reads
what it can, says which words it skipped, and the terminal logs the failures as
well as the successes.

**A failed turn can be typed in place.** Without that, a failure is a dead end
and the only way forward is editing an earlier turn, which destroys the thread.

---

## Operational

**The page and the server carry the same build number and compare them.** A
stale server running old code alongside a fresh page cost an entire evening of
diagnosing bugs that had already been fixed. If they disagree, the start screen
says so in red.

**Nothing downloads during a session.** Both recognisers are fetched by
`setup.sh`, which now stops if a fetch fails instead of warning and carrying
on. The server checks the disk before every transcription and refuses the turn
with a message rather than fetching. A session was lost to this: setup had
warned and continued, so the first turn pulled 3GB over fourteen minutes, and
later one badly heard turn pulled the 1.5GB fallback over another seven, both
with nothing on screen and a progress bar in a terminal window nobody was
watching. The start screen marks any recogniser that is not on disk.

**`start.sh` uses port 8765, kills only its own recorded process, and is not in
reload mode.**

**Non-finite numbers must never reach the browser.** Whisper returns `-inf` for
`avg_logprob` on a degenerate segment. Python writes that into JSON as a bare
`NaN`, which is invalid, and the browser rejects the entire response with a
parse error that explains nothing. Everything numeric is checked, the whole
response is scrubbed, a middleware turns any unhandled exception into JSON, and
the client reads bodies as text before parsing.

---

## Style

No em-dashes. No "not X but Y" constructions. No padding. Humour is welcome
when it does the teaching. Confirm the plan before building; the learner asked
for this repeatedly and it was ignored more than once, which wasted work on
both sides.

---

## Open

**A Mandarin-first recogniser.** SenseVoice or Paraformer via FunASR. Whisper
is English-first and its Chinese corpus is much smaller, which is the root of
the mishearing that no amount of downstream repair fully fixes. This is a
proper piece of work, not a patch, and should be scoped rather than bolted on.
Try large-v2 first: it may be enough.

**One word per sound in the dictionary.** The index keeps a single word for
each pinyin key, so `shi` is always 是 and 十 can never be reached. `xianzai shi
dian` reads as 现在是点 and no amount of the swap machinery gets to 现在十点,
because that machinery only ever changes the sounds, and here the sounds were
right. A second index of runners up was built and wired into the chips, and it
made them worse on every case tested: 我 attracted 握 窝 卧, which pushed 公园 and
我到了 out of the list. Reverted. Doing this properly needs a check for whether a
character sequence is grammatical, and `_sayable` only catches two particles in a
row. That is a language model over characters, which is a real piece of work.
The `lattice()` split was kept, since the chips need to know where the word
boundaries fell and a joined string has thrown that away.

**Tone feedback.** Deliberately absent. Recognisers reconstruct tone from
context rather than hearing it, so anything reported here would be invented.
Real tone practice needs pitch tracking against a reference contour, which is a
separate project of comparable size.

**Twenty to thirty turn sessions.** The remaining useful bugs will come from
actual use, not from more code review.
