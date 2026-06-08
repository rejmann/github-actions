# Ping

Arquivo: `.github/workflows/ping.yaml`

## Objetivo

Fornecer um workflow reutilizável mínimo para validar integração via `workflow_call`.

Comportamento:

1. Executa um job `ping` em `ubuntu-latest`.
2. Imprime `pong` no log.

## Tipo

Workflow reutilizável via `workflow_call`.

## Saídas

Este workflow não expõe saídas.

## Permissões e requisitos

- Não requer permissões especiais.
- Não depende de checkout do repositório.

## Exemplo de uso

```yaml
name: Ping Test

on:
  workflow_dispatch:

jobs:
  ping:
    uses: rejmann/github-actions/.github/workflows/ping.yaml@main
```
