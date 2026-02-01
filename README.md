# Docker OpenClaw + Ollama

Setup Docker para rodar OpenClaw com Ollama (LLM local) em VPS.

## Requisitos

- Docker Engine + Docker Compose v2
- VPS com 8GB+ RAM (16GB recomendado)
- 40GB+ disco SSD

Veja [requisitos-hardware-ollama.md](requisitos-hardware-ollama.md) para detalhes.

## Deploy Rapido

```bash
# Clonar repositorio principal do OpenClaw
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Copiar arquivos deste repo (ou baixar diretamente)
curl -O https://raw.githubusercontent.com/inematds/docker-clawdbot-ollama/main/docker-compose.ollama.yml
curl -O https://raw.githubusercontent.com/inematds/docker-clawdbot-ollama/main/docker-ollama-setup.sh
chmod +x docker-ollama-setup.sh

# Rodar setup automatico
./docker-ollama-setup.sh
```

## O que o script faz

1. Verifica recursos do sistema (RAM, disco)
2. Builda a imagem Docker do OpenClaw
3. Inicia o container Ollama
4. Baixa o modelo llama3.2 automaticamente
5. Configura OpenClaw para usar Ollama como provider
6. Roda o wizard de onboarding
7. Inicia todos os servicos

## Setup Manual

```bash
cd openclaw

# Build da imagem
docker build -t openclaw:local -f Dockerfile .

# Iniciar com Ollama
docker compose -f docker-compose.yml -f docker-compose.ollama.yml up -d

# Baixar modelo
docker compose -f docker-compose.yml -f docker-compose.ollama.yml exec ollama ollama pull llama3.2

# Onboarding
docker compose -f docker-compose.yml -f docker-compose.ollama.yml run --rm openclaw-cli onboard
```

## Configuracao

O script cria automaticamente `~/.openclaw/config.json5`:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/llama3.2" },
    },
  },
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama:11434/v1",
        apiKey: "ollama-local",
      },
    },
  },
}
```

## Modelos Recomendados

| RAM | Modelo | Comando |
|-----|--------|---------|
| 8 GB | llama3.2 (3B) | `ollama pull llama3.2` |
| 8 GB | phi3 (3.8B) | `ollama pull phi3` |
| 16 GB | mistral (7B) | `ollama pull mistral` |
| 16 GB | llama3.1:8b | `ollama pull llama3.1:8b` |

## Portas

| Porta | Servico |
|-------|---------|
| 18789 | Gateway (API + UI) |
| 18790 | Bridge |
| 11434 | Ollama API |

## Verificacao

```bash
# Status dos containers
docker compose -f docker-compose.yml -f docker-compose.ollama.yml ps

# Testar Ollama
curl http://localhost:11434/api/tags

# Testar Gateway
curl http://localhost:18789/health

# Ver modelos instalados
docker compose exec ollama ollama list

# Ver uso de recursos
docker stats
```

## Arquivos

- `docker-compose.ollama.yml` - Override do Docker Compose para adicionar Ollama
- `docker-ollama-setup.sh` - Script de setup automatizado
- `docker-vps-setup.md` - Documentacao completa de deploy em VPS
- `requisitos-hardware-ollama.md` - Requisitos de hardware detalhados

## Links

- [OpenClaw](https://github.com/openclaw/openclaw)
- [Ollama](https://ollama.ai)
- [Documentacao OpenClaw](https://docs.openclaw.ai)
