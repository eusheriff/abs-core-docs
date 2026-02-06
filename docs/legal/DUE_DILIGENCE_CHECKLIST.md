# Checklist de Due Diligence Técnica: Projeto ABS (V5.1)
**Alvo**: Validação de Infraestrutura, Segurança e Propriedade Intelectual.

Este checklist deve ser executado pelo time de Engenharia/Segurança do Adquirente.

## 1. Arquitetura e Kernel (Rust/WASM)
*O objetivo aqui é provar a eficiência e o isolamento do sistema.*

- [ ] **Integridade do Código-Fonte**: Verificação de que 100% do Hypervisor está em Rust (Memory Safe) e compila para `wasm32-unknown-unknown`.
- [ ] **Benchmarks de Latência**: Execução do script de stress test para validar o overhead inferior a 8ms por `tool_call`.
- [ ] **Zero-Dependency Check**: Confirmar que o Kernel WASM não possui dependências de sistema operacional (no `fs`, no `network` direto), operando apenas via imports controlados.

## 2. Criptografia e Cadeia de Confiança
*Aqui o auditor verificará se o "Liability Shield" não tem portas traseiras.*

- [ ] **Gestão de Chaves (KMS)**: Provar que o sistema não aceita chaves locais e exige integração com AWS KMS ou GCP KMS.
- [ ] **Validação de Assinatura Ed25519**: Teste de "Replay Attack" — tentar reutilizar uma assinatura de admin antiga e confirmar que o nonce do ABS a bloqueia.
- [ ] **Active Rationale Layer (ARL)**: Verificação de que o hash assinado pelo mobile inclui obrigatoriamente o código de racional selecionado pelo humano.

## 3. Motor de Imunidade e Governança
*Provar que o sistema "aprende" e escala.*

- [ ] **Simulação de Vacina P2P**: Disparar um bloqueio no Nó A e verificar o recebimento e aplicação automática no Nó B em <100ms.
- [ ] **Consenso de Quórum**: Tentar forçar uma vacina falsa a partir de um único nó e confirmar que ela é rejeitada por falta de assinaturas (2/3 de consenso).
- [ ] **Hot-Reload de Políticas**: Alterar o `abs-governance-core.yaml` em runtime e validar se o Hypervisor aplica as novas regras sem reinicialização.

## 4. Forense e Conformidade (Audit)
*A prova de que os logs são reais e válidos para seguro.*

- [ ] **Verificação da Hash-Chain**: Execução do `scripts/validate-compliance.ts` para confirmar que a corrente de hashes está íntegra desde o bloco gênese.
- [ ] **Ancoragem em L2 (Base/Optimism)**: Validar a transação na rede pública que contém o Merkle Root dos logs de auditoria das últimas 24h.
- [ ] **Mapeamento EU AI Act**: Demonstração visual de como o log do ABS atende aos requisitos de rastreabilidade do Artigo 12 e supervisão do Artigo 14.

---
**Status**: Ready for Inspection.
**Access Keys**: Provided in Data Room.
