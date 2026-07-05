# AGENTS.md

Guia rapido para agentes trabalhando neste repositorio.

## Objetivo do projeto

Este repositorio mantem imagens Docker base para aplicacoes Python.
Cada linha suportada do Python vive em um diretorio separado e gera uma imagem propria.
A lista de linhas suportadas fica em `versions.txt`.

O foco das imagens e permitir que builds de aplicacoes Python acontecam na VPS sem instalar ferramentas e bibliotecas de apoio diretamente no host.

## Estrutura principal

- `README.md`: documentacao de uso e visao geral.
- `AGENTS.md`: guia de manutencao para agentes.
- `versions.txt`: lista central das linhas Python suportadas.
- `taskfile.yml`: comandos locais para build.
- `3.x/Dockerfile`: definicoes das imagens por linha, uma pasta por linha em `versions.txt`.
- `.github/workflows/docker-build-push.yml`: pipeline de build e publicacao.
- `.github/dependabot.yml`: automacao de updates para as linhas suportadas.
- `scripts/add-version.sh`: helper para criar uma nova linha e regenerar o Dependabot.
- `scripts/test-builds.sh`: validacao ponta a ponta reutilizavel por `task test`.

## Convencoes do repositorio

- O diretorio da imagem deve corresponder a linha `MAJOR.MINOR` do Python, como `3.14`.
- O `FROM` do Dockerfile deve usar uma versao `MAJOR.MINOR.PATCH`, como `python:3.14.2-slim`, para permitir PRs de patch do Dependabot.
- A variavel `PYTHON_VERSION` no Dockerfile deve bater com a pasta da imagem.
- A tag `latest` acompanha a ultima linha listada em `versions.txt`.
- `versions.txt` e a fonte de verdade para o conjunto suportado e e lido pelo `Taskfile`, pela workflow e pelo helper de scaffolding.
- O Dependabot deve acompanhar `PATCH` updates apenas nas linhas ja existentes.
- Novas `MINOR` versions nao surgem automaticamente: crie a nova pasta e atualize a documentacao.

## Build local

Use `task` para reproduzir os builds da imagem:

- `task` lista os comandos disponiveis.
- `task build VERSION=3.14` constroi uma linha especifica.
- `task build-all` constroi todas as linhas.
- `task buildx VERSION=3.14` faz build multi-arquitetura.
- `task buildx-all` roda todas as linhas em multi-arquitetura.
- `task test` executa a validacao ponta a ponta em todas as linhas suportadas.
- `task test VERSION=3.14` executa a validacao em apenas uma linha.

O helper `scripts/add-version.sh` cria a nova pasta, deriva o Dockerfile da ultima linha suportada e recompõe o Dependabot.
O helper `scripts/test-builds.sh` cria um app temporario, constroi cada imagem e instala uma dependencia simples com `uv` dentro dela.

## O que observar ao editar

- Nao altere arquivos fora do escopo sem necessidade.
- Nao reverta mudancas do usuario.
- Mantenha os Dockerfiles consistentes entre si, mudando apenas o que varia por linha.
- Preserve o uso de `slim`, `uv`, `TZ=America/Sao_Paulo`, usuario `app` e `WORKDIR /app`, a menos que exista um motivo claro para mudar.
- Se mexer na pipeline, confira se os filtros de caminho e a matriz de versoes continuam alinhados com `versions.txt`.
