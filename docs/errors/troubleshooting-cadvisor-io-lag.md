# 🐳 Otimização de Performance e I/O do cAdvisor no Docker

| Informações do Projeto | |
| :--- | :--- |
| **Infraestrutura:** | Homelab V1 |
| **Data de Início:** | Junho de 2026 |
| **Sistema Operacional:** | Debian 13 (Trixie) / Docker Context |
| **Status do cAdvisor:** | Otimizado e Estável (Sucesso) |

---

## 🛠️ Cenário e Diagnóstico

Este guia documenta o procedimento realizado para mitigar o travamento e a lentidão crônica nas métricas do **cAdvisor**, que atua como o coletor de telemetria de containers no **Homelab V1**.

Durante a análise dos logs do container através do Portainer, identificou-se que o subsistema de arquivos do cAdvisor (`fsHandler.go`) entrava em gargalo severo ao tentar calcular o uso de disco e a contagem de *inodes* nas camadas de armazenamento (`overlayfs`) de containers específicos:

```text
fsHandler.go:135] fs: disk usage and inodes count on following dirs took 13.233134111s: [...]
fsHandler.go:135] fs: disk usage and inodes count on following dirs took 59.124197608s: [/var/lib/docker/rootfs/overlayfs/...]; will not log again for this container unless duration exceeds 4s

```

### Causa Raiz

O comportamento padrão do cAdvisor realiza varreduras exaustivas a cada poucos segundos. Em ambientes com múltiplos containers ou com containers que manipulam grande volume de arquivos pequenos (caches, logs ou uploads temporários), a checagem de *inodes* bloqueia o loop de processamento do cAdvisor por até **59 segundos**, fazendo com que o serviço pare de responder ou congestione o consumo de I/O do disco do servidor.

---

## 🚀 Resolução e Configuração Aplicada

Para solucionar o problema sem comprometer a coleta de métricas essenciais (como CPU e Memória), o intervalo de checagem global foi afrouxado e a varredura pesada de armazenamento interno por container foi desativada via *flags* de inicialização.

### 1. Identificação do Container Gargalo

Para descobrir qual container estava gerando a fila de leitura de quase 1 minuto, utilizou-se o hash parcial apontado no log:

```bash
docker ps -a --no-trunc | grep <ID_PARCIAL_DO_LOG>

```

*Nota: Após a identificação, recomenda-se aplicar políticas de rotação de logs ou limpeza de volumes no container afetado.*

### 2. Ajuste de Parâmetros (Docker Compose / Portainer)

O arquivo de deployment do cAdvisor foi atualizado com a inclusão de argumentos (`command`) de otimização, aumentando o tempo de *housekeeping* e desligando módulos de disco desnecessários para a infraestrutura do Homelab:

```yaml
version: '3.8'

services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:v0.49.1 # Ajuste para a sua versão utilizada
    container_name: cadvisor
    restart: unless-stopped
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    command:
      - '--housekeeping_interval=30s'          # Alivia o processador aumentando o intervalo de amostragem
      - '--global_housekeeping_interval=1m'     # Reduz a frequência de checagens globais do sistema
      - '--disable_metrics=disk,diskIO,tcp,udp' # Desativa métricas pesadas de tracking de inodes e socket
    ports:
      - "8080:8080"

```

---

## 📈 Resultados Obtidos

* **Estabilidade de Leitura:** O cAdvisor deixou de apresentar congelamentos na API e no envio de dados para o Prometheus/Grafana.
* **Redução de I/O Wait:** O consumo de disco gerado pela monitorização caiu drasticamente, eliminando os alertas de timeout superiores a 4 segundos gravados no log.
* **Leveza no Homelab:** A alteração do intervalo de 2s (padrão) para 30s reduziu o impacto residual na CPU do servidor central do Homelab.

```

```
