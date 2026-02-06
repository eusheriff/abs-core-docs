# ⚔️ PLANO DE GUERRA: 15 DIAS PARA SOBERANIA (HARDENING)

**OBJETIVO**: Transformar o ABS de "Software Simulation" para "Hardware Sovereign".
**STATUS ATUAL**: V6.5 Gold (Logic Ready, Physics Weak).
**PRAZO**: 15 Dias.

---

## 📊 A Métrica de Valor (REI)
Para validade atuarial (Seguradoras):
**REI < 15.0** (Índice de Exposição de Risco).
Isso garante o desconto de 30% nas apólices de Cyber-Insurance.

---

## 🗓️ CRONOGRAMA DE BATALHA

### 🏗️ DIAS 1-4: HARDWARE REAL (TEE)
*   **O Problema**: `tee_enclave.rs` é um stub (`println!`). O segredo está na RAM exposta.
*   **A Ação**: Integrar AWS Nitro Enclaves SDK.
*   **A Meta**: `Attestation Quote` assinado criptograficamente pelo processador em cada boot.
*   **Impacto**: Imunidade contra dumps de memória e rootkits no host.

### 📦 DIAS 5-8: A CAIXA PRETA (CONTEXT PROVENANCE)
*   **O Problema**: Logs mostram "O que" aconteceu, mas não "Por que". Falta causalidade.
*   **A Ação**: Implementar **Context Provenance Ledger** (Merkle Tree).
*   **A Meta**: Ancorar Hash(Prompt) + Hash(ToolCall) em uma estrutura auditável.
*   **Impacto**: Prova jurídica de alucinação vs comando explícito.

### 🛡️ DIAS 9-11: DEFESA ATIVA (ENTROPIA) - [JÁ INICIADO]
*   **O Problema**: Ataques polimórficos e ofuscação (Base64).
*   **A Ação**: Calibrar o `Entropy Monitor` no Kernel Rust (Já implementamos o básico).
*   **A Meta**: Bloqueio Zero-Day de injeções de alta entropia.
*   **Impacto**: Fim dos jailbreaks baseados em encoding.

### 🇧🇷 DIAS 12-13: SOBERANIA JURÍDICA (BR)
*   **O Problema**: O sistema é uma "arma ilegal" no Brasil (Sem Termos, Sem LGPD).
*   **A Ação**:
    *   Endpoint `/juridico/esquecer-me` (Data Shredding).
    *   `TERMOS_DE_USO_PT_BR.md`.
*   **Impacto**: Viabilidade comercial B2B no Brasil.

### ⛓️ DIAS 14-15: STRESS TEST & ANCHOR
*   **O Problema**: Nunca testamos sob fogo real.
*   **A Ação**: Rodar `chaos_test.py` contra o novo Kernel TEE.
*   **A Meta**: Ancorar o primeiro Merkle Root na Base L2 (Blockchain).
*   **Impacto**: "Mainnet Block Zero". Soberania comprovada.

---

## 💡 NOTAS TÉCNICAS E REFERÊNCIAS

1.  **Atestação**: Use AWS Nitro Enclaves SDK. Não invente criptografia própria.
2.  **Causalidade**: Merkle Trees são a única forma eficiente de provar estado passado.
3.  **Entropia**: Shannon Entropy > 4.8 é nosso "Smoke Detector".

---

## ⚖️ O VEREDITO DO TREINADOR

> "Você tem o mapa para o império, mas ainda está usando chaves de plástico em portas de aço."

Se falharmos nos Dias 1-4, o valuation cai de **$25M (Estratégico)** para **$2M (Acqui-hire)**.
O mercado não compra software; compra **Certeza**. A certeza só vem do Silício (Hardware).

**START DAY 1.**
