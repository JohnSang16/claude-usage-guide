# Claude Usage Optimal Standards

By John Sang. A simple guide to how Claude usage is counted, why it gets expensive, and how to keep it low. Written so a person or an AI assistant can read it and apply it.

**The one rule: keep the context window as lean as possible.**

## Part 1: How usage works

### Claude runs on tokens

A token is a small chunk of text, about 3/4 of a word. Everything Claude does costs tokens, and you are charged for them. Relative cost per token, priced against plain reading:

| What Claude does | What it is | Relative cost |
|---|---|---|
| Reading | Taking in text: your message, the earlier chat, a file | 1x |
| Reading saved notes | Re-reading text it has cached | about 0.1x |
| Saving notes | Storing text in the cache the first time | about 1.25x |
| Replying | Writing text back to you | about 5x |
| Tool call | Opening a file or running a command | Another round of reading the chat, plus what the tool returns |

### The catch: it re-reads everything

Claude has no memory. Every time you send a message, it re-reads the whole conversation from the start, then answers your newest message. Think of a tutor who forgets everything the second they finish talking, so you hand them the full binder of notes every time. One answer with 10 tool calls hands over the binder 10 times.

**Cache:** Claude's saved notes on the part of the binder it already read. If that part is exactly the same as last time, it uses the notes and charges only about 10% of the reading price. Saving the notes the first time costs about 25% extra. The notes are thrown out after about 5 minutes of no activity.

Picture the chat as 20 pages. Page 1 is the setup Claude gets before you say anything (your instructions file, its tools, which model you picked). The rest are messages and replies.

- **Adding to the end is cheap.** You send page 20. Pages 1 to 19 are identical to last time, so the saved notes cover them. Only page 20 is new.
- **Changing something earlier is expensive.** You change the setup on page 1 (switch models, edit your instructions file, add a tool). The saved notes only work if everything before a page is unchanged, so none of them can be reused. All 20 pages are re-read at full price and new notes are saved.

## Part 2: What makes it cost so much

In my case, I spent $1,300+ in a single month. In my local chat logs, 65% of my spend was re-reading old chat, 22% was writing notes, and only 13% was replies. My longest chat ran 1,141 messages, re-reading about 200k tokens per message.

The model was not the problem. Opus was 59% of that spend, but switching everything to Sonnet would have saved only about 12%, because re-reading costs about the same on both at current prices. The real driver was chat length.

### Common pitfalls

- **Never clearing:** one chat for many tasks, so every message re-sends all of it.
- **Big things in the chat:** a large file or log stays and is re-read every message.
- **Many small follow ups:** each one re-sends the whole binder.
- **Idle gaps:** over 5 minutes of silence rebuilds the notes at full price.
- **Changing model or tools mid chat:** the notes do not carry over.

## Part 3: The solution

**Keep the context window as lean as possible.**

### 5 core rules

1. **Small chat.** One task per chat. When it is done, start fresh. *Why: every old message is re-read each time you send a new one.*
2. **Fewer messages.** Decide what you want, then batch your questions into one message instead of sending them one at a time. *Why: each message re-sends the whole chat.*
3. **Right model.** Sonnet by default, Opus for hard reasoning. *Why: Opus costs about twice as much for new text and replies.*
4. **Delegate.** Keep the main chat for planning. Send big jobs to a subagent: a helper chat with its own fresh binder that returns only a short summary. Give it a clear brief: its goal, the files, what to return, and a short handoff when done. *Why: the big read never enters your main chat.*
5. **Keep the notes warm.** No model or tool changes mid chat, and no long breaks mid task. *Why: a change early on makes Claude re-read everything at full price.*

### The 4 commands (Claude Code)

| Command | What it does |
|---|---|
| `/clear` | Wipes the chat and starts fresh with an empty binder. Claude forgets everything, so paste back anything it still needs. |
| `/compact` | Squeezes the chat into a short summary and keeps going in the same conversation. A smaller binder, same thread. |
| `/context` | Shows what is filling the context window (messages, files, tools, instructions) so you can see what to cut. |
| `/cost` | Shows what the current session has cost so far. |

### When this happens, do this

| Situation | Action |
|---|---|
| Task done, or next task is unrelated | `/clear` |
| Chat is getting long (over 10 to 15% of the context window used) | `/compact`, or `/clear` and paste a short summary |
| You want a different model | `/clear` first |
| Big file or research | Search or read only the lines you need, give a file path instead of pasting, or use a subagent |
| Not sure what is using space | `/context` shows the binder, `/cost` shows the bill |

### Write short and specific

Each reply token costs about 5x a read token, and your wording sets how long replies run. The bigger cost is still the re-reading, so shorter chats come first, but short answers help too. Put instructions in a short list, then the material under a label:

```
instructions:
1. review each snippet for bugs only
2. return: file, line, fix
3. no intro, no summary, no extra options

material:
[paste code here]
```

- **Get to the point:** skip the greeting and state the ask.
- **Name the output:** say what you want back (a table, a list, one paragraph) and how long.
- **Say what not to do:** "no recap, no extra options" saves a reply you would throw away.
- **Batch:** number several questions in one message.

### Keeping long term history cheaply

Keep your history in a filing cabinet of notes, not in the backpack you carry every message.

Claude re-reads the whole chat every time you send a message, so a long chat gets expensive. Think of the chat as a backpack you carry all day: the more papers inside, the heavier every trip. So don't keep your history in the backpack. Keep it in a filing cabinet, which is a folder of plain text notes (Obsidian is a nice way to browse them, but Claude just reads the files). When you need something old, Claude pulls out just that one paper and puts it back. The backpack stays light and nothing is lost.

1. **Keep a short index:** one file listing what each note covers.
2. **Look up on demand:** search for the topic and open only that note, never "read all my notes".
3. **Write results back:** save the conclusion to a note, then `/clear`.

What quietly cancels the savings, like stuffing the cabinet back into the backpack:

- **Files loaded on every chat:** an instructions file or memory index is read at the start of every chat, so a bloated one is paid for on every message. Keep these short.
- **Loading too much up front:** starting every chat by loading all your history brings the big binder back.
- **Notes that only grow:** a log that gets longer every week costs more each time it is opened. Shrink finished entries to their conclusion.

For the full setup, see my Obsidian guide: [second-brain-guide](https://github.com/JohnSang16/second-brain-guide), a student-specific guide to pairing Obsidian and Claude Code into an agentic second brain.

---

*Numbers are from my own local chat logs at list API prices, not an invoice. Prices change, so re-check them.*
