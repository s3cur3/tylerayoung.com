---
title: "Fixing Full Recompiles of Elixir Caused by Claude Code"
layout: post
authors: ['tyler']
categories: ["Programming", "Functional Programming", "Elixir"]
excerpt_full: true
---

At work, many of us were having a persistent issue where our Elixir app was having to do a full recompile very frequently, despite not having touched the dependencies, `mix.exs`, and so on. We were finally able to narrow it down to affecting the people who use Claude Code and Mise together (it may affect ASDF users as well?).

The crux of the issue in our case was that Claude's Bash tool (and it always uses Bash, regardless of your default shell!) doesn't inherit the Mise configuration. I don't know why it doesn't (maybe you do, and you should tell me about it!), because we _are_ configuring Mise in `~/.bash_profile`, but the fix for us was to have Claude explicitly set up Mise and ASDF at the start of each session using a "hook" in our `.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/scripts/activate-env.sh\""
          }
        ]
      }
    ]
  }
}
```

That script it references is pretty simple:

```bash
#!/usr/bin/env bash

# Activate the tool version manager (mise or asdf) so that Claude Code
# uses the correct versions of Elixir, Erlang, Node.js, etc.

if command -v mise &> /dev/null; then
    eval "$(mise activate bash)"
elif [ -f "$HOME/.asdf/asdf.sh" ]; then
    . "$HOME/.asdf/asdf.sh"
fi

# Persist the updated PATH (and any other env vars) for Claude Code's
# subsequent Bash tool calls.
if [ -n "$CLAUDE_ENV_FILE" ]; then
    echo "PATH=\"$PATH\"" >> "$CLAUDE_ENV_FILE"
fi
```

With that in place, Claude now gets the correct version of Elixir and fast, incremental compiles.
