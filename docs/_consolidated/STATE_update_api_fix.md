
### 🩹 Hotfix: API Schema Mismatch (400 Bad Request)
- **Problem**: API rejeitava requests com "400 Bad Request".
- **Root Cause**: Extensão enviava JSON incompleto. O Backend exige `event_id`, `source`, `tenant_id` e `occurred_at`.
- **Fix**: Adicionado gerador de UUID e campos mandatórios no payload.
- **Ação**: Repackaged `v0.0.13` (sobrescreveu o anterior).
- **Próximo**: Re-upload no Marketplace (mesma versão, arquivo novo).
