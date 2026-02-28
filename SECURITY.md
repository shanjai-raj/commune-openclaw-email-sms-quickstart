# Security Policy

## Reporting

Please do not open public GitHub issues for security vulnerabilities. Email **security@commune.email** instead.

## Scope

- API key handling in skill files and CLI helpers
- Injection risks in email/SMS content processing

## Best Practices

All skill files and examples follow these patterns:
- API keys via environment variables only
- No credentials ever hardcoded
