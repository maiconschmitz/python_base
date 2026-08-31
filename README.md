# Python Base Image

Imagens Docker base, otimizadas para execução e preparação de aplicações Python.

## Visão geral

Este repositório mantém uma imagem por linha suportada do Python.
A lista de versões suportadas fica em `versions.txt`. Cada linha corresponde a uma pasta de versão, como `3.12/` ou `3.14/`.

Cada imagem usa a base oficial `python:<versão-patch>-slim`, instala o `uv`, configura um usuário não-root e deixa o ambiente pronto para ser usado como base de aplicações Python em builds locais, CI ou na VPS.
Os diretórios usam `MAJOR.MINOR`, enquanto o `FROM` do Dockerfile mantém `MAJOR.MINOR.PATCH`; assim, o Dependabot consegue abrir PRs apenas para correções de patch dentro da mesma linha.

## O que a imagem entrega

- Base oficial Python slim
- `uv` instalado como gerenciador de pacotes
- Usuário `app` não-root
- UID e GID estáveis em `1000`, compatíveis com volumes persistentes dos hosts
- `WORKDIR` definido em `/app`
- Timezone configurado para `America/Sao_Paulo`
- Cache do `uv` em `/home/app/.cache/uv`
- Bytecode compilado automaticamente pelo `uv`

## Versões suportadas

A imagem para cada linha suportada é gerada a partir do diretório correspondente em `versions.txt`.
Hoje as linhas suportadas são:

- `3.12/`, usando `python:3.12.8-slim`
- `3.14/`, usando `python:3.14.2-slim`

O helper `scripts/add-version.sh` cria uma nova pasta e o Dockerfile com base na última linha já suportada.

## Build local

Este projeto usa [`Taskfile`](https://taskfile.dev/) para facilitar builds locais e reproduzir a lógica da pipeline.

Pré-requisitos:

1. Docker ou Docker Desktop instalado
2. `go-task` instalado
3. `docker buildx` habilitado se você quiser build multi-arquitetura

Se for a primeira vez usando buildx:

```bash
docker buildx create --use
```

Comandos principais:

```bash
task
task build VERSION=3.14
task build-all
task buildx VERSION=3.14
task buildx-all
task test
task test VERSION=3.14
```

Observações:

- `task build` gera a imagem para a arquitetura atual.
- `task buildx` gera a imagem para `linux/amd64` e `linux/arm64`.
- O `Taskfile` adiciona a tag `latest` automaticamente quando a versão é a última linha em `versions.txt`.
- `task test` chama um script dedicado que constrói cada imagem e valida instalação de dependência com `uv`.
- `task test VERSION=3.14` valida apenas uma versão específica.

## Uso como imagem base

Exemplo de uso em um `Dockerfile` de aplicação:

```dockerfile
FROM maiconschmitz/python-base:3.14

WORKDIR /app

COPY requirements.txt ./
RUN uv venv .venv && \
    uv pip install --python .venv/bin/python --no-cache-dir -r requirements.txt

COPY . .
CMD ["/app/.venv/bin/python", "app.py"]
```

## Publicação e CI

A publicação automatizada acontece pela workflow `.github/workflows/docker-build-push.yml`:

- executa build para todas as versões listadas em `versions.txt`
- usa `docker buildx`
- publica imagens multi-arquitetura no Docker Hub
- aplica a tag `latest` apenas para a última versão da lista
- publica as tags estáveis de linha e `latest` somente a partir da branch `main`; `develop` permanece isolada em tags próprias

## Dependabot

O arquivo `.github/dependabot.yml` está configurado para acompanhar as imagens Docker dos diretórios atuais.
Na prática, ele ajuda a manter cada linha existente atualizada com os `PATCH` releases da respectiva linha do Python.

Isso significa:

- `3.12` continua recebendo correções da linha `3.12.x`
- `3.14` continua recebendo correções da linha `3.14.x`

Quando surgir uma nova versão menor, como `3.15`, a adição precisa ser feita manualmente no repositório, criando:

- a nova pasta `3.15/`
- o novo `Dockerfile`
- a entrada em `versions.txt`
- a entrada correspondente no Dependabot
- a atualização deste `README.md`

Para facilitar esse fluxo, há um helper em `scripts/add-version.sh`:

```bash
./scripts/add-version.sh 3.15 3.15.0
```

A validação ponta a ponta fica em `scripts/test-builds.sh` e é a mesma usada por `task test`.

O arquivo `AGENTS.md` contém orientações de manutenção mais detalhadas para agentes e colaboradores.
