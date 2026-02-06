# ADR 009: Black Box Cryptography (Non-Repudiation)

## Status
ACCEPTED

## Context
O ABS Core utilizava HMAC-SHA256 para assinar logs de decisão. Isso garantia integridade, mas falhava em **Non-Repudiation** (Não-Repúdio).
Como o HMAC requer o compartilhamento da chave secreta para verificação, qualquer auditor capaz de verificar a assinatura também seria capaz de forjá-la.
Isso impedia o uso do ABS como evidência forense irrefutável (auditabilidade de cauda longa).

## Decision
Migrar o esquema de assinatura de envelopes de decisão para **Ed25519** (Criptografia Assimétrica).

1.  **Assinador**: `EventProcessor` gera ou carrega um par de chaves Ed25519 no boot.
2.  **Schema**: `DecisionEnvelope` agora inclui `public_key` e suporta `alg: "Ed25519"`.
3.  **Verificação**: Novo comando `abs verify` permite validação offline usando apenas a chave pública e o JSON do recibo.
4.  **Interceptors**: Criação de `packages/interceptors` para captura transparente de tráfego, desacoplando a aplicação da decisão.

## Consequences
### Positive
*   **Auditabilidade Pública**: Terceiras partes (auditores, reguladores) podem verificar recibos sem acesso a segredos.
*   **Valor Forense**: Prova matemática de autoria (Non-Repudiation).
*   **Zero-Config**: Interceptors facilitam a adoção (Monkeypatching vs Proxy).

### Negative
*   **Performance**: Ed25519 é mais lento que HMAC-SHA256 (embora desprezível para o volume atual).
*   **Gestão de Chaves**: Necessidade de gerir rotação de chaves privadas e publicação de chaves públicas.

## Compliance
Atende:
*   **NIST SP 800-57**: Requirement for non-repudiation using asymmetric keys.
*   **EU AI Act**: Article 12 (Record-keeping).
