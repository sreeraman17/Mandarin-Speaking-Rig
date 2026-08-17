# Speaking cues

{Assuming you have setup the rig}
One page. Ten lines you can say out loud on your first run, in two scenes, with
what should happen for each. Print it, or keep it on your phone.

If you cannot read pinyin with tone marks, stop here: the whole app is pinyin
and this card will not help you.

**Before you start.** Pick the matching situation on the start screen. Hold
SPACE, say the whole line as one phrase, let go. Do not go syllable by
syllable: a pause mid-phrase makes the recogniser guess, and guessing is where
it goes wrong. If it does not catch you, type the third column. That is not
cheating, it is the design.

---

## Scene 1 · `A driver taking me to the airport, and I'm running late`

| Say this | Means | If it misses, type |
|---|---|---|
| `Wǒ yào qù jīchǎng` | I want to go to the airport | `wo yao qu jichang` |
| `Hái yào duōjiǔ dào jīchǎng?` | How much longer to the airport? | `hai yao duo jiu dao jichang` |
| `Wǒ shíjiān bù duō le` | I do not have much time | `wo shijian bu duo le` |
| `Qǐng kāi màn yìdiǎnr` | Please drive a bit slower | `qing kai man yidianr` |
| `Wǒ méi lái guò zhèlǐ` | I have not been here before | `wo mei lai guo zheli` |

**What a working reply looks like.** Say `Wǒ méi lái guò zhèlǐ`. The driver must
not then ask which terminal you usually use or which airline you prefer, because
you have just told him you have never been. He should take the no and ask
something else, or tell you something about the place. If he asks you a question
that only makes sense had you said yes, that is the failure this line tests.

The fourth line is the other test. You told him you were short of time, then
asked him to slow down. A good partner notices the contradiction.

---

## Scene 2 · `A waiter at a noodle place who doesn't think I can handle spicy`

| Say this | Means | If it misses, type |
|---|---|---|
| `Wǒ yào yìwǎn jīdàn miàn` | I want a bowl of egg noodles | `wo yao yi wan jidan mian` |
| `Wǒ bù néng chī tài là de` | I cannot eat very spicy food | `wo bu neng chi tai la de` |
| `Yǒuméiyǒu bú là de cài?` | Do you have anything not spicy? | `you mei you bu la de cai` |
| `Qǐng zài gěi wǒ yì bēi chá` | Please bring me another cup of tea | `qing zai gei wo yi bei cha` |
| `Yígòng duōshao qián?` | How much altogether? | `yi gong duo shao qian` |

**What a working reply looks like.** This waiter does not believe you can take
spice. Tell him you cannot eat very spicy food and he should have an opinion
about it, not just write it down. Order the noodles and he should ask you
something a waiter asks, or say something about the kitchen. If every reply is
your own words repeated back followed by a question, that is the failure these
lines test.

---

## Reading the result

**Three things can happen to a turn.** All three are normal.

| on screen | what it means |
|---|---|
| your line in pinyin and English, then a reply | understood |
| your line with two or three chips under it | it was unsure. Tap the right one |
| a red block with the transcription and the audio numbers | it could not work it out. Type the line |

**Every reply has a `what it understood` link.** Open it. It shows the one line
of English the machine wrote about your turn before it answered. When a reply
feels off, that link tells you whether it misheard you or heard you and had
nothing to say. Those are different problems.

**Every reply has `take it somewhere else`.** Type an instruction in plain
English, like `ask me about my work` or `stop asking about noodles`, and the
turn is replaced. Use it the moment a conversation goes in circles.

---

## If something looks broken

**It misheard you.** Expected, especially in the first few minutes and
especially on short lines. The red block shows what it heard, the confidence
score, and how much of your recording was actually speech. If that last number
is low, the room is noisy or the microphone is far away. Type the line and carry
on.

**Typed pinyin came out as nonsense.** If you see `一北茶` for `yi bei cha` or
`一玩鸡蛋面` for `yi wan jidan mian`, Ollama is not reachable and the dictionary
is answering on its own. The dictionary keeps one word per sound and gets these
wrong: `杯` loses to `北` and `碗` loses to `玩`. Check that Ollama is running
and try again. These two lines are on the card partly to catch exactly this.

**Replies are grammatical and empty.** Press **Compare your models** on the
start screen. An 8B model produces replies that acknowledge you without engaging
with you. A 14B model is meaningfully better, and the comparison prints both so
you can see the difference rather than take my word for it.

**Nothing happens at all.** Press **Run a test conversation** on the start
screen. It runs the whole pipeline on a fixed sentence with no microphone, and
reports each stage separately, so a failure points at one component.

---

## For an engineer evaluating this in ten minutes

1. `./setup.sh`, then `./start.sh`. Setup is mostly downloading.
2. `python tests/regression.py`. 421 checks, no microphone, no model, no
   network. Should take seconds.
3. **Run a test conversation** on the start screen. This exercises the model and
   the voice.
4. **Compare your models**, if you have more than one installed.
5. Then the ten lines above, five in each scene.

If you do not speak any Mandarin, type the third column for all ten. The
conversation works identically, and everything except the speech recogniser is
being tested.

The one number worth having afterwards is from `python
tests/label_session.py`, which walks the session you just had and asks two
questions per turn. It prints how many turns moved on without you fighting the
machine. A passing test count says the parts work. That number says the
conversation did.

---

### The characters, for anyone who reads them

Not shown anywhere in the app, on purpose.

```
Scene 1   我要去机场 · 还要多久到机场 · 我时间不多了 · 请开慢一点儿 · 我没来过这里
Scene 2   我要一碗鸡蛋面 · 我不能吃太辣的 · 有没有不辣的菜 · 请再给我一杯茶 · 一共多少钱
```
