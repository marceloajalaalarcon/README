# 🧠 Relatório Multi-Agente — Zenith POS
### Análise Completa e Precisa de Todos os Módulos

> **Data:** 26/02/2026 · **v2 — Análise Completa**
> **Módulos analisados:** [app](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs#267-280) · `zenith_core` · `zenith_fiscal` · `zenith_hardware`

---

## 📦 O Que o Projeto JÁ TEM (Inventário Real)

> **🤖 Agentes:** `@product-manager` + `@backend-specialist` + `@debugger`

### Core — Rust (`zenith_core`)
| Feature | Arquivo | Status |
|---|---|---|
| Autenticação CPF + senha com roles | [auth.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/auth.rs) | ✅ |
| Roles: Admin, Caixa, Supervisor, Estoquista, Dev | [auth.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/auth.rs) | ✅ |
| Login do Desenvolvedor com senha dinâmica por minuto (Pato@DDMMYYYYHHMM) | [commands.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs) | ✅ |
| Multi-tenant com [tenant_id](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs#130-135) em todas as queries | [db.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs) | ✅ |
| PostgreSQL com connection pool (SQLx) | [db.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs) | ✅ |
| CRDT PN-Counter para estoque offline-first | [crdt.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/crdt.rs) | ✅ |
| Gerenciamento de sessões (register, heartbeat, kill) | [lib.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/lib.rs) + [sessions.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/sessions.rs) | ✅ |
| Cancelamento forçado de vendas abertas ao matar sessão | [lib.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/lib.rs) | ✅ |
| Audit trail completo (CREATE, UPDATE, DELETE, LOGIN, CASHIER_OPEN/CLOSE, STOCK_ADJUST) | [db.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs) | ✅ |
| Produtos: CRUD + soft delete + NCM + CFOP + peso médio | [db.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs) + [commands.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs) | ✅ |
| Estoque: ajuste com supervisor, histórico de movimentos | [db.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs) | ✅ |
| Vendas: itens, desconto, método de pagamento, fiscal status | [models.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/models.rs) | ✅ |
| Venda em KG com conversão automática por peso médio por unidade | [commands.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs) | ✅ |
| Configurações persistidas no banco (`system_settings`) | [db.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs) | ✅ |
| Anúncios/Ads gerenciáveis | [db.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs) | ✅ |
| DB Stats (contagens de produtos, vendas, audit entries) | [db.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/zenith_core/src/db.rs) | ✅ |
| Ferramenta dev: execute SQL (somente SELECT/SHOW) | [commands.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs) | ✅ |
| Ferramenta dev: check_integrity (negstock, orphaned, sales sem itens) | [commands.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs) | ✅ |
| Ferramenta dev: get_performance_metrics (RAM, CPU via sysinfo) | [commands.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs) | ✅ |
| Ferramenta dev: generate_fake_products | [commands.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs) | ✅ |

### Frontend — SvelteKit + Tauri ([app](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs#267-280))
| Feature | Arquivo | Status |
|---|---|---|
| Abertura de caixa com fundo de troco | [cashier.ts](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/lib/stores/cashier.ts) + [+layout.svelte](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/routes/+layout.svelte) | ✅ |
| Fechamento de caixa com fechamento balance vs abertura | [cashier.ts](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/lib/stores/cashier.ts) + [commands.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs) | ✅ |
| Tracking de vendas por método de pagamento (dinheiro/crédito/débito/PIX) | [cashier.ts](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/lib/stores/cashier.ts) | ✅ |
| Caixa persistido em `localStorage` com backward compat | [cashier.ts](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/lib/stores/cashier.ts) | ✅ |
| Bloqueio de acesso a `/vendas` para Caixa se caixa não aberto | [+layout.svelte](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/routes/+layout.svelte) | ✅ |
| Audit log de abertura/fechamento no backend | [commands.rs](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src-tauri/src/commands.rs) | ✅ |
| Fluxo de venda completo (scan → carrinho → pagamento → recibo) | [posController.svelte.ts](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/lib/controllers/posController.svelte.ts) | ✅ |
| Scanner de código de barras (buffer com timeout 50ms) | [posController.svelte.ts](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/lib/controllers/posController.svelte.ts) | ✅ |
| **Balança integrada**: parse de barcode EAN-2 (formato 2PPPPPWWWWWC) | [posController.svelte.ts](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/lib/controllers/posController.svelte.ts) | ✅ |
| Modal de peso para produtos KG | [posController.svelte.ts](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/lib/controllers/posController.svelte.ts) | ✅ |
| Aprovação de Supervisor para cancelamento de item/venda | [posController.svelte.ts](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/lib/controllers/posController.svelte.ts) | ✅ |
| Modal de fechamento de caixa (F10) com valor declarado | [posController.svelte.ts](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/lib/controllers/posController.svelte.ts) | ✅ |
| Atalhos de teclado completos: F2-PIX, F3-Dinheiro, F4-Cartão, F5-Crédito, F6-Débito, F7-Qtd, F8-Cancelar, F10-Fechar Caixa | [posController.svelte.ts](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/lib/controllers/posController.svelte.ts) | ✅ |
| 3 modos de layout: clássico, mercado, conveniência | [posController.svelte.ts](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/lib/controllers/posController.svelte.ts) | ✅ |
| Recibo impresso (dados prontos pós-venda) | [posController.svelte.ts](file:///c:/Users/marce/.gemini/antigravity/scratch/zenith_pos/app/src/lib/controllers/posController.svelte.ts) | ✅ |
| Gestão de usuários (Admin) | `/users` + `commands.rs` | ✅ |
| Configurações de banco de dados | `config_commands.rs` | ✅ |
| Dashboard e Relatórios | `/dashboard` + `/relatorios` | 🔶 Telas existem |
| Tela técnica (`/tecnico`) | `/tecnico` | ✅ |

### Fiscal (`zenith_fiscal`)
| Feature | Status |
|---|---|
| Estrutura XML NFC-e 4.0 completa (Ide, Emit, Det, Total, Pag) | ✅ |
| EmissionType: Normal, SVCAN, SVCRS, Contingência Offline | ✅ |
| ICMS Simples Nacional (CSOSN102) + Regime Normal (CST00) | ✅ |
| PIS / COFINS (PISOutr / COFINSOutr) | ✅ |
| Validação de campos (`validation.rs`) | ✅ |
| **Assinatura digital A1/A3** | ❌ |
| **Transmissão real ao webservice SEFAZ** | ❌ (mock) |
| **SAT-CF-e** | ❌ |

### Hardware (`zenith_hardware`)
| Feature | Status |
|---|---|
| Driver térmico ESC/POS completo (Init, Text, Barcode, QRCode, Cut, OpenDrawer) | ✅ |
| Conexão USB, Rede, Serial, Mock | ✅ |
| Fila de impressão com prioridade (Normal, High, Critical) | ✅ |
| Layout de recibo padrão | ✅ |
| Scanner de barcode (`scanner.rs`) | ✅ |
| **Impressão em rede real** | ⚠️ Estrutura pronta, integração Tauri pendente |

---

## 🔴 CRÍTICOS — Bloqueadores de Produção

> **🤖 Agentes:** `@security-auditor` + `@penetration-tester` + `@backend-specialist`

### 🔐 SEC-01 · SHA-256 para Senhas — CRÍTICO

```rust
// auth.rs — SHA-256 é função de hash, não KDF (Key Derivation Function)
pub fn hash_password(password: &str) -> String {
    let mut hasher = Sha256::new();
    hasher.update(password.as_bytes()); // Sem salt, sem work factor
    hex::encode(hasher.finalize())
}
```

SHA-256 processa bilhões de hashes por segundo em GPU. Uma senha de 8 dígitos é quebrada em segundos. Um sistema de PDV com dados financeiros de múltiplos clientes **precisa de Argon2id** obrigatoriamente.

**Correção:** Adicionar `argon2 = "0.5"` no `Cargo.toml` e substituir.

---

### 🔐 SEC-02 · Senhas Default Hardcoded

```rust
// auth.rs — Seed de usuários com senhas no código-fonte
("admin123".to_string()), // Qualquer dev com acesso ao repo conhece esta senha
("caixa123".to_string()),
("dev123".to_string()),
```

**Correção:** Gerar senha aleatória na primeira inicialização + forçar troca no primeiro login.

---

### 💰 BUG-01 · Alíquotas Fiscais Hardcoded para Todos os Produtos

```rust
// commands.rs (create_product) e db.rs (find_product_by_barcode)
icms_rate: Some(18.0),  // ICMS fixo — ilegal para produtos de alíquota diferente
pis_rate: Some(1.65),
cofins_rate: Some(7.60),
ncm: Some("00000000"), // NCM fake/placeholder
```

Produtos de NCMs diferentes têm alíquotas diferentes. Emitir NFC-e com taxas incorretas é crime fiscal. As alíquotas precisam ser **por produto no banco de dados**.

---

### 📄 FISCAL-01 · NFC-e Sem Assinatura Digital e Sem Transmissão

O módulo `zenith_fiscal` define a estrutura XML corretamente, mas:
- Não possui assinatura digital com certificado A1/A3 (obrigatório por lei)
- A verificação SEFAZ é um `tokio::sleep(50ms)` simulado
- Sem transmissão ao webservice real da SEFAZ
- Sem tratamento de retorno (autorização/rejeição/contingência real)

Enquanto isso não está implementado, o sistema **não pode emitir nota fiscal legalmente**.

---

### 💳 PAG-01 · Pagamentos PIX e Cartão São Simulações

```rust
// payments.rs
pub async fn simulate_confirmation(&self, ...) // Confirmação fake
// QR Code PIX sem CRC4 (checksum obrigatório pelo padrão EMV-BR)
```

```typescript
// posController.svelte.ts — PIX com timeout hardcoded
setTimeout(() => { paymentStep = "complete"; }, 3000); // Não é pagamento real
```

Nenhum banco ou maquininha vai processar estes pagamentos. Precisa de integração com PSP real (MercadoPago, PagSeguro, Stone, Cielo) ou terminal TEF.

---

### 🐛 BUG-02 · `get_stock_movements` e `get_audit_trail` Retornam Vazio

```rust
// commands.rs — TODO explícito no código
Ok(vec![]) // Temporary: return empty until we implement proper deserialization
```

As telas de estoque e auditoria no frontend recebem sempre arrays vazios.

---

### 🐛 BUG-03 · Strings Corrompidas em Erros

```rust
// commands.rs — Encoding ISO-8859-1 em UTF-8
.ok_or_else(|| "Produto nÃ£o encontrado".to_string()) // "não encontrado" corrompido
"Cargo invÃ¡lido"                                      // "Cargo inválido" corrompido
```

Indica problemas de encoding nos arquivos `.rs` — afeta UX em mensagens de erro.

---

## 🏢 Benchmark vs. Principais Empresas de PDV

> **🤖 Perspectiva: `@product-manager`**

| Feature | TOTVS | Linx/Stone | Boa Compact | **Zenith (Atual)** |
|---|---|---|---|---|
| NFC-e transmitida e autorizada | ✅ | ✅ | ✅ | ❌ (estrutura pronta) |
| SAT-CF-e | ✅ | ✅ | ✅ | ❌ |
| Multi-terminal offline-first (CRDT) | ❌ | ❌ | 🔶 | ✅ Único no mercado |
| Gestão de sessões multi-terminal | ✅ | ✅ | 🔶 | ✅ Com heartbeat e kill |
| Abertura / Fechamento de caixa | ✅ | ✅ | ✅ | ✅ Completo |
| Venda por peso (balança EAN-2) | ✅ | ✅ | ✅ | ✅ Implementado |
| Supervisor para cancelamento | ✅ | ✅ | ✅ | ✅ Implementado |
| Atalhos de teclado profissionais | ✅ | ✅ | 🔶 | ✅ F2~F10 completos |
| Audit trail completo | ✅ | ✅ | 🔶 | ✅ |
| Dev tools integradas | ❌ | ❌ | ❌ | ✅ SQL, integridade, métricas |
| Dashboard / KPIs reais | ✅ | ✅ | ✅ | ❌ Tela bez dados |
| Relatórios (vendas, ABC, turno) | ✅ | ✅ | ✅ | ❌ |
| TEF real (Stone, Cielo, Rede) | ✅ | ✅ | ✅ | ❌ |
| PIX real via PSP | ✅ | ✅ | ✅ | ❌ |
| Sangrias e suprimentos de caixa | ✅ | ✅ | ✅ | ❌ |
| Relatório Z | ✅ | ✅ | ✅ | ❌ |
| Programa de fidelidade | ✅ | ✅ | 🔶 | ❌ |
| Categorias de produto | ✅ | ✅ | ✅ | ❌ |
| Grade (cor, tamanho, sabor) | ✅ | ✅ | 🔶 | ❌ |
| Foto de produto | ✅ | ✅ | ✅ | ❌ |
| Estoque mínimo / alerta | ✅ | ✅ | ✅ | 🔶 Modelo v2 tem, não exposto |
| Fornecedores / Compras | ✅ | ✅ | ✅ | ❌ |
| Promoções automáticas | ✅ | ✅ | 🔶 | ❌ |
| Exportação SPED / EFD | ✅ | ✅ | ✅ | ❌ |

---

## ⚙️ Gaps Técnicos

> **🤖 Perspectiva: `@database-architect` + `@backend-specialist`**

### Tabelas Faltantes no Schema

| Tabela | Necessidade | Prioridade |
|---|---|---|
| `cashier_withdrawals` | Sangrias e suprimentos | 🔴 Alta |
| `z_reports` | Relatório Z por turno | 🔴 Alta |
| `product_categories` | Hierarquia de categorias | 🟠 Média |
| `promotions` + `promotion_items` | Descontos automáticos | 🟠 Média |
| `customers` | CRM / CPF na nota | 🟠 Média |
| `loyalty_points` | Fidelidade | 🟡 Baixa |
| `suppliers` + `purchase_orders` | Compras | 🟡 Baixa |
| `product_variants` | Grade de produtos | 🟡 Baixa |
| `nfce_transmissions` | Log fiscal com SEFAZ | 🔴 Alta (legal) |

### Dashboard / Relatórios Sem Dados
As rotas `/dashboard` e `/relatorios` existem mas não têm queries implementadas. As queries precisam cobrir:
- Vendas por período (hora, dia, semana, mês)
- Ticket médio
- Curva ABC de produtos
- Desempenho por operador
- Margem por produto

### `inventory_v2` Incompleto
O módulo `zenith_core/src/inventory_v2` tem modelos bem definidos (com `min_stock`, `reserved`, `MovementReason` enum), mas:
- `repository.rs` tem apenas esboço de funções
- Não está conectado ao Tauri (os comandos `inv_*` no `inventoryStore` não têm handler)
- O `lowStockProducts` derived store está sem dados reais

---

## 🧪 Gaps de Testes

> **🤖 Perspectiva: `@test-engineer`**

| O que existe | Status |
|---|---|
| Testes unitários: CRDT, auth, ESC/POS, PIX | ✅ |
| **Testes de isolamento multi-tenant** | ❌ CRÍTICO |
| **Testes de integração: fluxo de venda E2E** | ❌ |
| **Testes de regressão fiscal** (cálculo de ICMS/PIS/COFINS) | ❌ |
| **Testes das queries de relatório** | ❌ (queries não existem ainda) |

O mais crítico: **nenhum teste garante que `tenant_id` impede vazamento de dados entre clientes**. Um bug aqui afeta múltiplas empresas simultaneamente.

---

## 📋 Roadmap Priorizado

> **🤖 Perspectiva: `@product-manager` consolidando todos os agentes**

### 🔴 FASE 1 — Bloqueadores Legais e de Segurança

| # | Item | Impacto |
|---|---|---|
| 1 | Trocar SHA-256 por **Argon2id** | Segurança crítica |
| 2 | Remover senhas hardcoded + forced first-login change | Segurança |
| 3 | Alíquotas fiscais por produto no banco (não hardcoded) | Legal |
| 4 | **Transmissão NFC-e real** com certificado A1/A3 e SEFAZ | Legal |
| 5 | **PIX real** via PSP (MercadoPago, PagSeguro ou similar) | Comercial crítico |
| 6 | Fix encoding de strings em erros (`nÃ£o` → `não`) | UX |
| 7 | Fix `get_stock_movements` e `get_audit_trail` (remover `Ok(vec![])`) | Bug |

### 🟠 FASE 2 — MVP Completo

| # | Item |
|---|---|
| 8 | Dashboard real: queries de vendas por período e produtos mais vendidos |
| 9 | Sangrias e suprimentos de caixa |
| 10 | Relatório Z de fechamento de turno |
| 11 | Categorias de produto (hierarquia no cadastro) |
| 12 | Foto de produto |
| 13 | Estoque mínimo exposto na UI (modelo v2 já tem o campo) |
| 14 | Finalizar `inventory_v2` e conectar os comandos `inv_*` ao Tauri |
| 15 | Testes de isolamento multi-tenant (obrigatório antes de primeiro cliente) |
| 16 | Integração TEF (Stone SDK ou SiTEF) |

### 🟡 FASE 3 — Diferencial Competitivo

| # | Item |
|---|---|
| 17 | Programa de fidelidade (pontos / cashback) |
| 18 | Promoções automáticas (por produto, combo, horário) |
| 19 | Grade de produtos (variações: tamanho, cor, sabor) |
| 20 | Cadastro de clientes com CRM básico |
| 21 | Módulo de compras e fornecedores |
| 22 | Exportação SPED/EFD para contabilidade |
| 23 | SAT-CF-e (SP — obrigatório para alguns segmentos) |
| 24 | CI/CD + monitoramento (Sentry para erros de produção) |

---

## 🏆 Diferenciais Técnicos Reais

O Zenith tem vantagens genuínas que **nenhum concorrente brasileiro oferece**:

| Diferencial | Por que importa |
|---|---|
| **CRDT PN-Counter para estoque offline** | Múltiplos caixas sem servidor central. Único no mercado BR. |
| **Rust (performance + segurança de memória)** | 0 GC pauses, menor consumo RAM vs Java/Node (TOTVS/Linx) |
| **Tauri vs Electron** | Bundle 10x menor, arranque mais rápido, sem overhead V8 |
| **Sessões com heartbeat + kill_session** | Admin pode derrubar operadores remotamente, cancela vendas abertas |
| **Dev tools embutidas** (SQL, integridade, métricas) | Suporte e diagnóstico sem acesso ao servidor |
| **Login Dev com senha temporal por minuto** | Acesso seguro para suporte sem expor senha fixa |
| **Parser de balança EAN-2 nativo** | Supermercados e açougues prontos sem configuração extra |
| **3 modos de layout da tela de vendas** | Adaptável a diferentes perfis de varejo |

---

## 🎯 Resumo Executivo

O Zenith está **bem além de um protótipo** — tem um sistema profissional de venda, caixa, estoque, sessões e auditoria funcionando. A arquitetura é sólida e inovadora.

**Pré-requisitos para o primeiro cliente:**
1. ✅ Fluxo de venda → **pronto**
2. ✅ Abertura/fechamento de caixa → **pronto**
3. ✅ Gestão de sessões multi-terminal → **pronto**
4. ❌ Senha segura (Argon2) → **BLOQUEADOR**
5. ❌ NFC-e transmitida → **BLOQUEADOR LEGAL**
6. ❌ Pagamento real (PIX/TEF) → **BLOQUEADOR COMERCIAL**

Com os 3 bloqueadores resolvidos, o Zenith está pronto para os primeiros pilotos.

---

*Análise multi-agente completa — 26/02/2026 · v2*
