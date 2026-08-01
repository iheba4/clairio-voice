# CLAIRIO Voice

A co-presenter that stands on stage with you. It learns the talk beforehand, listens while you give it, picks up the thread when you lose it, and speaks to the audience when their attention starts to drift.

This is the voice half of a larger project. The other half watches the audience and reports when engagement drops.

## The problem it solves

Presenting is two jobs at once. You are delivering content and you are reading the room, and the second one is what falls apart when you are nervous. You lose your place, you fill the gap with noise, and by the time you recover the room has gone somewhere else.

So the system takes the second job. It knows the presentation as a partition, tracks where you are in it, and notices when you stall. If you stop mid sentence it can carry the next point. If the audience model reports attention falling, it can put a question to the room. The presenter stays in charge throughout and can cut it off with one key.

## Modules

| Folder | Role |
| --- | --- |
| `voice/` | The spoken identity. Streaming text to speech, interruptible mid sentence |
| `ears/` | Audio capture, voice activity detection, streaming transcription |
| `script/` | Turns a slide deck into `partition.json`, the learned presentation |
| `align/` | Tracks position in the partition, detects stalls and disfluency |
| `audience/` | Bridge to the engagement model |
| `director/` | Decision policy plus a language tier backed by an LLM |
| `stage/` | Turn taking, barge in handling, the audio output loop |
| `console/` | Operator dashboard, rehearsal mode, kill switch |

## What made this hard

Latency. Everything else was tractable.

A co-presenter that answers three seconds late is worse than none at all, because the audience watches you wait. I measured the local speech recognition path on the target machine and it needed three to four seconds per utterance, which no amount of tuning was going to fix on that CPU. That measurement changed the design rather than the parameters.

The response was to stop pretending the machine could do everything locally. Speech recognition moved to a cloud first path with a local fallback. Frequently used lines are synthesised ahead of time and cached, so common responses come back in milliseconds instead of hundreds. A fact bank answers predictable audience questions from memory. And a phone acts as a push to talk remote, which removes the hardest problem entirely by letting the presenter decide when the system is listening instead of making it guess.

There is also a manual mode that never interrupts, for when you want the safety net without the risk.

## Running it

    pip install -r requirements.txt
    cp .env.example .env          # then fill in your own keys
    python ears/capture.py --list                        # enumerate audio devices
    python voice/speak.py --say "Hello, I am CLAIRIO."   # check the voice works

Useful flags once it runs: `--rehearse` for a coaching pass over your delivery, `--profile` for preset behaviour, `--replay` to re-run a recorded session, `--manual` for the never interrupt mode.

The system speaks English and French. The stage language is selected in `director/lang.py`.

## Secrets

`.env` is gitignored and is not in this repository. `.env.example` lists every variable you need. Bring your own text to speech and language model credentials.

## Status

195 tests pass. The system went through six rounds of revision after the first live test failed, and the notes from each round are in `docs/`. `PROGRESS.md` is the running log, including what is still broken.
