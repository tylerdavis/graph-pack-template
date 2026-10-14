# example

Describe files on disk: stats, an excerpt, and a one-paragraph summary.

| Item | What it does |
|---|---|
| `example__tool__file_info` | size, line and word counts, MIME type and the first 4 KB of a file |
| `example__plan__describe_file` | a file's stats plus a one-paragraph summary |
| `example__agent__file_helper` | answers questions about files using both |

## Install

```bash
graph pack add github.com/<owner>/<repo> --path example
```

## Requirements

`bash`, `jq`, `wc`, `file` and `head` on `PATH`, on macOS or Linux.
