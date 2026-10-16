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

claude has no memory. every time you send a message, it re-reads the whole conversation from the start, then answers your newest message. think of a tutor who forgets everything the second they finish talking, so you hand them the full binder of notes every time. one answer with 10 tool calls hands over the binder 10 times.

**cache:** claude's saved notes on the part of the binder it already read. if that part is exactly the same as last time, it uses the notes and charges only about 10% of the reading price. saving the notes the first time costs about 25% extra. the notes are thrown out after about 5 minutes of no activity.

picture the chat as 20 pages. page 1 is the setup claude gets before you say anything (your instructions file, its tools, which model you picked). the rest are messages and replies.

- **adding to the end is cheap.** you send page 20. pages 1 to 19 are identical to last time, so the saved notes cover them. only page 20 is new.
- **changing something earlier is expensive.** you change the setup on page 1 (switch models, edit your instructions file, add a tool). the saved notes only work if everything before a page is unchanged, so none of them can be reused. all 20 pages are re-read at full price and new notes are saved.

## part 2: what makes it cost so much

in my case, i processed about 2 billion tokens in a single month. out of every 100 tokens, about 98 were re-reading old chat, about 2 were saving notes, and fewer than 1 was a reply. my longest chat ran 1,141 messages, re-reading about 200k tokens per message.

replies cost more per token, but there are so few of them that they barely count. the model was not the main driver either: re-reading costs about the same on opus and sonnet at current prices. the real driver was chat length.

### common pitfalls

- **never clearing:** one chat for many tasks, so every message re-sends all of it.
- **big things in the chat:** a large file or log stays and is re-read every message.
- **many small follow ups:** each one re-sends the whole binder.
- **idle gaps:** over 5 minutes of silence rebuilds the notes at full price.
- **changing model or tools mid chat:** the notes do not carry over.

## part 3: the solution

**keep the context window as lean as possible.**

### 5 core rules

1. **small chat.** one task per chat. when it is done, start fresh. *why: every old message is re-read each time you send a new one.*
2. **fewer messages.** decide what you want, then batch your questions into one message instead of sending them one at a time. *why: each message re-sends the whole chat.*
3. **right model.** sonnet by default, opus for hard reasoning. *why: opus costs about twice as much for new text and replies.*
4. **delegate.** keep the main chat for planning. send big jobs to a subagent: a helper chat with its own fresh binder that returns only a short summary. give it a clear brief: its goal, the files, what to return, and a short handoff when done. *why: the big read never enters your main chat.*
5. **keep the notes warm.** no model or tool changes mid chat, and no long breaks mid task. *why: a change early on makes claude re-read everything at full price.*

### the 4 commands (claude code)

| command | what it does |
|---|---|
| `/clear` | wipes the chat and starts fresh with an empty binder. claude forgets everything, so paste back anything it still needs. |
| `/compact` | squeezes the chat into a short summary and keeps going in the same conversation. a smaller binder, same thread. |
| `/context` | shows what is filling the context window (messages, files, tools, instructions) so you can see what to cut. |
| `/cost` | shows what the current session has cost so far. |

### when this happens, do this

| situation | action |
|---|---|
| task done, or next task is unrelated | `/clear` |
| chat is getting long (over 10 to 15% of the context window used) | `/compact`, or `/clear` and paste a short summary |
| you want a different model | `/clear` first |
| big file or research | search or read only the lines you need, give a file path instead of pasting, or use a subagent |
| not sure what is using space | `/context` shows the binder, `/cost` shows the bill |

### write short and specific

each reply token costs about 5x a read token, and your wording sets how long replies run. the bigger cost is still the re-reading, so shorter chats come first, but short answers help too. put instructions in a short list, then the material under a label:

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
- **batch:** number several questions in one message.

### keeping long term history cheaply

keep your history in a filing cabinet of notes, not in the backpack you carry every message.

claude re-reads the whole chat every time you send a message, so a long chat gets expensive. think of the chat as a backpack you carry all day: the more papers inside, the heavier every trip. so don't keep your history in the backpack. keep it in a filing cabinet, which is a folder of plain text notes (obsidian is a nice way to browse them, but claude just reads the files). when you need something old, claude pulls out just that one paper and puts it back. the backpack stays light and nothing is lost.

1. **keep a short index:** one file listing what each note covers.
2. **look up on demand:** search for the topic and open only that note, never "read all my notes".
3. **write results back:** save the conclusion to a note, then `/clear`.

what quietly cancels the savings, like stuffing the cabinet back into the backpack:

- **files loaded on every chat:** an instructions file or memory index is read at the start of every chat, so a bloated one is paid for on every message. keep these short.
- **loading too much up front:** starting every chat by loading all your history brings the big binder back.
- **notes that only grow:** a log that gets longer every week costs more each time it is opened. shrink finished entries to their conclusion.

for the full setup, see my obsidian guide: [second-brain-guide](https://github.com/JohnSang16/second-brain-guide), a student-specific guide to pairing obsidian and claude code into an agentic second brain.
