
### 🐛 Hotfix: Sidebar not triggering Scan
- **Sintoma**: Clicar em "Scans Selected Files" mostrava toast mas não gerava relatório.
- **Causa**: O botão estava chamando `abs.scanFile` (legacy) em vez de `abs.scanWorkspace` (v0.0.12 logic).
- **Correção**: Atualizado `SidebarProvider.ts` para disparar o comando correto.
- **Ação**: O pacote `.vsix` foi recriado. Requer novo upload para o Marketplace.
