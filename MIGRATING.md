# Migrating

## v1 to v2

- `tool__file_stats` is renamed `tool__file_info`.
- The `path` input of `tool__file_info` and `plan__describe_file` is renamed `file`. Output fields are unchanged.

`graph pack update example --major` runs `plan__migrate_v2` for you: it shows each affected file's diff, asks, and then writes the changes. To run it yourself:

```bash
graph plan run example__plan__migrate_v2 --input pack=example --input root=.graph
graph plan run example__plan__migrate_v2 --input pack=example --input root=.graph --input apply=true
graph config check
```

The first command only returns diffs. Use `root=~/.config/graph` for global files, and the name you installed the pack under for `pack`.

By hand: replace `example__tool__file_stats` with `example__tool__file_info` everywhere, and in steps calling either tool or `example__plan__describe_file`, rename the input key `path` to `file`.
