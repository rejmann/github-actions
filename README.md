# GitHub Actions Reutilizáveis

Este repositório centraliza workflows reutilizáveis para uso em outros projetos.

## Estrutura

- `.github/workflows/`: definição dos workflows reutilizáveis.
- `docs/workflows/`: documentação detalhada de cada workflow.

## Workflows disponíveis

- `Resolve Version Tag` ([docs/workflows/version-tag.md](docs/workflows/version-tag.md))

## Como usar

Em outro repositório, chame o workflow com `uses`:

```yaml
jobs:
  version:
    uses: rejmann/github-actions/.github/workflows/version-tag.yml@main
```

## Saídas dos workflows

Cada workflow documenta suas saídas em `docs/workflows`.
No caso de `version-tag.yml`, a saída principal é:

- `version`: tag resolvida/criada (exemplo: `v1.0.1`).

## Observações

- Os workflows deste repositório são pensados para serem chamados via `workflow_call`.
- Recomenda-se versionar por tag em vez de usar `@main` em produção.
