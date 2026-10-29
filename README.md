# AI Research Assistant

A command-line research assistant built on Google Gemini with function calling. You chat with it in the terminal, and Gemini decides when to use one of four Python tools:

| Tool | What it does |
|---|---|
| `web_research` | Searches the web with Google Search (and optionally reads a URL), then returns a 3-bullet summary |
| `save_note` | Appends a note to a local text file |
| `analyze_sentiment` | Labels text as Positive, Negative or Neutral using a simple keyword count |
| `generate_concise_summary` | Shortens text to a maximum number of words |

Gemini only chooses which tool to call and with what arguments. The script runs the Python function, sends the result back to Gemini, and Gemini writes the final answer from it.

## Requirements

- Python 3.10 or newer
- A Gemini API key, free from [Google AI Studio](https://aistudio.google.com/apikey)

## Setup

1. Clone the repository and go into the folder:

   ```bash
   git clone https://github.com/<your-username>/<repo-name>.git
   cd <repo-name>
   ```

2. Create and activate a virtual environment:

   Windows (PowerShell):

   ```powershell
   python -m venv .venv
   .venv\Scripts\Activate.ps1
   ```

   If PowerShell refuses to run the activation script, run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` once and try again.

   macOS / Linux:

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

3. Install the dependencies:

   ```bash
   pip install -r requirements.txt
   ```

4. Create a file named `.env` in the project folder with your API key:

   ```
   GEMINI_API_KEY=your-api-key-here
   ```

   The line must include `GEMINI_API_KEY=`; the key on its own will not be found. `.env` is listed in `.gitignore`, so your key is not committed.

   Optionally, choose a different model on a second line (the default is `gemini-3-flash-preview`):

   ```
   GEMINI_MODEL=gemini-3-pro-preview
   ```

## Usage

With the virtual environment active:

```bash
python main.py
```

```
AI Research Assistant ready.
Type 'exit' to quit.

You: What are the latest developments in AI agents?
```

Things to try:

- `Research the latest news about renewable energy`
- `Summarize https://example.com/article in a few bullets`
- `Analyze the sentiment of: I love this course but the quizzes are hard`
- `Save a note saying "check sources tomorrow" to notes.txt`

Type `exit` or `quit` to stop.

## How it works

Each message goes through two calls to Gemini:

1. The conversation history is sent with the four function declarations. Gemini either answers directly or asks for one or more functions.
2. The script runs the requested functions, adds their results to the history with the `tool` role, and calls Gemini again (with tools turned off) to write the final answer.

The function declarations (schemas) are what Gemini sees; it never reads the Python code. `web_research` makes its own separate Gemini call with the built-in `google_search` and `url_context` tools, because those are not sent in the same request as the custom function declarations.

## Limitations

- `analyze_sentiment` and `generate_concise_summary` are simple rules, not AI. The sentiment check matches keywords anywhere in the text, and the summary cuts the text off after a word count.
- Only one round of tool calls per message, so the assistant cannot research a topic and then save the findings in the same turn.
- `save_note` writes to whatever filename Gemini chooses, relative to the folder you run the script from.
- The full conversation is resent on every call, so very long sessions use more tokens.

## Troubleshooting

| Error | Fix |
|---|---|
| `ModuleNotFoundError: No module named 'dotenv'` | The virtual environment isn't active, or the dependencies aren't installed in it. Activate it and run `pip install -r requirements.txt`. In VS Code, use **Python: Select Interpreter** and pick the one in `.venv`. |
| `RuntimeError: GEMINI_API_KEY is not set` | `.env` is missing, is in a different folder, or doesn't start the line with `GEMINI_API_KEY=`. |
| `Error calling the model: ... 400` or `403` | The API key is invalid or the model name isn't available to your key. Check the key in Google AI Studio, or set `GEMINI_MODEL` to a model you have access to. |
