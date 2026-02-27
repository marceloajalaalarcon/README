# Audit Trail Completo + Sistema de Permissões Dinâmico

## Contexto e Problema

### Audit Trail — Gaps Identificados
O [audit_trail](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs#484-514) existe e [log_audit()](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs#1146-1165) funciona, mas há buracos críticos:

| Operação | Status Atual | Problema |
|---|---|---|
| Venda concluída | ✅ Logado | Só loga `items_count`, não lista os itens com preço/qtd |
| Cancelamento de venda | ❌ **Não logado** | [cancel_sale()](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/state.rs#320-325) não chama [log_audit](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs#1146-1165) |
| Remoção de item | ❌ **Não logado** | [remove_item()](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/state.rs#213-236) no core não é auditado |
| Supervisor de cancelamento | ❌ **Não capturado** | ID do supervisor não é registrado |
| Movimentos de estoque | ✅ Gravado na tabela [stock_movements](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs#638-674) | [get_stock_movements](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs#638-674) retorna `Ok(vec![])` — frontend nunca vê |
| Ajuste de estoque com supervisor | 🔶 Parcial | Supervisor ID passado mas não gravado no audit |
| Login / Logout | ✅ Login logado | Logout não logado |
| Abertura/fechamento de caixa | ✅ Em [audit_trail](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs#484-514) | Falta terminal_id e saldo por método |

### Sistema de Permissões — Problema Atual
```rust
// auth.rs — hardcoded, exige rebuild para qualquer mudança
pub fn allowed_routes(&self) -> Vec<&'static str> {
    match self {
        UserRole::Admin => vec!["/dashboard", "/vendas", ...],
        UserRole::Caixa => vec!["/vendas"],
    }
}
```
Para adicionar uma rota nova, criar um cargo novo, ou mudar o que alguém pode fazer → **rebuild obrigatório**. O cliente não consegue adaptar o sistema sozinho.

---

## Proposed Changes

### Parte 1 — Audit Trail Completo

---

#### [MODIFY] [db.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs)

**1.1 — [finalize_sale](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs#1039-1124): incluir itens completos na auditoria**

```diff
- "items_count": sale.items.len(),
+ "items": sale.items.iter().map(|i| serde_json::json!({
+     "product_id": i.product_id,
+     "product_name": i.product_name,
+     "barcode": i.barcode,
+     "quantity": i.quantity as f64 / 1000.0,
+     "unit_price": i.unit_price as f64 / 100.0,
+     "total": i.total as f64 / 100.0,
+ })).collect::<Vec<_>>(),
```

**1.2 — `cancel_sale_by_user`: adicionar audit log**
Atualmente só atualiza `payment_status = 'cancelled'` sem auditoria. Adicionar:
```sql
INSERT INTO audit_trail (tenant_id, user_id, action, entity_type, entity_id, old_values, new_values, reason)
VALUES (..., 'CANCEL', 'SALE', sale_id, old_state, new_state, reason)
```
Incluindo: `cancelled_by_user_id`, `cancelled_at`, `authorized_by_supervisor_id` (se existir).

**1.3 — [adjust_stock](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs#578-637): gravar `supervisor_id` no audit**
```diff
serde_json::json!({
    "quantity": new_quantity,
    "change": new_quantity - old_quantity,
    "reason": reason,
+   "authorized_by": supervisor_id,
+   "terminal_id": terminal_id,
})
```

**1.4 — [get_stock_movements](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs#638-674): corrigir a query real**
O método existe no [db.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs) mas o handler no [commands.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs) retorna `Ok(vec![])`. Corrigir para fazer join com [products](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs#485-529) e [users](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs#676-692):
```sql
SELECT sm.*, p.name as product_name, u.name as user_name
FROM stock_movements sm
JOIN products p ON sm.product_id = p.id
LEFT JOIN users u ON sm.user_id = u.id
WHERE sm.tenant_id = $1 AND sm.product_id = $2
ORDER BY sm.created_at DESC LIMIT $3
```

**1.5 — Nova função: `get_audit_trail_filtered`**
Permitir filtros por: `entity_type`, `date_from`, `date_to`, `user_id`, `action`. Retorna JSON paginado.

---

#### [MODIFY] [commands.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs)

**1.6 — [cancel_sale](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/state.rs#320-325): chamar audit no backend**
```rust
// Adicionar ao cancel_sale command:
db.log_audit(
    &operator_id,
    "CANCEL",
    "SALE",
    &sale_id,
    Some(&items_snapshot), // itens da venda antes do cancelamento
    Some(&json!({ "reason": reason, "authorized_by": supervisor_id })),
    None,
).await;
```

**1.7 — [get_stock_movements](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs#638-674): remover o `Ok(vec![])` e usar a query real**

**1.8 — Novo command: `get_audit_filtered`**
Expor filtros ao frontend para a tela de auditoria.

---

### Parte 2 — Sistema de Permissões Dinâmico

> [!IMPORTANT]
> Esta é a mudança mais significativa. A ideia é manter os `UserRole` existentes no Rust (para não quebrar autenticação), mas **mover tudo que é permission/route para o banco de dados**, lido em runtime.

---

#### [NEW] [004_permissions.sql](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/migrations/004_permissions.sql)

Novo schema para o sistema de permissões:

```sql
-- Permissões atômicas disponíveis no sistema
CREATE TABLE IF NOT EXISTS permissions (
    id TEXT PRIMARY KEY DEFAULT gen_random_uuid()::TEXT,
    name TEXT UNIQUE NOT NULL,          -- Ex: 'sale.cancel', 'stock.adjust'
    description TEXT NOT NULL,
    category TEXT NOT NULL,             -- 'vendas', 'estoque', 'relatorios', 'admin'
    created_at TIMESTAMP DEFAULT NOW()
);

-- Cargos configuráveis pelo cliente
CREATE TABLE IF NOT EXISTS roles (
    id TEXT PRIMARY KEY DEFAULT gen_random_uuid()::TEXT,
    tenant_id TEXT NOT NULL,
    name TEXT NOT NULL,                 -- Ex: 'Caixa', 'Gerente de Noite'
    description TEXT,
    is_system BOOLEAN DEFAULT FALSE,    -- TRUE = cargo padrão, não pode deletar
    allowed_routes TEXT[] NOT NULL DEFAULT '{}',
    default_route TEXT NOT NULL DEFAULT '/vendas',
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(tenant_id, name)
);

-- Vínculo cargo ↔ permissões
CREATE TABLE IF NOT EXISTS role_permissions (
    role_id TEXT NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    permission_name TEXT NOT NULL REFERENCES permissions(name) ON DELETE CASCADE,
    PRIMARY KEY (role_id, permission_name)
);

-- Vínculo usuário → cargo personalizado (sobrescreve o role do users)
-- Permite: "este usuário específico tem o cargo X dentro deste tenant"
ALTER TABLE users ADD COLUMN IF NOT EXISTS role_id TEXT REFERENCES roles(id);
```

**Permissões pré-definidas (seed):**

| Permissão | Categoria | Descrição |
|---|---|---|
| `sale.create` | vendas | Iniciar uma venda |
| `sale.cancel.own` | vendas | Cancelar própria venda |
| `sale.cancel.any` | vendas | Cancelar venda de qualquer operador |
| `sale.discount` | vendas | Aplicar desconto |
| `sale.view` | vendas | Ver histórico de vendas |
| `stock.view` | estoque | Ver estoque |
| `stock.adjust` | estoque | Ajustar estoque |
| `stock.movements.view` | estoque | Ver movimentos de estoque |
| `cashier.open` | caixa | Abrir caixa |
| `cashier.close` | caixa | Fechar caixa |
| `cashier.withdrawal` | caixa | Fazer sangria |
| `report.sales` | relatorios | Ver relatório de vendas |
| `report.stock` | relatorios | Ver relatório de estoque |
| `report.audit` | relatorios | Ver trilha de auditoria |
| `user.manage` | admin | Criar/editar usuários |
| `role.manage` | admin | Criar/editar cargos e permissões |
| `product.create` | produtos | Cadastrar produto |
| `product.edit` | produtos | Editar produto |
| `product.delete` | produtos | Excluir produto |
| `config.system` | admin | Alterar configurações do sistema |

---

#### [MODIFY] [db.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs)

**2.1 — `get_user_permissions(user_id)` → lê permissões do banco**
```rust
pub async fn get_user_permissions(&self, user_id: &str) -> DbResult<UserPermissions> {
    // 1. Busca role_id do usuário (se tiver role_id, usa; senão usa o role string mapeado)
    // 2. Busca role_permissions JOIN permissions
    // 3. Retorna allowed_routes, default_route, permission_names[]
}
```

**2.2 — `seed_default_roles_and_permissions()`**
Chamada no init para garantir que as permissões padrão sempre existam (idempotente com ON CONFLICT DO NOTHING).

**2.3 — Funções CRUD para cargos (para tela de admin)**
- `list_roles() -> Vec<Role>`
- `create_role(name, description, allowed_routes, default_route) -> Role`
- `update_role(role_id, ...) -> ()`
- `delete_role(role_id) -> ()` (bloquear se `is_system = true`)
- `set_role_permissions(role_id, [permission_names]) -> ()`
- `assign_role_to_user(user_id, role_id) -> ()`

---

#### [MODIFY] [commands.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs)

**2.4 — Novo command: `get_my_permissions(user_id)`**
Chamado após o login para carregar permissões dinâmicas:
```rust
#[tauri::command]
pub async fn get_my_permissions(state, user_id: String) -> Result<UserPermissionsDisplay, String>
// Retorna: { allowed_routes, default_route, permissions: ["sale.cancel", "stock.adjust", ...] }
```

**2.5 — Commands CRUD de cargos/permissões (para tela admin)**
- `list_roles`
- `create_role`
- `update_role`
- `delete_role`
- `list_permissions` (lista todas as permissões disponíveis)
- `set_role_permissions`
- `assign_role_to_user`

---

#### [NEW] Tela de Administração de Permissões no Frontend

**2.6 — `/configuracoes/permissoes`**
Página SvelteKit acessível apenas para Admin + `role.manage`:

```
┌─────────────────────────────────────────────────┐
│  Cargos              │  Permissões do Cargo      │
│ ─────────────────    │  ─────────────────────    │
│ [+] Caixa            │  ☑ sale.create            │
│     Supervisor Caixa │  ☑ sale.cancel.own        │
│     Estoquista       │  ☐ sale.cancel.any        │
│     Admin            │  ☐ sale.discount          │
│  [+ Novo Cargo]      │  ☑ cashier.open           │
│                      │  [Rotas: /vendas]         │
│                      │  [Rota Padrão: /vendas]   │
└─────────────────────────────────────────────────┘
```

---

#### [MODIFY] [+layout.svelte](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/routes/(app)/+layout.svelte)

**2.7 — Carregar permissões do banco após login**
```typescript
// Após login bem-sucedido:
const perms = await invoke('get_my_permissions', { userId: user.id });
permissionsStore.set(perms);
// Usar perms.allowed_routes para navegação
// Usar perms.permissions.includes('sale.cancel.any') para gates de UI
```

**2.8 — Remover referências a `user.allowed_routes` hardcoded**
Substituir por `$permissionsStore.allowed_routes` lido do banco.

---

#### [NEW] [permissions.ts](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/lib/stores/permissions.ts)

```typescript
interface PermissionsState {
    loaded: boolean;
    allowed_routes: string[];
    default_route: string;
    permissions: string[];  // ex: ['sale.create', 'cashier.open']
}

export const permissionsStore = createPermissionsStore();
export function hasPermission(perm: string): boolean { ... }
```

---

## Plano de Verificação

### Testes Automatizados

**Rodar testes existentes (não quebrar):**
```bash
cd zenith_core
cargo test
```
Cobertura atual: auth, CRDT. Os novos módulos (permissions, audit) devem ter testes adicionados.

**Novos testes unitários (Rust):**
```rust
// zenith_core/src/db.rs — na seção #[cfg(test)]
#[tokio::test]
async fn test_audit_cancel_sale_logs_items() { ... }

#[tokio::test]  
async fn test_get_user_permissions_from_db() { ... }

#[tokio::test]
async fn test_permission_change_takes_effect_without_rebuild() { ... }
```

### Verificação Manual pelo Admin

1. **Login como Admin** → ir em `/configuracoes/permissoes`
2. Criar um cargo chamado "Caixa Noturno" com apenas `sale.create`, `cashier.open`
3. **Sem rebuild**, atribuir o cargo a um usuário
4. Logar como esse usuário → confirmar que só vê `/vendas` e não tem botão de cancelamento
5. Voltar ao Admin → adicionar `sale.cancel.own` ao cargo
6. Relogar como o usuário → confirmar que agora tem o botão de cancelamento

**Validar Audit Trail:**
1. Fazer uma venda completa → ir em Trilha de Auditoria → confirmar que aparece com todos os itens (nome, quantidade, preço)
2. Cancelar uma venda → confirmar que aparece o evento `CANCEL` com quem cancelou e quem autorizou
3. Ajustar estoque com supervisor → confirmar que aparece `STOCK_ADJUST` com `authorized_by` preenchido
4. Abrir/Fechar caixa → confirmar que aparece com saldo de abertura e fechamento

---

## Decisões de Design

> [!IMPORTANT]
> **Compatibilidade retroativa:** Os `UserRole` existentes no Rust continuam sendo usados para autenticação e são mapeados para cargos do banco na primeira inicialização. Não há breaking change no login.

> [!NOTE]
> **Cache de permissões:** Permissões são carregadas do banco **uma vez por sessão** (após login). Para que uma mudança de cargo tome efeito imediato, o usuário precisa relogar ou o Admin pode usar `kill_session` para forçar. Isso é seguro e simples.

> [!NOTE]
> **Sem JWT/Bearer token:** O sistema é Local-First. Permissões ficam em `sessionStorage` no frontend e são revalidadas periodicamente via heartbeat. O backend sempre valida [tenant_id](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs#130-135) em todas as queries.
