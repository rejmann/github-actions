# Build

Arquivo: `.github/workflows/golang-build.yml`

## Objetivo

Compilar um binário Go para múltiplas plataformas e publicar cada build como artifact empacotado.

Comportamento:

1. O job `prepare-matrix` monta a matriz de build conforme o input `platform`.
2. O job `build` roda uma vez por combinação `goos`/`goarch` da matriz:
   1. Faz checkout do ref informado em `version` (ou do ref padrão, se vazio).
   2. Instala o Go na versão definida em `go.mod`.
   3. Compila o pacote `package` com `-ldflags "-s -w -X main.version=<versão>"`.
   4. Empacota o binário (`.tar.gz` para Linux/macOS, `.zip` para Windows).
   5. Publica o pacote como artifact `<name>-<goos>-<goarch>`.

## Tipo

Workflow reutilizável via `workflow_call`.

## Entradas

| Nome       | Tipo   | Obrigatório | Padrão | Descrição |
|------------|--------|-------------|--------|-----------|
| `version`  | string | não         | `""`   | Versão embutida no binário (`main.version`). Quando informada, também é o ref usado no checkout (ex.: `v1.0.2`). Se vazia, o binário recebe `dev`. |
| `platform` | string | não         | `all`  | Plataformas alvo: `all` ou uma lista separada por vírgula de `linux`, `darwin` e `windows` (ex.: `linux,windows`). Valores não reconhecidos fazem o job falhar. |
| `name`     | string | não         | `pvm`  | Nome do binário e dos artifacts (no Windows recebe `.exe`). |
| `package`  | string | não         | `.`    | Pacote Go a compilar, relativo à raiz do repositório (ex.: `./cmd/ponto`). |

## Plataformas

| Plataforma | Arquiteturas      | Binário      | Pacote    |
|------------|-------------------|--------------|-----------|
| `linux`    | `amd64`, `arm64`  | `<name>`     | `.tar.gz` |
| `darwin`   | `amd64`, `arm64`  | `<name>`     | `.tar.gz` |
| `windows`  | `amd64`           | `<name>.exe` | `.zip`    |

## Saídas

Este workflow não expõe outputs. Os resultados são publicados como artifacts:

- `<name>-<goos>-<goarch>`: contém `<name>-<goos>-<goarch>.tar.gz` ou `.zip` (ex.: `pvm-linux-amd64`).

## Permissões e requisitos

- Permissão `contents: read`.
- O repositório chamador precisa ter um `go.mod` na raiz e um pacote `main` (na raiz ou no caminho de `package`) com a variável `version` (`var version string`).
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

Projeto com o `main` fora da raiz:

```yaml
jobs:
  build:
    uses: rejmann/github-actions/.github/workflows/golang-build.yml@main
    with:
      name: ponto
      package: ./cmd/ponto
      platform: linux
```

## Notas técnicas

- A matriz é gerada dinamicamente em JSON e consumida com `fromJson`.
- Todos os builds rodam em `ubuntu-latest` via cross-compilation (`GOOS`/`GOARCH`).
- O nome do binário vem do input `name` (padrão `pvm`, para manter compatibilidade).
