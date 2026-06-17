# 🔋 Configuração de Limite de Carga da Bateria para Homelab (Debian)

Este guia documenta o procedimento realizado para limitar a carga da bateria em um notebook transformado em servidor doméstico (**Homelab**) rodando **Debian 13 (Trixie)**. 

O objetivo principal é evitar o estresse químico e o estufamento da bateria, já que o servidor permanece conectado à tomada 24 horas por dia, 7 dias por semana. Para este cenário, o hardware foi configurado para o **Modo de Conservação**, que estabiliza a carga em **60%**.

---

## 🛠️ Cenário e Diagnóstico

Ao tentar gerenciar a bateria diretamente pelo sistema de arquivos do Kernel (`/sys/class/power_supply/BAT0/`), o hardware apresentou restrições de escrita devido à arquitetura do firmware (comum em notebooks Lenovo/IdeaPad e similares). 

A solução ideal foi utilizar o **TLP**, uma ferramenta avançada de gerenciamento de energia para Linux que interage diretamente com os módulos específicos do firmware do fabricante.

---

## 🚀 Passo a Passo de Configuração

### 1. Instalação do TLP
Primeiro, atualize a lista de pacotes e instale o daemon do TLP:

```bash
sudo apt update
sudo apt install tlp tlp-rdw -y

```

> **Nota de Troubleshooting:** Caso encontre erros de trava no APT (`/var/lib/apt/lists/lock`), encerre processos fantasmas com:
> ```bash
> sudo rm /var/lib/apt/lists/lock
> sudo rm /var/lib/dpkg/lock-frontend
> sudo dpkg --configure -a
> 
> ```
> 
> 

### 2. Configuração do Modo de Conservação

Como o hardware utiliza um seletor binário (`0` ou `1`) em vez de porcentagens customizadas livres, devemos ativar o modo nativo do fabricante.

Abra o arquivo de configuração do TLP:

```bash
sudo nano /etc/tlp.conf

```

Garantir que as linhas de limite percentual padrão estejam **comentadas** (com `#`), pois elas geram conflito:

```text
#START_CHARGE_THRESH_BAT0=45
#STOP_CHARGE_THRESH_BAT0=50

```

Navegue até o final do arquivo e adicione explicitamente a flag de conservação (substituindo ou adicionando a linha):

```text
LENOVO_BAT_CONSERVATION_MODE=1

```

*Salve o arquivo (`Ctrl+O`, `Enter`) e saia (`Ctrl+X`).*

### 3. Aplicando as Configurações

Reinicie o serviço do TLP para que o perfil seja lido e aplicado imediatamente ao hardware:

```bash
sudo tlp start

```

*Alternativa rápida via CLI:* Se preferir forçar a ativação sem editar o arquivo, o TLP disponibiliza o comando:

```bash
sudo tlp setcharge 0 1

```

---

## 📊 Validação do Status

Para certificar-se de que o ecossistema aceitou a configuração, execute o validador de bateria do TLP:

```bash
sudo tlp-stat -b

```

Verifique nas últimas linhas se o parâmetro foi alterado com sucesso:

```text
conservation_mode = 1 (on)

```

Você também pode checar o status direto do fornecimento de energia. Se a bateria estiver acima de 60%, o status deve constar como interrompido:

```bash
cat /sys/class/power_supply/BAT0/status
# Saída esperada: Not charging (ou Unknown)

```

---

💾 *Mantido como parte da documentação de infraestrutura do meu Homelab.*
