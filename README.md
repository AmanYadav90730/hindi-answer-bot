# English to Hindi Answer Generator

Small LangGraph project — takes a question, answers it in English using Gemini, then translates the answer to Hindi. Made this to practice subgraphs in LangGraph, specifically how to call a graph inside another graph without the states getting mixed up.

## What's happening here

There's a parent graph and a subgraph.

Parent graph does two things: generates an English answer, then calls the subgraph to translate it.

Subgraph is just one node — takes text, sends it to Gemini with a translation prompt, returns Hindi text. That's it.

```
START -> generate_answer -> translate_answer -> END
```

`translate_answer` internally calls the subgraph, which looks like this:

```
START -> translate_node -> END
```

Kept them separate on purpose. Could've just written the translation logic inside `translate_answer` directly, but then it's stuck there. As a subgraph, I can reuse it anywhere — like later if I want to add Hindi output to my RAG assistant, I just import this subgraph instead of copy-pasting the same translate function again.

## Running it

```bash
pip install langchain-google-genai langgraph python-dotenv
```

Need a `.env` file with:

```
GOOGLE_API_KEY=your_key_here
```

Then:

```bash
python translate_subgraph.py
```

You'll get something like:

```
English Answer:
An LLM is an AI model trained on large amounts of text data...

Hindi Answer:
एलएलएम एक एआई मॉडल है जो बड़ी मात्रा में टेक्स्ट डेटा पर प्रशिक्षित है...
```

## Bugs I hit

Spent a bit debugging why the English answer was showing up as literally the prompt text instead of an actual answer — turns out I forgot to call `.invoke()` on the LLM in `generate_answer`, was just returning the f-string. Classic.

Also Gemini kept adding little explanations when translating ("this means..." type stuff) even though I didn't ask for that, so had to be explicit in the prompt: no extra content, just translate.

`draw_mermaid_png()` for visualizing the graph only works in notebooks, not plain scripts, so left that part commented out.

## Todo / ideas

- support other languages, not just Hindi
- maybe stream the English answer while Hindi is still being generated
- plug this into the main RAG assistant so answers can be translated on the fly
