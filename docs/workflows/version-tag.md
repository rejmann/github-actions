# Resolve Version Tag

Arquivo: `.github/workflows/version-tag.yml`

## Objetivo

Resolver uma versão de release para o commit atual.

Comportamento:

1. Se já existir uma tag `v*` apontando para o `HEAD`, ela é reutilizada.
2. Se não existir, o workflow identifica a última tag `v*` e incrementa o patch.
3. Se não houver tags no repositório, inicia em `v1.0.0` e cria `v1.0.1`.

## Tipo

Workflow reutilizável via `workflow_call`.

## Saídas

- `version`: tag resolvida ou criada.

## Permissões e requisitos

- O repositório chamador precisa permitir push de tags pelo token utilizado.
- O checkout usa `fetch-depth: 0` para ler histórico e tags.

## Exemplo de uso

```yaml
name: Release

on:
  push:
    branches: [main]

jobs:
  resolve-version:
    uses: rejmann/github-actions/.github/workflows/version-tag.yml@main

  print-version:
    needs: resolve-version
    runs-on: ubuntu-latest
    steps:
      - name: Show version
        run: echo "Version: ${{ needs.resolve-version.outputs.version }}"
```

## Notas técnicas

- Busca tags com padrão `v*`.
- Ordena versão por semver com `--sort=-v:refname`.
- Cria tag anotada (`git tag -a`) e faz push para `origin`.
