# Defensium

O sistema Defensium tem como principal objetivo gerenciar credenciais pessoais e sensíveis de forma segura, centralizada e confiável. A plataforma foi projetada para permitir o armazenamento, recuperação e auditoria de informações sigilosas — como senhas, chaves de API, tokens de acesso e outros dados confidenciais — respeitando os mais altos padrões de segurança e conformidade. Entre os principais objetivos estão:

- Reduzir o risco de vazamento de credenciais por meio do uso de criptografia em repouso e em trânsito;

- Fornecer controle de acesso granular e autenticação segura;

- Permitir o versionamento, auditoria e rastreabilidade de alterações em dados sensíveis;

- Facilitar a integração com aplicações externas via API, de forma segura;

- Promover a automação de processos que envolvem credenciais, minimizando o uso manual e a exposição inadvertida.
---

## Índice

1. [Arquitetura](#arquitetura)
2. [Configurações de Ambiente](#configurações-de-ambiente)
3. [Exportação de Variáveis](#exportação-de-variáveis)
4. [Atualização do Ambiente](#atualização-do-ambiente)
5. [Versionamento com Git](#versionamento-com-git)
6. [Referências Futuras](#referências-futuras)
7. [Modelo de Dados](#modelo)

---

## Arquitetura

### Variáveis de Banco de Dados

| Variável | Valor |
|----------|-------|
| DEFENSIUM_DATABASE_HOST     | db.jrkgryytjyfadavepemq.supabase.co |
| DEFENSIUM_DATABASE_DATABASE | postgres                             |
| DEFENSIUM_DATABASE_PORT     | 5432                                 |
| DEFENSIUM_DATABASE_USERNAME | postgres                             |
| DEFENSIUM_DATABASE_PASSWORD | defensium                            |

---

## Configurações de Ambiente

Este sistema depende de variáveis de ambiente para acessar recursos como banco de dados, autenticação, etc.

---

## Exportação de Variáveis

As variáveis podem ser adicionadas ao seu `.bashrc` (Linux/macOS) ou `.bash_profile`, conforme o shell usado.

### Exemplo de exportação

```bash
export DEFENSIUM_DATABASE_HOST=db.jrkgryytjyfadavepemq.supabase.co
export DEFENSIUM_DATABASE_DATABASE=postgres
export DEFENSIUM_DATABASE_PORT=5432
export DEFENSIUM_DATABASE_USERNAME=postgres
export DEFENSIUM_DATABASE_PASSWORD=defensium
```

## Modelo

#### Funcionalidade de Gerenciamento de Credenciais

```sql
```