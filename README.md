# claude usage optimal standards

a simple guide to how claude usage is counted, why it gets expensive, and how to keep it low.

**the main rule: keep the context window as lean as possible.**

## part 1: how usage works

### claude runs on tokens

a token is a small chunk of text, about 3/4 of a word. everything claude does costs tokens, and you are charged for them. relative cost per token, priced against plain reading:

| what claude does | what it is | relative cost |
|---|---|---|
| reading | taking in text: your message, the earlier chat, a file | 1x |
| reading saved notes | re-reading text it has cached | about 0.1x |
| saving notes | storing text in the cache the first time | about 1.25x |
| replying | writing text back to you | about 5x |
| tool call | opening a file or running a command | another round of reading the chat, plus what the tool returns |

### the catch: it re-reads everything

claude has no memory, so every message re-reads the whole conversation first. think of a tutor who forgets everything the second they finish talking, so you hand them the full binder of notes every time. one answer with 10 tool calls hands over the binder 10 times.

**cache:** claude's saved notes on the part of the binder it already read. if that part is exactly the same as last time, it uses the notes and charges only about 10% of the reading price. saving the notes the first time costs about 25% extra. the notes are thrown out after about 5 minutes of no activity.

picture the chat as 20 pages. page 1 is the setup claude gets before you say anything (your instructions file, its tools, which model you picked). the rest are messages and replies.

- **adding to the end is cheap.** you send page 20. pages 1 to 19 are identical to last time, so the saved notes cover them. only page 20 is new.
- **changing something earlier is expensive.** you change the setup on page 1 (switch models, edit your instructions file, add a tool). the saved notes only work if everything before a page is unchanged, so none of them can be reused. all 20 pages are re-read at full price and new notes are saved.

## part 2: what makes it cost so much

in my case, i processed about 2 billion tokens in a single month. out of every 100 tokens, about 98 were re-reading old chat, about 2 were saving notes, and fewer than 1 was a reply. my longest chat ran 1,141 messages, re-reading about 200k tokens per message.

replies are so few they barely count, and re-reading costs about the same on opus and sonnet. the real driver was chat length.

### common pitfalls

- **never clearing:** one chat for many tasks, so every message re-sends all of it.
- **big things in the chat:** a large file or log stays and is re-read every message.
- **many small follow ups:** each one re-sends the whole binder.
- **idle gaps:** over 5 minutes of silence rebuilds the notes at full price.
- **changing model or tools mid chat:** the notes do not carry over.

## part 3: the solution

**keep the context window as lean as possible.**

### 5 core rules

1. **delegate.** keep the main chat lean. use it for planning and decisions, and hand big jobs to a helper. *why: the big read never enters your main chat.*
2. **small chat.** one task per chat, then start fresh. a subagent is a helper chat that returns only a short summary, so give it a clear brief. *why: every old message is re-read each time you send a new one.*
3. **define done.** before you start, write down the goal and what "done" looks like. a scoping template or skill helps if your team has one. *why: no aim means a wandering, long chat.*
4. **fewer messages.** decide what you want, then batch it into one message instead of sending small asks one after another. *why: each message re-sends the whole chat.*
5. **right model, steady setup.** sonnet by default, opus for hard reasoning. do not change the model or tools mid chat, and avoid long breaks mid task. *why: opus costs about twice as much, and changing the model or tools mid chat makes claude re-read everything at full price.*

### when the task is bigger

as devs, we make big code changes on prod, and those can be big tasks. break them into smaller tasks using the SDD (spec-driven development) idea: write the spec or plan first, then build from it one stage at a time.

two additions keep it cheap: run each stage in a fresh context window (a new chat), and end each stage with a short handoff note (done, changed, still open) so the next stage starts from the plan plus the note, not the old context. to get a fresh window, `/clear` or start a new chat, or tell claude "run stage 2 in a subagent": it works in its own window and returns only a short summary, which is the delegate rule.

the plan file and handoff notes are the memory, and the chat is only the workspace for one stage.

### commands: when to use what

| when this happens | do this |
|---|---|
| task done or unrelated next task | `/clear`: wipes the context window and starts fresh. paste back anything claude still needs. |
| chat is getting long (over 10 to 15% of the context window used) | `/compact`: squeezes the chat into a short summary, same thread. or `/clear` and paste a short summary. |
| you want a different model | `/clear` first, then switch. |
| not sure what is filling the chat | `/context`: shows what is filling the window so you can see what to cut. |
| want to see the bill | `/cost`: shows what the session has cost so far. |
| big file or research | search or read only the lines you need, give a file path instead of pasting, or use a subagent. |

**pro tip:** run `claude --resume` to look through old chats and reopen one.

### write short and specific

replies cost about 5x a read token, and your wording sets their length. put instructions in a short list, then the material under a label:

```
instructions:
1. review each snippet for bugs only
2. return: file, line, fix
3. no intro, no summary, no extra options

material:
[paste code here]
```

- **get to the point:** skip the greeting and state the ask.
- **name the output:** say what you want back (a table, a list, one paragraph) and how long.
- **say what not to do:** "no recap, no extra options" saves a reply you would throw away.

### keeping long term history cheaply

to pick work back up tomorrow, or revisit an old project without searching chat logs, keep your history as plain text notes in a folder (obsidian or any file manager works) and let claude open only the note it needs.

1. **keep a short index:** one file listing what each note covers.
2. **look up on demand:** open only the note you need, never "read all my notes".
3. **write results back:** save the conclusion to a note, then `/clear`.

what quietly cancels the savings:

- **bloated always-loaded files:** instructions and memory files are read every chat, so keep them short.
- **loading all history up front:** brings the big backpack back.
- **notes that only grow:** shrink finished entries to their conclusion.

for the full setup, see my obsidian guide: [second-brain-guide](https://github.com/JohnSang16/second-brain-guide), a student-specific guide to pairing obsidian and claude code into an agentic second brain.
