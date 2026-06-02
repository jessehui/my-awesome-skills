# verify-claims

A Claude Code skill that automatically verifies factual claims, causal inferences, and technical advice in LLM output — inline-annotated with evidence.

## What it does

When Claude's output contains critical assertions (version numbers, API behavior, compatibility matrices, technical recommendations), this skill triggers a verification workflow:

1. **Identify** key claims that downstream conclusions depend on
2. **Verify** against local project files, documentation, or web search (via anysearch)
3. **Annotate** inline with verdict and source

### Annotation format

```
PyTorch 2.6 supports CUDA 12.4 [✅已验证: official compatibility matrix]

HugePages reduce TLB misses and improve throughput [⚠️部分验证: TLB miss reduction confirmed, throughput gain varies by workload]

Set NCCL_ALGO=Ring for best performance [❗未验证: official docs do not recommend pinning algorithm, auto-selection is preferred]
```

### Trigger

- **Auto**: when output contains specific version numbers, dates, API behavior, or executable technical advice
- **Manual**: user says "验证一下", "verify", or "check this"

## Installation

Add the marketplace and install:

```bash
/plugin marketplace add jessehui/verify-claim
/plugin install verify-claims@verify-claim
```

## Dependencies

This skill uses the [anysearch](https://github.com/Claude-Plus-Marketplace/anysearch-skill) skill for web verification when local sources are insufficient. Install it for full functionality:

```bash
/plugin marketplace add Claude-Plus-Marketplace/anysearch-skill
/plugin install anysearch@anysearch-skill
```

## License

MIT
