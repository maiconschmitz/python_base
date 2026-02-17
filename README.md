# Python Base Image 🐍

Imagem Docker, otimizada para execução de aplicações Python.

## 📋 Sobre o Projeto

Este repositório contém uma imagem Docker base otimizada para exeução de aplicações Python.
A imagem é projetada com foco em:

- **Segurança**: Execução com usuário não-root
- **Performance**: Otimização de layers e cache

## 🚀 Versão Disponível

### Python 3.14.2 (Latest)

- **Diretório**: `3.14.2/`
- **Base**: `python:3.14.2-slim`
- **Tag**: `python-base:3.14`

### Python 3.12.8

- **Diretório**: `3.12.8/`
- **Base**: `python:3.12.8-slim`
- **Tag**: `python-base:3.12`

## 🛠️ Características da Imagem

### Configurações de Ambiente

- **Timezone**: America/Sao_Paulo
- **Python**: Configurações otimizadas para produção
- **Usuário**: `app` (não-root)
- **Diretório de trabalho**: `/app`
- **UV Cache**: Configurado para `/home/app/.cache/uv`
- **Bytecode**: Compilação automática habilitada

### Dependências Incluídas

- `curl` - Cliente HTTP
- `ca-certificates` - Certificados SSL/TLS
- `uv` - Gerenciador de pacotes Python moderno e ultra-rápido

### Gerenciador de Pacotes UV

Esta imagem utiliza o `uv`, um gerenciador de pacotes Python moderno que oferece:

- ⚡ **Velocidade**: Até 10-100x mais rápido que pip
- 🔒 **Segurança**: Resolução de dependências determinística
- 📦 **Compatibilidade**: Totalmente compatível com pip e requirements.txt
- 🚀 **Performance**: Instalação otimizada com cache inteligente

### Otimizações Aplicadas

- ✅ Redução de layers Docker
- ✅ Limpeza completa de caches
- ✅ Remoção de pacotes desnecessários
- ✅ Configurações de segurança
- ✅ Documentação detalhada

## 📖 Como Usar

### Construção da Imagem

```bash
# Construir a imagem Python 3.12.8
cd 3.12.8
docker build -t python-base:3.12 .
```

### Uso como Base

```dockerfile
# Dockerfile da sua aplicação
FROM python-base:3.12

# Instalar dependências da aplicação
COPY requirements.txt .
RUN uv pip install --no-cache-dir -r requirements.txt

# Copiar código da aplicação
COPY . .

# Comando de execução
CMD ["python", "app.py"]
```

### Exemplo com Docker Compose

```yaml
version: '3.8'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      - PYTHONPATH=/app
    volumes:
      - .:/app
```

> 💡 **Dica**: Use esta imagem como base para seus projetos Python e mantenha-a atualizada para garantir segurança e performance.
