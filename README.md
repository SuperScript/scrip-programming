# scrip-programming

A [Claude Code plugin](https://docs.anthropic.com/en/docs/claude-code) that teaches Claude shell programming conventions for the [scrip](https://github.com/superscript/scrip) project.

## What It Does

When installed, this plugin provides Claude with knowledge of scrip's programming patterns:

- **Error handling vocabulary** -- `shout`, `barf`, `usage`, `safe`, `catch`
- **Module composition** via `#include`
- **Subcommand routing** with `do_` prefix functions
- **Help extraction** using `#_#` comment markers
- **Output-based testing** with diff against expected output

The skill activates when Claude is asked to create or modify shell scripts written in scrip style. It does not activate for general Bash scripting.

## Installation

Add this plugin to your Claude Code configuration:

```sh
claude plugin add superscript/scrip-programming
```

## License

BSD 3-Clause. See [LICENSE](LICENSE).
