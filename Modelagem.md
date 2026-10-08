# Documento de Modelagem de Rede — Serviço DHCP, Roteamento e NAT

**Disciplina:** Redes de Computadores / Trilha de Serviços de Rede

**Ambiente de Emulação:** Kathará

**Integrantes:** [Sarah Oliveira] / [Evelin Leah]

**Data:** 08/10/2026

---

## 1. Visão Geral da Topologia

A topologia é composta por 3 sub-redes locais (LAN A, LAN B e LAN C), interconectadas por roteadores Linux emulados via Kathará. O roteador **R1** atua como Roteador de Borda e Gateway Principal da infraestrutura, realizando a tradução de endereços de rede (NAT - Masquerading) via interface `bridged` para fornecer conectividade com a Internet externa aos demais nós da rede.

Cada roteador local (`R1`, `R2` e `R3`) atua simultaneamente como **Gateway Padrão** e **Servidor DHCP** (utilizando o serviço `dnsmasq`) para a sua respectiva sub-rede, eliminando a necessidade de retransmissores DHCP (*DHCP Relay*).

### Mapeamento dos Barramentos (Kathará)

* **Barramento A (LAN A):** `pc0`, `pc1`, `r2[0]`
* **Barramento B (LAN B):** `pc2`, `pc3`, `r3[0]`
* **Barramento C (LAN C / Backbone):** `pc4`, `pc5`, `r1[0]`, `r2[1]`, `r3[1]`
* **Barramento Bridged (Internet):** `r1[bridged]=true` (Interface física `eth1` do `r1` mapeada na máquina Host)

---

## 2. Tabela de Endereçamento IP

| Nó | Interface | Endereço IP / Máscara | Gateway Padrão | Função no Ambiente |
| --- | --- | --- | --- | --- |
| **`r1`** | `eth0` | `192.168.10.1/24` | N/A | Gateway LAN C + Servidor DHCP LAN C |
| **`r1`** | `eth1` | Atribuído via DHCP (Host) | Gateway da Rede Física | Interface de Saída WAN com NAT Masquerade |
| **`r2`** | `eth0` | `192.168.20.1/24` | N/A | Gateway LAN A + Servidor DHCP LAN A |
| **`r2`** | `eth1` | `192.168.10.2/24` | `192.168.10.1` | Interface Backbone (Conexão com `r1`) |
| **`r3`** | `eth0` | `192.168.30.1/24` | N/A | Gateway LAN B + Servidor DHCP LAN B |
| **`r3`** | `eth1` | `192.168.10.3/24` | `192.168.10.1` | Interface Backbone (Conexão com `r1`) |
| **`pc0`** | `eth0` | DHCP Dinâmico | `192.168.20.1` | Cliente de Rede - LAN A |
| **`pc1`** | `eth0` | DHCP Dinâmico | `192.168.20.1` | Cliente de Rede - LAN A |
| **`pc2`** | `eth0` | DHCP Dinâmico | `192.168.30.1` | Cliente de Rede - LAN B |
| **`pc3`** | `eth0` | DHCP Dinâmico | `192.168.30.1` | Cliente de Rede - LAN B |
| **`pc4`** | `eth0` | DHCP Dinâmico | `192.168.10.1` | Cliente de Rede - LAN C |
| **`pc5`** | `eth0` | DHCP Dinâmico | `192.168.10.1` | Cliente de Rede - LAN C |

---

## 3. Configuração dos Escopos e Faixas DHCP

Os serviços DHCP foram distribuídos descentralizadamente nos roteadores locais de cada sub-rede usando a ferramenta `dnsmasq`.

### Faixas de Atribuição e Parâmetros da Rede

* **LAN A (Servidor `r2` na `eth0`):**
* Sub-rede: `192.168.20.0/24`
* Intervalo de Concessão: `192.168.20.100` até `192.168.20.200`
* Máscara de Rede: `255.255.255.0`
* Gateway Oferecido: `192.168.20.1`
* Servidor DNS Oferecido: `8.8.8.8`
* Tempo de Concessão (*Lease Time*): 12 Horas


