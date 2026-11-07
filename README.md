# 小葵（Aoi） Grammar Cards — ChatGPT plugin

A ChatGPT plugin that notices meaningful English grammar mistakes in a
conversation, saves them as cards on the learner's 小葵（Aoi） account, and shows
the correction in the reply. A corrected sentence holds all of its grammar
cards; cards are grouped by grammar pattern, so one pattern can be reviewed
across every sentence where it came up.

The plugin has no server of its own: ChatGPT connects over MCP to the 小葵（Aoi）
backend at `https://aoilang.com`, and signs each user in with OAuth 2.1.

```
ChatGPT ──MCP Streamable HTTP + OAuth 2.1──▶ https://aoilang.com/grammar/mcp
                                              ├── grammar.CorrectedType (shared patterns)
                                              ├── grammar.CorrectedSentence (per-user source sentence)
                                              └── grammar.SentenceCard (grammar issues inside a sentence)
```

## Install in ChatGPT

1. The backend must be deployed at `https://aoilang.com` with the grammar
   migration applied (see [Backend requirements](#backend-requirements)).
2. In ChatGPT, open **Plugins → Add custom MCP server** (developer mode).
3. Enter `https://aoilang.com/grammar/mcp`.
4. Choose **Create as plugin**, then connect the account with **OAuth**.
5. Install the skill for the plugin (see [Skill](#skill)) if your host does not
   load `skills/` automatically.

The first tool call starts the OAuth flow. The consent screen lists the
requested scopes:

| scope | grants |
| --- | --- |
| `grammar:read` | list/search corrected sentences with cards, read one sentence, read the profile |
| `grammar:write` | create corrected sentences with cards (batch), delete a sentence |

## Install in Codex / ChatGPT desktop

The plugin directory is a standard Agent Plugins package:

- [`plugin.json`](plugin.json) carries the package name, version and the
  ChatGPT-facing interface metadata (display name, description, starter
  prompts, icon) alongside `skills/`.
- [`mcp.json`](mcp.json) and [`mcp.remote.json`](mcp.remote.json) point at the
  remote MCP server and request OAuth.

Point Codex at this folder (plugin directory install), or open the URL directly
in a custom MCP connection.

## Tools

| tool | scope | purpose |
| --- | --- | --- |
| `create_english_learning_cards` | write | save 1–50 corrected sentences with nested cards in one atomic call |
| `delete_english_learning_card` | write | delete a corrected sentence and its cards |
| `list_english_learning_cards` | read | corrected sentences with nested cards, filtered by `query` or `corrected_type_id` |
| `show_english_learning_cards` | read | the interactive English Cards panel (conversation, thread panel, sidebar) |
| `get_english_learning_card` | read | one corrected sentence and its cards by id |
| `get_profile` | — | the linked 小葵（Aoi） account and avatar (`openai/profile` tool) |

A corrected sentence carries the full source sentence, with removed words in
`~~strikethrough~~` and replacements in `**bold**`, and owns one or more grammar
cards. Cards with the same `title` share one corrected type (types are global;
ownership stays on the sentences and cards). Re-submitting
the same corrected sentence updates it and upserts its cards instead of
duplicating them. Independent sentences are sent as separate sentence entries.

## Card UI

`show_english_learning_cards` is an MCP App tool. Its UI resource renders the
grouped cards in the conversation, and the same tool registers both OpenAI
entrypoints:

- `thread` — the panel beside a conversation.
- `global` — the ChatGPT sidebar app.

The panel shows one combined corrected sentence per group, followed by the
grammar cards (title, rule, example). It has its own search and a
Refresh action. The skill calls it after saving cards so the learner sees the
correction immediately.

The UI is served by the MCP server (`resources/read`, `text/html;profile=mcp-app`),
so it only appears after the plugin is connected and its tools are scanned.
After a server update, refresh/rescan the MCP connection in ChatGPT so the new
tool metadata and UI resource are picked up.

## Skill

[`skills/aoi-grammar-cards/SKILL.md`](skills/aoi-grammar-cards/SKILL.md) tells the
model to call `create_english_learning_cards` before starting the user's task,
how to group issues by source sentence, and how to show the combined correction
in the reply.

### Making the tools fire on every message

The model decides when to call a tool, so no server change can force it down to
"every message". Three levers exist, weakest to strongest:

1. **Tool metadata.** The tool description and the server `instructions` both ask
   for a save before answering, and the app ships three starter prompts
   (`defaultPrompt` in `plugin.json`).
2. **The skill.** Applies when the host loads `skills/` automatically.
3. **Custom Instructions.** Always in context, so this is the reliable override
   when the skill is not picked up. Paste one line under
   **Settings → Personalization → Custom instructions**:

   ```text
   Before you answer, save the English grammar corrections from my message with
   create_english_learning_cards, then continue with my request.
   ```

The save is cheap and idempotent — re-sending the same sentence updates it — and
it renders the card panel itself, so asking for it on every message costs one
tool call, not two.

## Backend requirements

Deploy the `souka` Django project from this workspace at `https://aoilang.com`
behind HTTPS and:

1. Apply the grammar migration:

   ```bash
   ./manage.py migrate grammar
   ```

2. Set the public origin (defaults to `https://aoilang.com`):

   ```
   OAUTH_PUBLIC_ORIGIN=https://aoilang.com
   ```

3. Make sure these endpoints are reachable (they are part of the app):

   | endpoint | purpose |
   | --- | --- |
   | `POST /grammar/mcp` | MCP Streamable HTTP server |
   | `GET /.well-known/oauth-protected-resource` | resource metadata (RFC 9728) |
   | `GET /.well-known/oauth-authorization-server` | authorization server metadata (RFC 8414) |
   | `/accounts/oauth/authorize` | consent screen (login required) |
   | `POST /accounts/oauth/token` | code + refresh token exchange |
   | `POST /accounts/oauth/register` | dynamic client registration (RFC 7591) |

4. For local testing, expose the dev server over HTTPS with a tunnel and start
   Django with `OAUTH_PUBLIC_ORIGIN=https://<tunnel host>` so the discovery
   documents and the `WWW-Authenticate` challenge use the tunnel origin.

### nginx

The `/.well-known/oauth-*` documents are Django views, not files. A block such
as `location /.well-known { try_files $uri $uri/ =404; }` resolves them on disk
and returns `404`, which stops ChatGPT from discovering OAuth.

Keep the static files that live under `/.well-known/` and fall back to Django
when no file matches:

```nginx
# certbot webroot (adjust to your setup)
location ^~ /.well-known/acme-challenge/ {
    root /var/www/certbot;
}

# Serve existing files first, proxy everything else (OAuth discovery, etc.)
location ^~ /.well-known/ {
    # The directory that contains your .well-known folder. Omit if the
    # server block already sets the right root.
    root /var/www/html;
    try_files $uri @aoi_app;
}

location @aoi_app {
    include uwsgi_params;
    proxy_pass http://192.168.0.105:8083;
}
```

Notes:

- Do not add `$uri/` to `try_files`; a directory match would stop the fallback.
- `proxy_pass` has no trailing slash in the named location, so Django receives
  the original path.
- If nginx talks to uWSGI directly, use `uwsgi_pass 127.0.0.1:8084;` instead of
  `proxy_pass` in the named location (keep `include uwsgi_params;`).
- Add exact `location = ...` blocks before the generic one if a file needs a
  special Content-Type, for example `apple-app-site-association`.

Verify with:

```bash
curl -i https://aoilang.com/.well-known/oauth-protected-resource
curl -i https://aoilang.com/.well-known/oauth-authorization-server
```

Both must return `200` with JSON; a `404` means nginx is handling them.

### OAuth details for operators

- Unauthenticated MCP requests return `401` with
  `WWW-Authenticate: Bearer resource_metadata="https://aoilang.com/.well-known/oauth-protected-resource", scope="grammar:read grammar:write"`.
- Authorization code + PKCE (`S256`) only; codes live 5 minutes and are single
  use.
- Access tokens are opaque `aoi_at_…` strings, valid for 1 hour. They are stored
  as SHA-256 hashes, never in plain text.
- Refresh tokens are `aoi_rt_…` strings valid for 30 days and rotate on every
  use.
- Tokens are bound to the MCP resource (`resource=https://aoilang.com/grammar/mcp`);
  a token minted for another resource is rejected.
- The plugin declares `auth.client.mode: "dcr"` in `mcp.json`; the backend
  performs dynamic client registration at `/accounts/oauth/register` (RFC 7591)
  and requests both grammar scopes.
- Sessions/scopes are per user; all tools filter by `CorrectedSentence.user`.

### Revoking access

The user can disconnect the plugin in ChatGPT. From the backend, deleting the
user's `accounts_oauthtoken` rows (or the `OAuthClient` created by the connection)
invalidates the tokens immediately; access tokens also expire after an hour.

## Limits

- This first version is tool-only: ChatGPT shows the saved card through the
  reply text and tool results. A visual MCP App deck (sidebar/thread panel like
  the local English Learning Card plugin) can be added later without changing
  the storage model.
- ChatGPT custom connectors require public HTTPS; `localhost` URLs do not work
  in ChatGPT web.

## Development

The backend lives in this workspace at `souka/grammar/` (cards and MCP) and `souka/accounts/` (OAuth). Tests:

```bash
cd souka
./dev.sh test grammar
```

### Checking the card UI locally

ChatGPT shows the panel only after the view completes the `ui/initialize` →
`ui/notifications/initialized` handshake; when that never arrives it replaces
the panel with "The app didn't finish initializing within 5 seconds". This check
renders the view in a sandboxed iframe joined to a strict MCP Apps host and
fails when the handshake does not complete:

```bash
cd souka
python3 scripts/check_mcp_app.py                  # view in the working tree
python3 scripts/check_mcp_app.py --drop-first 2   # also exercises the retry
```

To check what a running server actually serves — the HTML a deploy ships — mint
a token against the local origin:

```bash
# terminal 1
OAUTH_PUBLIC_ORIGIN=http://127.0.0.1:8084 ./dev.sh runserver

# terminal 2
./dev.sh manage shell -c "
from accounts import oauth
from accounts.models import OAuthClient
from django.contrib.auth import get_user_model
client, _ = OAuthClient.register('local-check', ['http://localhost'])
token, access, _ = oauth.issue_tokens(
    client, get_user_model().objects.first(),
    ['grammar:read', 'grammar:write'], oauth.resource())
print(access)
"

python3 scripts/check_mcp_app.py \
  --server http://127.0.0.1:8084/grammar/mcp --token aoi_at_...
```

Add `--keep` to keep the generated files and open `host.html` in a normal
browser to click through the panel. ChatGPT itself only renders the panel from
public HTTPS, so the last mile stays a tunnel plus a connector rescan.

## License

MIT — see [LICENSE](LICENSE).
