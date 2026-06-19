# 🌐 Personal Homelab Infrastructure

Este repositório documenta a arquitetura, a configuração e o gerenciamento do meu antigo **Homelab pessoal**. Projetado com foco em alta disponibilidade, segurança e observabilidade, este ambiente serviu como meu principal laboratório técnico para simular cenários de produção, testar novas tecnologias e hospedar serviços essenciais de forma totalmente autônoma (*self-hosted*).

O objetivo deste documento é apresentar de forma clara e estruturada as minhas decisões de engenharia, a pilha tecnológica utilizada e as competências desenvolvidas em administração de sistemas, redes, segurança e DevOps — servindo como portfólio técnico para entrevistas de emprego e avaliações profissionais.

---

## 🏗️ Arquitetura e Pilha Tecnológica Central

O Homelab foi construído seguindo os princípios de infraestrutura enxuta, isolamento de processos e segurança *Zero-Trust*.

* **Sistema Operacional Base:** `Debian 13 (Trixie)` — Escolhido por sua lendária estabilidade, robustez como servidor e compatibilidade avançada com o ecossistema Docker e kernels Linux modernos.
* **Orquestração e Conteinerização:** `Docker & Docker Compose` — Toda a infraestrutura foi modularizada em contêineres independentes, garantindo isolamento de dependências, portabilidade e facilidade de atualização (seguindo conceitos de *Infrastructure as Code*).
* **Rede Segura (Mesh VPN):** `Tailscale (WireGuard)` — Implementação de uma rede sobreposta (*overlay network*) segura. Todos os serviços críticos eram acessíveis remotamente de forma criptografada ponto a ponto, **sem a necessidade de abrir portas públicas (Port Forwarding)** no roteador residencial, mitigando vetores de ataque externos.

---

## 🗂️ Ecossistema de Serviços (Módulos)

Abaixo estão os serviços que compunham o ecossistema do meu Homelab, organizados logicamente por domínio técnico de acordo com a estrutura de diretórios do projeto:

### 1. Entrada de Rede & Gerenciamento (Ingress & Control Plane)
* **`nginx` (Reverse Proxy):** Atuava como o único ponto de entrada para o tráfego HTTP/HTTPS interno. Configurado para terminação SSL/TLS, roteamento de subdomínios internos legíveis e segurança de cabeçalhos.
* **`portainer` (Docker Management):** Interface gráfica avançada para monitoramento e administração em tempo real de contêineres, volumes, imagens e redes virtuais do Docker.

### 2. Observabilidade & Monitoramento (SRE / Day-2 Operations)
* **`prometheus + grafana`:** Stack profissional de observabilidade. O Prometheus atuava na coleta ativa de métricas (time-series) do host e do Docker Daemon, enquanto o Grafana renderizava dashboards customizados para análise de performance, consumo de recursos e capacidade.
* **`kumaUptime` (Uptime Kuma):** Motor de monitoramento contínuo de disponibilidade (HTTP, Ping, DNS). Configurado para acompanhar o SLA dos serviços e emitir alertas em tempo real caso alguma aplicação ficasse indisponível.
* **`dashdot`:** Dashboard moderno e responsivo para visualização rápida da integridade do hardware (uso de CPU, memória RAM, armazenamento e tráfego de rede).
* **`speedtestTracker`:** Automação de testes periódicos de largura de banda da internet, salvando os históricos em banco de dados para auditoria de desempenho do link residencial.

### 3. Inteligência Artificial Local (AI & Innovation)
* **`ollama + openwebui`:** Infraestrutura local para execução de Large Language Models (LLMs). O Ollama gerenciava os pesos dos modelos (ex: Llama 3, Mistral) com aceleração local, e o Open WebUI provia uma interface rica de chat (estilo ChatGPT), garantindo total privacidade de dados e permitindo o teste de integrações de IA locais.

### 4. Cloud Privada & Produtividade
* **`nextcloud`:** Uma suíte completa de nuvem privada (alternativa ao Google Drive/OneDrive) para sincronização de arquivos, calendários e contatos, garantindo a soberania absoluta dos dados.
* **`jupyter` (Data Science):** Servidor Jupyter Notebook para desenvolvimento, prototipagem em Python, execução de scripts de automação de infraestrutura e análise de dados.
* **`homarr` (Application Portal):** O portal de entrada do Homelab. Um dashboard centralizado, customizado com atalhos e integrações de status via API para todos os serviços ativos na malha.

### 5. Camada de Persistência (Data Layer)
* **`databaseJP`:** Instância/Cluster dedicado de banco de dados (ex: PostgreSQL/MySQL) estruturado de forma centralizada para fornecer persistência de dados de alta performance para os serviços que demandavam armazenamento relacional robusto.

---

## 🔒 Engenharia de Redes e Segurança (SecOps)

Um dos maiores diferenciais deste projeto foi o foco rigoroso na superfície de ataque e no design de rede:

1.  **Isolamento de Redes no Docker:** Os contêineres de banco de dados (`databaseJP`) e backends não possuíam portas expostas diretamente para o host. O isolamento era garantido por redes virtuais Docker customizadas (`bridge`), permitindo comunicação estritamente necessária (ex: apenas o Nextcloud falava com o seu banco correspondente).
2.  **Topologia Zero-Trust com Tailscale:** O servidor Debian não possuía nenhum IP público exposto na internet e o roteador da operadora operava com o firewall totalmente fechado. O acesso externo era feito exclusivamente pela interface de rede virtual do Tailscale (criptografada via protocolo WireGuard).
3.  **Segurança a Nível de Host:** Firewall nativo do Linux (`UFW`/`nftables`) configurado com políticas restritivas baseadas em lista branca, liberando conexões apenas provenientes da sub-rede local segura e da interface da VPN.

---

## 🧠 Hard Skills Consolidadas e Casos de Uso para Entrevistas

A manutenção contínua deste Homelab me proporcionou sólida experiência prática nas seguintes frentes:

* **Sysadmin & Linux Avançado:** Administração profunda do Debian (systemd, gerenciamento de pacotes apt, análise fina de logs com `journalctl`, gerenciamento de armazenamento e permissões POSIX).
* **Cultura DevOps / IaC:** Pensamento modular voltado a contêineres, ciclo de vida do Docker, persistência segura de volumes, criação de redes isoladas e escrita de arquivos `docker-compose.yml` otimizados e reutilizáveis.
* **Princípios de SRE (Site Reliability Engineering):** Definição de métricas de saúde de sistemas, monitoramento proativo, visualização de dados complexos com Grafana e estabelecimento de rotinas de alertas de incidentes.
* **Arquitetura de Redes e Cibersegurança:** Compreensão prática de redes sobrepostas (Mesh VPNs), roteamento interna de proxies reversos, gerenciamento de certificados SSL/TLS e blindagem de servidores contra varreduras de portas externas.

---
*Nota: Este projeto reflete a minha paixão por tecnologia, capacidade de autoaprendizado e a habilidade de projetar sistemas complexos auto-hospedados utilizando as melhores práticas do mercado corporativo.*
