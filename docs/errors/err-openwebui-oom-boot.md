# 🐳 Docker: Loop de Inicialização e Status Unhealthy no Open WebUI

| Informações do Projeto | |
| :--- | :--- |
| **Infraestrutura:** | Homelab V1 |
| **Data do Incidente:** | Junho de 2026 |
| **Ambiente:** | Docker / Docker Compose |
| **Status do Serviço:** | Operacional (Sucesso) |

---

## 🛠️ Cenário e Diagnóstico

Este guia documenta o procedimento realizado para solucionar o travamento, o status de *Unhealthy* e o loop de reinicialização do container **Open WebUI** no servidor central do **Homelab V1**.

### O Problema
Após atualizações automáticas na imagem de tag `:main` do Open WebUI, o container passou a entrar em um ciclo de reinicializações silenciosas. Os logs paravam abruptamente logo após as mensagens de migração de banco de dados (`alembic`) e validação de CORS:

```text
INFO  [alembic.runtime.migration] Running upgrade 4de81c2a3af1 -> a0b1c2d3e4f5, Add memory user_id index
WARNI [open_webui.env] 
WARNING: CORS_ALLOW_ORIGIN IS SET TO '*' - NOT RECOMMENDED FOR PRODUCTION DEPLOYMENTS.

```

No painel de gerenciamento, tanto o container do `open-webui` quanto o do `cadvisor` (que também expõe a porta interna `8080`) exibiam a tag **unhealthy**.

### Causa Raiz

1. **Insuficiência de Memória (OOM):** O container do Open WebUI estava severamente limitado a `mem_limit: 512m`. Com as novas atualizações da imagem, o processo Python exige mais memória na inicialização para processar migrações do SQLite e subfunções de IA. O sistema operacional do host matava o processo por falta de RAM (*Out of Memory*).
2. **Concorrência de Inicialização:** A escassez de recursos gerava concorrência indireta com outros serviços analíticos (como o `cadvisor`), fazendo com que o banco de dados ficasse preso em estados de *lock* temporários durante as tentativas automáticas de reinicialização do Docker (`unless-stopped`).

---

## 🚀 Resolução e Ajustes

A solução consistiu em redimensionar os recursos atribuídos ao container para permitir que a pilha de migração de dados fosse concluída com sucesso.

### Alteração no Docker Compose

A configuração de memória foi expandida de **512MB para 1GB (ou 2GB para maior margem de segurança)**, dando fôlego para o interpretador Python estabilizar.

Abaixo está o trecho do arquivo corrigido:

```yaml
open-webui:
  image: ghcr.io/open-webui/open-webui:main
  container_name: open-webui
  restart: unless-stopped
  environment:
    - OLLAMA_BASE_URL=http://ollama:11434
  ports:
    - "${WEBUI_PORT}:8080"
  volumes:
    - webui_data:/app/backend/data
  depends_on:
    - ollama
  mem_limit: 1g # <-- Incrementado de 512m para 1g para evitar OOM no boot
  networks:
    - labnet

```

---

## 📊 Comportamento Pós-Correção e Validação

Após aplicar o novo limite de memória e reiniciar a stack, observou-se o seguinte comportamento:

* **Primeiro Boot Técnico:** O container levou aproximadamente **12 minutos** em background para concluir a inicialização.
* **O que ocorreu nesse intervalo:** O processo utilizou a memória extra para destravar os arquivos de índice do SQLite (`webui.db`), validar as tabelas estruturais pendentes (`pinned_note`, `memory user_id`) e sincronizar o socket com a rede interna.
* **Resultado Atual:** O serviço estabilizou completamente, o status de *Unhealthy* foi mitigado e o Open WebUI encontra-se 100% operacional através da porta externa configurada.

---

💡 **Lição Aprendida para o Homelab:** Evitar limitar aplicações baseadas em Python/FastAPI voltadas para IA a menos de 1GB de RAM, pois rotinas de migração de banco de dados e checagem de concorrência podem causar falhas silenciosas de alocação de memória no boot.
