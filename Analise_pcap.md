# 📑 Relatório de Análise de Tráfego DHCP — Captura de Pacotes (.pcap)

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

