# Manual de Integração Técnica: ABS-Insure Connect
**Versão**: 1.0 (Protocolo Soberano)
**Público-alvo**: Engenheiros de Dados, Arquitetos de Segurança e Atuários.

---

## 1. Conectividade e Autenticação
A comunicação com o consórcio ABS é feita via gRPC ou REST API protegida por enclaves de hardware.

*   **Endpoint Principal**: `https://api.abs-governance.io/v1/insurance`
*   **Autenticação**: Baseada em MTLS (Mutual TLS) com certificados emitidos pela raiz da DAO.
*   **Headers Obrigatórios**:
    *   `X-Insurance-Partner-ID`: Identificador da seguradora.
    *   `X-ABS-Signature`: Assinatura do payload via chave privada do parceiro.

---

## 2. Ingestão do REI (Risk Exposure Index)
O REI é a métrica principal para o cálculo do prêmio dinâmico. Ele deve ser consultado em tempo real ou via Webhooks.

### Consulta de Risco por Frota
```bash
# Exemplo de consulta via cURL
curl -X GET "https://api.abs-governance.io/v1/insurance/fleet-risk/FLEET_ID" \
     -H "Authorization: Bearer <INSURANCE_TOKEN>"
```

**Resposta (JSON):**
```json
{
  "fleet_id": "CORP_7721",
  "rei_score": 12.4,
  "insurability_grade": "A+",
  "active_enclaves": 500,
  "last_audit_anchor": "2026-02-05T19:42:00Z",
  "anomalies_24h": 0
}
```

---

## 3. Validação de Sinistro (Claims Proof)
Em caso de incidente, o sistema de sinistros da seguradora deve solicitar a **Prova Forense Imutável**. O ABS fornece um pacote criptográfico que serve como evidência legal.

### Endpoint de Verificação
`POST /v1/insurance/verify-incident`

**Payload de Requisição:**
*   `incident_hash`: O hash da transação suspeita.
*   `agent_did`: DID do agente envolvido.

**Payload de Resposta (The Proof Bundle):**
*   **ARL Proof**: Escolha racional do humano registrada no momento da ação.
*   **L2 Proof**: Link da transação em Camada 2 provando que o log existia antes do incidente ser relatado.
*   **Kernel State**: Dump assinado pelo Enclave provando que as políticas estavam ativas.

---

## 4. Implementação da Lógica de Cálculo (Python/Node.js)
A seguradora deve implementar o script de ajuste de prêmio conforme a tabela de desconto do YAML V6.5.

```python
# Exemplo de lógica para o motor de precificação da seguradora
def calculate_dynamic_premium(base_premium, rei_score):
    if rei_score <= 10.0:
        discount = 0.35 # 35% de redução
    elif rei_score <= 15.0:
        discount = 0.20 # 20% de redução
    elif rei_score > 25.1:
        return None # RISCO RECUSADO - BLOQUEIO DE MERCADO
    else:
        discount = 0.05
    
    return base_premium * (1 - discount)

# Integração com a Clearing House
current_premium = calculate_dynamic_premium(500000, 12.4)
print(f"Novo Prêmio Ajustado ABS: ${current_premium}")
```

---

## 5. Webhooks de Alerta Crítico
A seguradora deve configurar um endpoint para receber alertas de "**Revogação de DID**". Se um agente perde o selo ABS por má conduta, a cobertura deve ser suspensa em milissegundos.

| Evento | Descrição | Ação Recomendada |
| :--- | :--- | :--- |
| `ABS.IDENTITY_REVOKED` | Agente banido por tentativa de bypass. | Suspensão imediata da cobertura. |
| `ABS.REI_THRESHOLD_EXCEEDED` | Risco da frota subiu acima de 25. | Re-precificação ou aviso de risco. |
| `ABS.ENCLAVE_OFFLINE` | Perda de integridade física. | Invalidar logs do período offline. |
