# Maxson-dev fork

Based on upstream Takopi v0.23.4 (`e0123d4`). The package version is
`0.23.4+maxson.1`. `origin` is this fork; `upstream` is `banteg/takopi`.

This fork adds three optional Telegram settings, with unchanged upstream defaults:

```toml
[transports.telegram]
# Existing bot_token, chat_id and allowed_user_ids stay here.
startup_thread_id = 20
general_trigger_mode = "mentions"
only_configured_chat = true

[transports.telegram.topics]
enabled = true
```

`startup_thread_id` sends the startup announcement to that topic in the main
chat. `general_trigger_mode` overrides General (no thread ID or ID 1) only in
the main chat with topics enabled. Other topics use the existing chat/topic
trigger state. To auto-reply in them, clear the old chat-level `mentions` and
any per-topic `mentions` overrides. `/trigger` reports the effective mode.
`only_configured_chat` restricts polling and update routing to `chat_id`. With
`allowed_user_ids = []`, every member of that one group can use the bot, while
DMs and other chats are ignored.
No chat IDs, tokens or deployment-specific topic IDs are embedded in code.

## Development and deployment

1. Work in a branch; run the existing CI checks (`uv sync --frozen --group docs`,
   `uv run ruff check src tests`, `uv run ruff format --check src tests`,
   `uv run ty check src tests`, `uv run pytest`, `uv build`, and the docs build).
2. Push and verify CI. Deploy the reviewed, tested full commit SHA, not a moving
   branch. The fork's CI Telegram notifications and upstream PyPI publication
   are disabled outside the upstream repository.
3. Check that no tasks are running. Stop the service and back up its config,
   state, and installed tool environment before replacing the installation.
4. From a checkout at the chosen SHA, install as the service user:

   ```sh
   uv export --frozen --no-dev --no-emit-project --no-hashes -o /tmp/takopi-constraints.txt
   uv tool install --force --python 3.14 --constraint /tmp/takopi-constraints.txt \
     'git+https://github.com/Maxson-dev/takopi.git@FULL_COMMIT_SHA'
   takopi --version
   takopi doctor
   ```

5. Start the service; verify startup destination, trigger modes, and a Codex
   runner response. Config and session state remain outside the installed
   package. Record the deployed SHA in the deployment runbook. Roll back to the
   previous SHA with its matching constraints and config if checks fail.

The service manager still owns restarts and child process cleanup. Do not edit
installed `site-packages`: all changes belong in this repository. Updates are
operator-controlled; no automatic deploy-on-push is configured.

This fork retains upstream's approval behavior: Codex app-server permission
requests are accepted automatically. It does not add a Telegram approval gate,
passive discussion history, or reliable replay of tasks interrupted by restart.
