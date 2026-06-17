# 🔋 Configuração do Comportamento da Tampa do Notebook (Evitar Modo Repouso)

| Informações do Projeto | |
| :--- | :--- |
| **Infraestrutura:** | Homelab V1 |
| **Data de Início:** | Abril de 2026 |
| **Sistema Operacional:** | Debian 13 (Trixie) / Proxmox VE |
| **Status da Bateria:** | Tampa configurada para IGNORE (Sucesso) |

---

## 🛠️ Cenário e Diagnóstico

Este guia documenta o procedimento realizado para impedir que o notebook que atua como o servidor central do **Homelab V1** entre em modo de suspensão (repouso) ao fechar a tampa física do hardware. 

Este ajuste é fundamental para garantir a alta disponibilidade e a continuidade dos serviços, Máquinas Virtuais (VMs) e Containers (LXCs) que rodam dentro do ambiente virtualizado do Proxmox, impedindo que o sistema operacional mude o estado de energia do processador e das interfaces de rede.

---

## 📝 Passo a Passo Prático via Terminal

O comportamento dos eventos de energia do hardware (como o fechamento da tampa) é gerenciado pelo serviço `systemd-logind`. Para alterar a configuração padrão, siga os passos abaixo:

### 1. Acessando o arquivo de configuração
Abra o shell do seu nó Proxmox (via interface web ou SSH) e edite o arquivo com privilégios de superusuário:

```bash
sudo nano /etc/systemd/logind.conf

```

### 2. Alterando as Diretivas de Fechamento (Lid Switch)

Dentro do arquivo, localize as linhas descritas abaixo. Se houver um caractere `#` (comentário) no início delas, remova-o e modifique seus valores para `ignore`:

```text
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore

```

### 3. Salvando e Aplicando as Modanças

1. No editor Nano, pressione **Ctrl + O** seguido de **Enter** para salvar as alterações.
2. Pressione **Ctrl + X** para fechar o editor.

Para aplicar as novas diretivas imediatamente sem a necessidade de reiniciar o servidor físico, execute o reinício do serviço correspondente:

```bash
sudo systemctl restart systemd-logind

```

---

## 🔍 Significado dos Parâmetros Configurados

Ao definir o valor de comportamento como `ignore`, o `systemd` simplesmente desconsidera o sinal físico enviado pelo sensor da tampa.

* **`HandleLidSwitch`**: Define a ação quando a tampa é fechada enquanto o dispositivo está operando na **bateria**.
* **`HandleLidSwitchExternalPower`**: Define a ação quando a tampa é fechada com o dispositivo conectado diretamente à **tomada (energia externa)**. Esta é a regra mais importante para o ecossistema do Homelab se manter online.
* **`HandleLidSwitchDocked`**: Define o comportamento caso o notebook identifique que há um **monitor externo** ou uma Dock Station conectada (garante estabilidade mesmo se cabos de vídeo forem manipulados).

---

## ⚠️ Observações de Manutenção de Hardware

* **Fluxo de Ar e Dissipação**: Notebooks que rodam como servidores fechados 24/7 exigem atenção redobrada. Muitos modelos utilizam a região do teclado para dissipar parte do calor interno. Certifique-se de que o aparelho esteja em local ventilado para evitar superaquecimento.
* **Monitoramento**: Acompanhe as métricas de temperatura do processador (`sensors`) periodicamente através do painel principal do Proxmox VE.

```

```
