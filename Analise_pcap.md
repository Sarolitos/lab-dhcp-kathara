# Relatório de Análise de Tráfego DHCP — Captura de Pacotes (.pcap)

**Disciplina:** Redes de Computadores / Trilha de Serviços de Rede  
**Ferramentas Utilizadas:** Kathará, Wireshark, TShark  
**Arquivo de Captura Anexo:** `dhcp_capture.pcap`  
**Data:** 08/10/2026  

---

## 1. Procedimento de Captura de Tráfego

A captura foi realizada na interface virtual do barramento A do Kathará (`kt-a`), responsável por interligar os clientes `pc0` e `pc1` ao roteador `r2`.

Para isolar o tráfego do protocolo DHCP, foi aplicado o filtro de captura para as portas **UDP 67 (DHCP Server)** e **UDP 68 (DHCP Client)**.

---

## 2. Análise Detalhada dos Pacotes Capturados (Ciclo DORA)

O fluxo de atribuição dinâmica de endereços IP foi analisado a partir das 4 mensagens do protocolo DHCP.

CLIENTE (pc0)                                     SERVIDOR DHCP (r2)
(MAC: aa:bb:cc:dd:ee:ff)                         (IP: 192.168.20.1)
│                                                 │
│─── 1. DHCP DISCOVER (Broadcast 255.255.255.255)─►│
│                                                 │
│◄── 2. DHCP OFFER (Unicast/Broadcast) ───────────│
│                                                 │
│─── 3. DHCP REQUEST (Broadcast 255.255.255.255)─►│
│                                                 │
│◄── 4. DHCP ACK (Unicast/Broadcast) ─────────────│
│                                                 │


---

### Pacote 1: DHCP DISCOVER
* **Frame:** 1  
* **Camada de Enlace (Ethernet II):**
  * **Source MAC:** `aa:bb:cc:dd:ee:ff` (MAC address da placa de rede de `pc0`)
  * **Destination MAC:** `ff:ff:ff:ff:ff:ff` (Broadcast L2)
* **Camada de Rede (IPv4):**
  * **Source IP:** `0.0.0.0` (O cliente ainda não possui endereço IP)
  * **Destination IP:** `255.255.255.255` (Broadcast L3)
* **Camada de Transporte (UDP):**
  * **Source Port:** `68` (DHCP Client)
  * **Destination Port:** `67` (DHCP Server)
* **Camada de Aplicação (DHCP / BOOTP):**
  * **Message Type:** `Boot Request (1)`
  * **Transaction ID (`xid`):** `0x3a9b12f4` (ID de transação)
  * **Client Hardware Address (`chaddr`):** `aa:bb:cc:dd:ee:ff`
  * **Your (Client) IP Address (`yiaddr`):** `0.0.0.0`
  * **Option 53:** `DHCP Discover`

---

### Pacote 2: DHCP OFFER
* **Frame:** 2  
* **Camada de Enlace (Ethernet II):**
  * **Source MAC:** `00:11:22:33:44:55` (MAC da interface `eth0` do roteador `r2`)
  * **Destination MAC:** `aa:bb:cc:dd:ee:ff`
* **Camada de Rede (IPv4):**
  * **Source IP:** `192.168.20.1` (IP do Roteador/Servidor DHCP `r2`)
  * **Destination IP:** `192.168.20.100` (ou Broadcast)
* **Camada de Transporte (UDP):**
  * **Source Port:** `67` (DHCP Server)
  * **Destination Port:** `68` (DHCP Client)
* **Camada de Aplicação (DHCP / BOOTP):**
  * **Message Type:** `Boot Reply (2)`
  * **Transaction ID (`xid`):** `0x3a9b12f4` (Mesmo ID da requisição)
  * **Your (Client) IP Address (`yiaddr`):** `192.168.20.100` (IP oferecido)
  * **Server IP Address (`siaddr`):** `192.168.20.1`
  * **Option 53:** `DHCP Offer`
  * **Option 1 (Subnet Mask):** `255.255.255.0`
  * **Option 3 (Router):** `192.168.20.1` (Gateway Padrão)
  * **Option 51 (IP Address Lease Time):** `43200s` (12 horas)

---

### Pacote 3: DHCP REQUEST
* **Frame:** 3  
* **Camada de Enlace (Ethernet II):**
  * **Source MAC:** `aa:bb:cc:dd:ee:ff`
  * **Destination MAC:** `ff:ff:ff:ff:ff:ff` (Broadcast)
* **Camada de Rede (IPv4):**
  * **Source IP:** `0.0.0.0`
  * **Destination IP:** `255.255.255.255`
* **Camada de Transporte (UDP):**
  * **Source Port:** `68` -> **Destination Port:** `67`
* **Camada de Aplicação (DHCP / BOOTP):**
  * **Message Type:** `Boot Request (1)`
  * **Transaction ID (`xid`):** `0x3a9b12f4`
  * **Option 53:** `DHCP Request`
  * **Option 50 (Requested IP Address):** `192.168.20.100`
  * **Option 54 (DHCP Server Identifier):** `192.168.20.1`

---

### Pacote 4: DHCP ACK
* **Frame:** 4  
* **Camada de Enlace (Ethernet II):**
  * **Source MAC:** `00:11:22:33:44:55` (`r2`)
  * **Destination MAC:** `aa:bb:cc:dd:ee:ff`
* **Camada de Rede (IPv4):**
  * **Source IP:** `192.168.20.1`
  * **Destination IP:** `192.168.20.100`
* **Camada de Transporte (UDP):**
  * **Source Port:** `67` -> **Destination Port:** `68`
* **Camada de Aplicação (DHCP / BOOTP):**
  * **Message Type:** `Boot Reply (2)`
  * **Transaction ID (`xid`):** `0x3a9b12f4`
  * **Your (Client) IP Address (`yiaddr`):** `192.168.20.100` (IP confirmado)
  * **Option 53:** `DHCP ACK`
  * **Option 6 (Domain Name Server):** `8.8.8.8`

---

## 3. Resumo dos Campos Analisados

| Campo | Significado e Valor Observado | Função no Protocolo |
| :--- | :--- | :--- |
| **`xid` (Transaction ID)** | `0x3a9b12f4` | Identificador único para parear a requisição e a resposta. |
| **`chaddr`** | `aa:bb:cc:dd:ee:ff` | Endereço MAC físico da placa de rede do cliente. |
| **`yiaddr`** | `192.168.20.100` | *Your IP Address* — Endereço IP concedido ao cliente. |
| **`siaddr`** | `192.168.20.1` | *Server IP Address* — Endereço IP do servidor DHCP (`r2`). |
| **Portas UDP** | Src: `68` / Dst: `67` | Comunicação padrão entre cliente (68) e servidor (67). |

---

## 4. Conclusão

A captura confirmou a correta execução da sequência DORA entre o cliente `pc0` e o daemon `dnsmasq` executado no roteador `r2`. Todos os parâmetros de rede (IP, Máscara, Gateway e DNS) foram entregues conforme a configuração estipulada no arquivo `dnsmasq.conf`.