* **LAN B (Servidor `r3` na `eth0`):**
* Sub-rede: `192.168.30.0/24`
* Intervalo de Concessão: `192.168.30.100` até `192.168.30.200`
* Máscara de Rede: `255.255.255.0`
* Gateway Oferecido: `192.168.30.1`
* Servidor DNS Oferecido: `8.8.8.8`
* Tempo de Concessão (*Lease Time*): 12 Horas


* **LAN C (Servidor `r1` na `eth0`):**
* Sub-rede: `192.168.10.0/24`
* Intervalo de Concessão: `192.168.10.100` até `192.168.10.200`
* Máscara de Rede: `255.255.255.0`
* Gateway Oferecido: `192.168.10.1`
* Servidor DNS Oferecido: `8.8.8.8`
* Tempo de Concessão (*Lease Time*): 12 Horas



---

## 4. Plano de Roteamento Estático e NAT

Para permitir a trafegabilidade entre todas as sub-redes e o encaminhamento para a Internet, a tabela de rotas estáticas e regras de firewall foram definidas conforme abaixo:

### Tabelas de Roteamento Estático

1. **Roteador `r1` (Borda):**
* Rota Padrão (`0.0.0.0/0`): Dinâmica via `eth1` (`dhclient`) para a WAN.
* Rota LAN A: `192.168.20.0/24` via `192.168.10.2` (`eth0`).
* Rota LAN B: `192.168.30.0/24` via `192.168.10.3` (`eth0`).


2. **Roteadores `r2` e `r3` (Internos):**
* Rota Padrão (`0.0.0.0/0` em ambos): `via 192.168.10.1` (Encaminha todo o tráfego interno e externo para `r1`).



### Regras de Tradução de Endereço (NAT) no `r1`

* **Encaminhamento IP:** `sysctl net.ipv4.ip_forward=1` habilitado no kernel.
* **Masquerading (SNAT):** Traduz o IP de origem de qualquer pacote vindo das redes internas (`192.168.0.0/16`) para o IP da interface `eth1` ao sair para a Internet.
```bash
iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE

```



---

## 5. Plano de Testes e Validação

| ID do Teste | Descrição do Teste | Procedimento / Comando | Resultado Esperado |
| --- | --- | --- | --- |
| **TC-01** | Obtenção Dinâmica de IP via DHCP | Executar `dhclient eth0` nos PCs e verificar com `ip addr show eth0`. | O PC deve receber um IP válido dentro do escopo da sua LAN, máscara `/24` e gateway correto. |
| **TC-02** | Teste de Conectividade Local (Intra-LAN) | No `pc0`, executar `ping 192.168.20.1` (Gateway local). | Resposta de ICMP Reply sem perda de pacotes ($0\%$ *loss*). |
| **TC-03** | Roteamento Inter-LAN (Inter-Subredes) | No `pc0` (LAN A), executar `ping` para o IP do `pc2` (LAN B). | O tráfego deve atravessar `r2` -> `r1` -> `r3` -> `pc2` e retornar com sucesso. |
| **TC-04** | Acesso à Internet e NAT | Executar `ping 8.8.8.8` a partir do `pc0`, `pc2` ou `pc4`. | Resposta do IP do DNS do Google, confirmando o funcionamento do NAT no `r1`. |
| **TC-05** | Mapeamento de Rota (Traceroute) | Executar `traceroute 8.8.8.8` no `pc0`. | Mapeamento detalhado exibindo os saltos pelos gateways intermediários (`192.168.20.1` -> `192.168.10.1` -> Roteador Host/Internet). |
| **TC-06** | Captura de Pacotes DHCP no Wireshark | Iniciar o Wireshark na interface virtual do barramento A e renovar o IP no `pc0` (`dhclient -r && dhclient`). | Captura dos 4 pacotes do processo DORA (*Discover, Offer, Request, ACK*) com porta UDP 67/68. |



