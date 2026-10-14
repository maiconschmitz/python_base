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
- O `FROM` do Dockerfile deve usar uma versao exata `MAJOR.MINOR.PATCH`, como `python:3.14.8-slim`, para permitir PRs de patch do Dependabot.
- Linhas ainda nao lancadas podem usar temporariamente uma versao `MAJOR.MINOR.PATCHrcN`, como `python:3.16.0rc1-slim`; documente claramente que a imagem e preview e atualize para a final assim que ela existir.
- A variavel `PYTHON_VERSION` no Dockerfile deve bater com a pasta da imagem.
- A tag `latest` acompanha a ultima linha listada em `versions.txt`.
- As tags de linha (`3.12`, `3.14`, `3.15`) e `latest` sao publicadas somente pela branch `main`; pushes em `develop` geram apenas tags identificadas pela branch. Se a ultima linha estiver em pre-lancamento, documente que `latest` tambem e preview.
- `versions.txt` e a fonte de verdade para o conjunto suportado e e lido pelo `Taskfile`, pela workflow e pelo helper de scaffolding.
- O Dependabot deve acompanhar `PATCH` updates apenas nas linhas ja existentes.
- Novas `MINOR` versions nao surgem automaticamente: crie a nova pasta e atualize a documentacao.

## Build local

Use `task` para reproduzir os builds da imagem:

- `task` lista os comandos disponiveis.
- `task build VERSION=3.15` constroi uma linha especifica.
- `task build-all` constroi todas as linhas.
- `task buildx VERSION=3.15` faz build multi-arquitetura.
- `task buildx-all` roda todas as linhas em multi-arquitetura.
- `task test` executa a validacao ponta a ponta em todas as linhas suportadas.
- `task test VERSION=3.15` executa a validacao em apenas uma linha.

O helper `scripts/add-version.sh` cria a nova pasta, deriva o Dockerfile da ultima linha suportada e recompõe o Dependabot. O patch pode ser estavel (`3.16.0`) ou pre-lancamento (`3.16.0a1`, `3.16.0b1`, `3.16.0rc1`).
O helper `scripts/test-builds.sh` cria um app temporario, constroi cada imagem e instala uma dependencia simples com `uv` dentro dela.

## O que observar ao editar

- Nao altere arquivos fora do escopo sem necessidade.
- Nao reverta mudancas do usuario.
- Mantenha os Dockerfiles consistentes entre si, mudando apenas o que varia por linha.
- Preserve o uso de `slim`, `uv`, `TZ=America/Sao_Paulo`, usuario `app` e `WORKDIR /app`, a menos que exista um motivo claro para mudar.
- Preserve o UID e o GID `1000` do usuario `app`. Os volumes persistentes dos ambientes de producao dependem dessa identidade numerica estavel; os testes devem validar tanto o usuario efetivo quanto a entrada `app`.
- Se mexer na pipeline, confira se os filtros de caminho e a matriz de versoes continuam alinhados com `versions.txt`.
