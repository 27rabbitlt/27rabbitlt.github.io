# Linux Kernel Mailing List & Patch Review Guide

Notes from studying the lore.kernel.org thread:
`https://lore.kernel.org/linux-staging/20260317120114.39057-1-dihed1973@gmail.com/T/#u`

---

## How to Read lore.kernel.org

lore.kernel.org is the public archive for Linux kernel mailing lists. URL fragments control the view:

| Fragment | View | Description |
|----------|------|-------------|
| `#u` | Flat/unstripped | All messages in chronological order, fully expanded |
| `#t` | Threaded | Messages indented to show reply structure |
| `#m` | Message | Jumps to the specific message in the URL |
| `#r` | Related | Shows related threads/patches |

Each message has links: `[permalink]`, `[raw]`, `[flat]`, `[nested]`, `[reply]`, and `mbox.gz` download.

## Why Multiple People Send Patches with the Same Subject

The subject line format is standardized: `[PATCH] subsystem: file: description`. Different people independently fixing the same checkpatch warning will produce the same subject. Each is an independent submission, not a reply.

Example from the studied thread — three people attempted the same block comment fix in `nvec.c`:

1. **Alexandru Hossu** — Rejected: sent patch as email attachment instead of inline, and no changelog text in the commit message.
2. **Oskar Ray-Frayssinet** — Rejected: snuck in undocumented changes (capitalization fixes) not mentioned in the changelog.
3. **Martin Bojorquez** — Clean version: only the described change, proper changelog.

## Anatomy of a Correct Patch Email

```
Subject: [PATCH] staging: vme_user: add blank line after declarations

Add a blank line between regular field declarations and function pointer
declarations in struct fake_driver to follow the kernel coding style
convention of separating declarations for improved readability.    <-- changelog text

This addresses the following checkpatch warning:                   <-- reference
  WARNING: Missing a blank line after declarations

Signed-off-by: Name <email>                                        <-- required
---
 drivers/staging/vme_user/vme_fake.c | 1 +                        <-- diffstat
 1 file changed, 1 insertion(+)

diff --git a/...                                                   <-- the actual diff
--
2.53.0                                                             <-- git version (metadata, ignored)
```

Key rules:
- **Changelog text is mandatory** — between the subject and `Signed-off-by`. Explains *what* and *why*.
- **Patch must be inline plain text** — not an email attachment.
- **Only do what you describe** — no undocumented changes.
- **One patch = one logical change** — each patch should compile cleanly on its own.

## Sending Patches as a Series (for larger changes)

Large refactors are split into a numbered series:

```
[PATCH 0/5] staging: vme_user: refactor driver initialization   <-- cover letter, no code
[PATCH 1/5] staging: vme_user: extract helper for X
[PATCH 2/5] staging: vme_user: rename Y to Z
[PATCH 3/5] staging: vme_user: move A into B
[PATCH 4/5] staging: vme_user: simplify C
[PATCH 5/5] staging: vme_user: update D
```

## How Maintainers Review Patches

Maintainers typically apply patches locally for review:

```bash
# Fetch a series from lore
b4 am <message-id>

# Or apply from mailbox
git am *.patch
```

Then review with their usual tools — most kernel devs use terminal-based workflows:
- **Editors:** Vim/Neovim, Emacs (with `magit`)
- **Diff viewing:** `git diff`, `git log -p`, `delta`, `vimdiff`, `tig`
- **Email clients:** `mutt` / `neomutt` — many review directly in email
- **GUI options:** `gitk`, `meld`, `kdiff3` (less common)

## Replying to Patches / Adding Review Comments

### Threading

The `In-Reply-To` email header links your reply into the thread. This is set automatically by your email client or `git send-email --in-reply-to`.

### Body format: interleaved reply with trimming

Only quote the lines you're commenting on. Delete everything else.

```
On Mon, 17 Mar 2026, Martin Bojorquez wrote:
> +	/* delay ACK due to AP20 HW Bug
> +	 * do not replace by usleep_range
> +	 */

Nit: could you also capitalize "delay" -> "Delay" to match
the rest of the comment style in this file?
```

Do NOT quote the entire email (the opposite of what Gmail does by default).

### Common review tags

| Tag | Meaning |
|-----|---------|
| `Reviewed-by: Name <email>` | You've reviewed it, looks correct |
| `Acked-by: Name <email>` | You approve (usually from a subsystem maintainer) |
| `Tested-by: Name <email>` | You've tested it and it works |
| `Nit:` | Minor style suggestion, not blocking |
| `nack` | You object to the patch |

For new contributors, `Tested-by` and general inline comments are most appropriate.

### Sending replies

- **`b4`** — `b4 send --reply-to <message-id>`
- **`git send-email`** — with `--in-reply-to='<message-id>'`
- **`mutt`/`neomutt`** — hit reply, trim quotes in nvim, send

### Tooling gap

There is no "GitHub PR review" equivalent for email-based patches. The review-and-comment workflow is: apply patch locally, review in editor with full context, then switch to email client and manually write the interleaved reply. This is a known pain point for new contributors.

## Requirements for Email Clients

- Plain text only (no HTML)
- No automatic line wrapping that mangles diffs
- Proper `In-Reply-To` headers for threading
- Gmail web UI is not recommended (wraps lines); use it via IMAP in `mutt` or configure `git send-email` with Gmail SMTP
