# Imagem base otimizada para Python 3.12 com uv
FROM python:3.14.0-slim

# Metadados da imagem
LABEL maintainer="maiconschmitz@gmail.com" \
      version="2.0" \
      description="Imagem base Python 3.12 otimizada com uv" \
      python.version="3.12.8"

# Variáveis de ambiente otimizadas
ENV TZ=America/Sao_Paulo \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    HOME=/home/app \
    USER=app \
    PATH="/home/app/.local/bin:$PATH" \
    UV_CACHE_DIR=/home/app/.cache/uv \
    UV_COMPILE_BYTECODE=1 \
    UV_PYTHON=/usr/local/bin/python3.12

# Configuração do diretório de trabalho
WORKDIR /app

# Instalação otimizada em uma única camada
RUN set -eux; \
    # Atualiza e instala apenas dependências essenciais
    apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        ca-certificates && \
    # Instala uv diretamente
    curl -LsSf https://astral.sh/uv/install.sh | sh && \
    # Cria usuário não-root
    groupadd -r app && \
    useradd --no-log-init -r -g app -m -d /home/app app && \
    # Configura permissões e cache do uv
    mkdir -p /home/app/.cache/uv && \
    chown -R app:app /home/app /app && \
    chmod 755 /home/app /home/app/.cache/uv && \
    chmod 777 /app && \
    # Remove qualquer ambiente virtual existente
    rm -rf /app/.venv && \
    # Limpeza completa
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /root/.cache

USER app
