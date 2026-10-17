# Build

Arquivo: `.github/workflows/golang-build.yml`

## Objetivo

Compilar o binário Go `pvm` para múltiplas plataformas e publicar cada build como artifact empacotado.

Comportamento:

1. O job `prepare-matrix` monta a matriz de build conforme o input `platform`.
2. O job `build` roda uma vez por combinação `goos`/`goarch` da matriz:
   1. Faz checkout do ref informado em `version` (ou do ref padrão, se vazio).
   2. Instala o Go na versão definida em `go.mod`.
   3. Compila com `-ldflags "-s -w -X main.version=<versão>"`.
   4. Empacota o binário (`.tar.gz` para Linux/macOS, `.zip` para Windows).
   5. Publica o pacote como artifact `pvm-<goos>-<goarch>`.

## Tipo

Workflow reutilizável via `workflow_call`.

## Entradas

| Nome       | Tipo   | Obrigatório | Padrão | Descrição |
|------------|--------|-------------|--------|-----------|
| `version`  | string | não         | `""`   | Versão embutida no binário (`main.version`). Quando informada, também é o ref usado no checkout (ex.: `v1.0.2`). Se vazia, o binário recebe `dev`. |
| `platform` | string | não         | `all`  | Plataforma alvo: `all`, `linux`, `darwin` ou `windows`. Qualquer valor não reconhecido é tratado como `all`. |

## Plataformas

| Plataforma | Arquiteturas      | Binário   | Pacote    |
|------------|-------------------|-----------|-----------|
| `linux`    | `amd64`, `arm64`  | `pvm`     | `.tar.gz` |
| `darwin`   | `amd64`, `arm64`  | `pvm`     | `.tar.gz` |
| `windows`  | `amd64`           | `pvm.exe` | `.zip`    |

## Saídas

Este workflow não expõe outputs. Os resultados são publicados como artifacts:

- `pvm-<goos>-<goarch>`: contém `pvm-<goos>-<goarch>.tar.gz` ou `.zip` (ex.: `pvm-linux-amd64`).

## Permissões e requisitos

- Permissão `contents: read`.
- O repositório chamador precisa ter um `go.mod` na raiz e um pacote `main` na raiz com a variável `version` (`var version string`).
- Se `version` for informada, o ref (tag/branch) precisa existir no repositório chamador.

## Exemplo de uso

Build de todas as plataformas usando a versão resolvida por `version-tag.yml`:

```yaml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write

jobs:
  resolve-version:
    uses: rejmann/github-actions/.github/workflows/version-tag.yml@main

  build:
    needs: resolve-version
    uses: rejmann/github-actions/.github/workflows/golang-build.yml@main
    with:
      version: ${{ needs.resolve-version.outputs.version }}
```

Build apenas para Linux, sem versão (binário `dev`):

```yaml
jobs:
  build:
    uses: rejmann/github-actions/.github/workflows/golang-build.yml@main
    with:
      platform: linux
```

## Notas técnicas

- A matriz é gerada dinamicamente em JSON e consumida com `fromJson`.
- Todos os builds rodam em `ubuntu-latest` via cross-compilation (`GOOS`/`GOARCH`).
- O nome do binário (`pvm`) está fixo no workflow; para outros projetos é preciso ajustá-lo.
