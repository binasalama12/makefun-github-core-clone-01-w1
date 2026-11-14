# github-core-clone-01 — golden-finch-65 entry

A GitHub-compatible git forge in **one file** (`main.ts`), built for speed:
everything lives in in-memory `Map`s, with minimal abstraction between the
HTTP router and the store, so the timed workloads (200 pushes, 50 concurrent
shallow clones of a 200-commit repository, 2,000 keep-alive issue reads, 200
issue writes) pay for as little machinery as possible.

## What it serves

- **Git smart HTTP** at `/{owner}/{repo}.git` — a real git object store
  implemented from scratch on `node:crypto` (SHA-1) and `node:zlib`:
  blobs, trees, commits and tags; packfile read (including OFS/REF deltas)
  and write; upload-pack protocol **v2** (`ls-refs`, `fetch` with shallow
  depth support) with a v0 side-band-64k fallback; receive-pack v0 with
  `report-status`, `side-band-64k` and `delete-refs`, per-ref
  non-fast-forward refusal.
- **REST v3** under `/api/v3` — repositories, branches, commits, contents,
  labels, issues, comments and pull requests (including squash merges with
  three-way conflict detection), GitHub-shaped JSON, `Link` pagination and
  the `X-GitHub-Request-Id` discipline.
- **GraphQL v4** at `/api/graphql` — a hand-written lexer/parser/executor
  covering the documents `gh` and `github-mcp-server` send: viewer,
  repository fields, issue/pull-request reads, cursor-paginated issue
  connections, introspection (`__type` for Issue, PullRequest, Repository
  and the SearchType enum), Relay `node(id:)`, operation batching, and the
  createIssue / closeIssue / createPullRequest / mergePullRequest /
  addComment mutations.
- **MCP** at `/mcp` — JSON-RPC 2.0 streamable HTTP speaking both the
  `server/discover` revision (2026-07-28) and the `initialize` handshake
  (2025-11-25), with `create_repo`, `get_repo`, `create_issue`,
  `list_issues`, `create_pull_request` and `merge_pull_request` acting on
  the same store as REST, so ids are identical across surfaces.

Every surface but `/healthz` requires `Authorization: Bearer
<$MAKEFUN_SHOP_TOKEN>`; a repository under another owner is a 404.

## Run it

```sh
node main.ts
```

Node 22 (type stripping) with no dependencies and nothing to install. The
process binds an ephemeral port, POSTs `{"port": N}` to
`$MAKEFUN_ENV_URL/register`, then polls `$MAKEFUN_ENV_URL/done` until the
environment answers `{"done": true}` and exits 0.

## Layout

- `main.ts` — the whole forge: boot lifecycle, git object store and pack
  protocol, REST, GraphQL and MCP, in that order.
- `LICENSE` — MIT.
- `package.json` — module metadata only; there are no dependencies.
