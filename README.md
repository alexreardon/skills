# skills

Alex Reardon's collection of [agent skills](https://vercel.com/docs/agent-resources/skills).

## Install

```bash
npx skills add alexreardon/skills
```

To install a single skill from this repo:

```bash
npx skills add alexreardon/skills --skill cook-me
```

## Skills

### [`/cook-me`](skills/cook-me/SKILL.md) 🧑‍🍳

Opinionated version of the [`/grill-me` skill](https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md) by [Matt Pocock](https://x.com/mattpocockuk).

The big idea is to make it easier to work with `/grill-me` by encouraging the model to use consistent patterns for question asking, language and layout.

The opinions:

- 1️⃣ Only one question per turn. Never ask multiple questions at a time
- 🙌 Questions and options are phrased in positive language so it's easy to say "yes" to things (no double negatives)
- ⬇️ Every question orders recommendations in order from most to least recommended paths forward
- 💅 Leveraging consistent markdown formatting to make it easy to predictably parse outputs (super helpful when you have to answer lots of questions!)
- 🧘 What you need to do for every turn should be obvious (answer a question, provide more details, etc)