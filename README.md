# 🚗 Meu Corre Despachante

> Módulo de gestão interna para despachante autônomo — parte do ecossistema **Meu Corre**.

---

## O problema

Despachantes autônomos que captam clientes direto operam hoje no caos:
documentos físicos sem rastreio, prazos perdidos por falta de alerta,
clientes com saldo em aberto sem visibilidade, e processos travados
esperando papel que ninguém foi cobrar.

**Nenhuma planilha resolve isso. Precisa de sistema.**

---

## As 4 dores que este sistema resolve

### 🔴 Dor 1 — Onde está o documento físico?

Cada papel tem uma localização rastreada em tempo real.

| Status               | Significado                                 |
| -------------------- | ------------------------------------------- |
| `Com você`           | Na sua posse, com referência (ex: gaveta A) |
| `No DETRAN`          | Com número de protocolo                     |
| `No cartório`        | Com data de entrada                         |
| `Aguardando cliente` | 🚨 Processo bloqueado                       |
| `Entregue`           | Finalizado                                  |

---

### 🟡 Dor 2 — Prazo de vistoria ou licenciamento estourando

O sistema exibe contagem regressiva para cada OS e dispara alertas visuais:

- 🔴 **Crítico** — vence em menos de 5 dias
- 🟡 **Atenção** — vence em menos de 15 dias
- 🟢 **OK** — prazo confortável

Você abre o sistema de manhã e já sabe quem ligar primeiro.

---

### 🔵 Dor 3 — Controlar o que cada cliente deve

Cada OS tem seu próprio mini-financeiro:

```
Honorário combinado:    R$ 350,00
Taxas pagas por você:   R$  89,40  (DETRAN + cartório)
─────────────────────────────────
Total a receber:        R$ 439,40
Já recebido:            R$ 200,00
─────────────────────────────────
Saldo devedor:          R$ 239,40  ← visível no painel
```

Um cliente pode ter várias OS abertas. O sistema consolida o saldo total por cliente.

---

### 🟡 Dor 4 — Processos parados esperando documento

Filtro dedicado **"Aguardando documento"** mostra exatamente o que está travado,
qual documento falta, e há quantos dias está parado.
Chega de lembrar na cabeça o que está pendente.

---

## Ciclo de vida de uma Ordem de Serviço

```
Recebida → Aguardando doc. → Em andamento → Finalizando → Concluída
                ↑
         (processo parado)
         alerta automático
```

Ao concluir uma OS de transferência, o processo correspondente
no **Meu Corre Financeiro** fecha automaticamente — sem trabalho duplicado.

---

## Funcionalidades do MVP

| Módulo                        | O que faz                                        |
| ----------------------------- | ------------------------------------------------ |
| **Painel de ordens**          | Lista de OS com filtros por status e prazo       |
| **Localização de documentos** | Rastreio físico de cada papel por OS             |
| **Financeiro por OS**         | Honorário, taxas, recebido, saldo devedor        |
| **Alertas de prazo**          | Vistoria e licenciamento com contagem regressiva |
| **Fila de bloqueados**        | OS paradas esperando documento do cliente        |
| **Cadastro de clientes**      | Histórico de OS e saldo consolidado por pessoa   |
| **Gerador de PDF**            | Comprovante de OS e relatório mensal             |

---

## Tipos de serviço cobertos

- Transferência de propriedade
- 1º emplacamento (veículo 0km)
- Licenciamento anual
- Vistoria DETRAN
- Pagamento de IPVA
- Regularização de débitos
- Gravame / alienação fiduciária
- CRV digital
- Baixa de veículo

---

## Modelo de dados (entidades principais)

```
Cliente
  ├── id, nome, telefone, email
  └── OrdemServico (1:N)
        ├── id, placa, veiculo, tipo_servico
        ├── status: recebida | aguardando_doc | andamento | finalizando | concluida
        ├── vencimento_vistoria, vencimento_licenciamento
        ├── Documento (1:N)
        │     ├── tipo, localizacao, protocolo, observacao
        │     └── arquivo_url (foto/scan)
        └── LancamentoFinanceiro (1:N)
              ├── tipo: honorario | taxa_paga | recebimento
              └── valor, data, descricao
```

---

## Stack técnica

| Camada         | Tecnologia             | Por quê                                     |
| -------------- | ---------------------- | ------------------------------------------- |
| Desktop        | Python + PySide6       | Maduro, padrões de mercado, SOLID aplicável |
| Banco local    | SQLite + SQLAlchemy    | Zero configuração, funciona offline         |
| Mobile         | Flutter                | Atualizar OS em campo, foto de protocolo    |
| Relatórios     | ReportLab / WeasyPrint | PDF de comprovante e relatório mensal       |
| Nuvem (fase 3) | Supabase               | PostgreSQL + Storage + Auth gratuitos       |

---

## Arquitetura (Clean Architecture)

```
Presentation (PySide6 / Flutter)
        ↓
  Use Cases  ←── regras de negócio puras, sem dependência de UI ou banco
        ↓
  Repository  ←── interface abstrata (DIP do SOLID)
        ↓
  SQLAlchemy + SQLite
```

Quando o banco migrar de SQLite para PostgreSQL (fase 3),
**nenhuma linha de Use Case muda** — só a implementação do Repository.

---

## Roadmap

### Fase 1 — MVP Desktop (Python + SQLite)

- [ ] Modelo de dados com SQLAlchemy
- [ ] CRUD de clientes e ordens de serviço
- [ ] Rastreio de localização de documentos
- [ ] Painel com filtros de status
- [ ] Alertas de prazo (vermelho / amarelo / verde)
- [ ] Financeiro por OS com saldo automático
- [ ] Gerador de PDF (comprovante + relatório)

### Fase 2 — Mobile (Flutter)

- [ ] Consulta de OS por placa
- [ ] Atualização de status em campo
- [ ] Foto de protocolo vinculada à OS
- [ ] Notificação de prazo crítico

### Fase 3 — Nuvem + Integração

- [ ] Migração para Supabase (PostgreSQL)
- [ ] Sincronização com Meu Corre Financeiro
- [ ] Backup automático de documentos
- [ ] Acesso via browser (opcional)

---

## Integração com o ecossistema Meu Corre

```
┌─────────────────────┐     ┌──────────────────────┐
│  Meu Corre          │     │  Meu Corre           │
│  Financeiro         │◄────│  Despachante         │
│                     │     │                      │
│  Processo de venda  │     │  Ordem de serviço    │
│  Status da transfer.│     │  Docs + prazos       │
│  DUT + IPVA         │     │  Financeiro do desp. │
└─────────────────────┘     └──────────────────────┘
         ▲                            ▲
         └──────── SQLite local ──────┘
                  (mesmo banco)
```

Quando a OS de transferência é concluída no Despachante,
o processo fecha automaticamente no Financeiro.
DUT, arquivos e status sincronizados sem trabalho manual.

---

## Por que construir isso?

O mercado de despachantes no Brasil opera 100% no improviso —
WhatsApp, planilha e memória. Não existe ferramenta específica
para o despachante autônomo que capta direto.

Este sistema resolve dores reais, com stack acessível para aprender
e evoluir, aplicando SOLID e Clean Architecture desde o primeiro commit.

---

_Meu Corre Despachante — parte do universo [Meu Corre](../)_
