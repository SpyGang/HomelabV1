# Transformando o Debian 13 (Trixie) em um Servidor Minimalista para Homelab


| Informações do Projeto | |
| :--- | :--- |
| **Infraestrutura:** | Homelab V1 |
| **Data de Início:** | Abril de 2026 |
| **Sistema Operacional:** | Debian 13 (Trixie) |
| **Placa de Rede:** | Qualcomm Atheros QCA9377 |

Este repositório documenta o processo de conversão de uma instalação padrão do Debian 13 Desktop (com interface gráfica) em um servidor **Headless** (via terminal) otimizado para Homelab, mantendo o funcionamento da placa de rede Wi-Fi **Qualcomm Atheros QCA9377** em um notebook.

---

## 🔍 O Problema: Por que a interface não sumia e o Wi-Fi caía?

Ao tentar transformar um sistema Desktop em Servidor, dois problemas principais costumam acontecer:

1. **O comportamento do `tasksel`:** A ferramenta `tasksel` é excelente para instalar pacotes em lote (como o "Debian desktop environment"), mas na hora de remover, ela é projetada para ser conservadora. Se algum arquivo ou serviço essencial do sistema marcar a interface gráfica como uma dependência ativa, o `tasksel` simplesmente ignora a remoção para evitar que o usuário fique com uma tela preta quebrada.
2. **A armadilha do Wi-Fi:** Ambientes gráficos como o GNOME trazem consigo o `network-manager-gnome` e dependências de firmware. Se você remove a interface gráfica de forma bruta, o Debian desinstala o gerenciador de rede junto. Sem um utilitário de terminal instalado previamente (como o `nmtui` ou `iwd`) e sem garantir o pacote `firmware-atheros`, o chip Qualcomm QCA9377 perde a comunicação com o Kernel, deixando o notebook isolado e sem internet.

A solução foi forçar o sistema a iniciar em modo texto (`multi-user.target`), expurgar manualmente os pacotes principais do GNOME e limpar as sobras garantindo a permanência do driver de rede.

---

## 🛠️ Passo a Passo da Conversão

### Passo 1: Garantir as Dependências de Rede (Antes de deletar a GUI)
Para não perder o acesso ao Wi-Fi durante o processo, certifique-se de que o gerenciador de rede via terminal (`NetworkManager` + `nmtui`) e o firmware correto da Atheros estão instalados e protegidos:

```bash
sudo apt update
sudo apt install network-manager firmware-atheros -y

```

### Passo 2: Alterar o Alvo de Inicialização do Sistema

Força o systemd a iniciar o sistema diretamente no modo texto (Multi-User) em vez do modo gráfico (Graphical):

```bash
sudo systemctl set-default multi-user.target

```

### Passo 3: Remoção Manual e Agressiva da Interface Gráfica

Contorne a falha do `tasksel` removendo diretamente o núcleo do ambiente gráfico padrão (GNOME) e o gerenciador de login (GDM3):

```bash
sudo apt purge gnome-shell gdm3 gnome-session -y

```

*(Nota: Se o seu sistema utilizava XFCE, KDE ou Cinnamon, substitua `gnome-shell` pelo pacote correspondente da sua respectiva DE).*

### Passo 4: Limpeza Profunda de Resíduos

Elimine todas as dependências sobressalentes, drivers de vídeo desnecessários, fontes e bibliotecas gráficas que ficaram órfãs e estão ocupando espaço:

```bash
sudo apt autoremove --purge -y

```

---

## 📶 Gerenciamento do Wi-Fi no Modo Texto

Após reiniciar o servidor (`sudo reboot`), o sistema subirá exclusivamente via linha de comando. Para gerenciar, escanear e conectar a novas redes Wi-Fi usando a interface visual em modo texto, utilize o utilitário integrado:

```bash
nmtui

```

### Verificação do Consumo de Recursos

Para validar a eficiência do processo, verifique o consumo de memória RAM com o comando:

```bash
free -h

```

*O consumo esperado pós-oficina deve girar entre **100MB e 150MB de RAM**, deixando praticamente 100% do hardware livre para Docker, containers e serviços do Homelab.*

---
