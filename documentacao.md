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

Acesso: https://supabase.com/dashboard/project/jrkgryytjyfadavepemq/database/schemas
---

## Configurações de Ambiente

Este sistema depende de variáveis de ambiente para acessar recursos como banco de dados, autenticação, etc.

---

## Exportação de Variáveis

As variáveis podem ser adicionadas ao seu `.bashrc`.

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
drop table if exists flyway_schema_history cascade;
drop table if exists tb_plataforma cascade;
drop table if exists tb_credencial cascade;
drop table if exists tb_cartao_bancario cascade;

create extension if not exists pgcrypto;

create table if not exists tb_plataforma (
    codigo bigserial not null,
    nome varchar (100) not null,
    ativa boolean not null default true,
    data_criacao timestamp not null default now(),
    data_edicao timestamp not null default now(),
    data_delecao timestamp not null default now(),
    constraint pk_plataforma primary key (codigo),
    constraint un_plataforma_nome unique (nome)
);

create table if not exists tb_credencial (
    codigo bigserial not null,
    codigo_publico uuid default gen_random_uuid(),
    id_plataforma bigserial not null,
    descricao varchar (100) not null,
    usuario varchar (60) not null,
    senha varchar (100) not null,
    url varchar (255) null,
    data_criacao timestamp not null default now(),
    data_edicao timestamp not null default now(),
    data_delecao timestamp not null default now(),
    constraint pk_credencial primary key (codigo),
    constraint pk_plataforma foreign key (id_plataforma) references tb_plataforma (codigo),
    constraint un_credencial unique (id_plataforma, descricao)
);

create table if not exists tb_cartao_bancario (
    codigo bigserial not null,
    codigo_publico uuid default gen_random_uuid (),
    descricao varchar(100) not null,
    numero_cartao varchar not null,
    nome_titular varchar(100) not null,
    data_validade date not null,
    cvv varchar not null,
    data_criacao timestamp not null default now(),
    data_edicao timestamp not null default now(),
    data_delecao timestamp not null default now(),
    constraint pk_cartao_bancario primary key (codigo),
    constraint un_cartao_bancario unique (numero_cartao, data_validade)
);

insert into tb_plataforma (nome) values 
    ('Google'), ('Microsoft'), ('Dribbble'), ('Amazon'), ('Twitter'), ('Meta');

insert into tb_credencial (id_plataforma, descricao, usuario, senha, url) values (
    (select codigo from tb_plataforma where nome = 'Google'),
    'Conta Google', 'defensium.project@gmail.com', 'senha-segura', 'http://www.account.google.com/auth'
);

insert into tb_credencial (id_plataforma, descricao, usuario, senha, url) values (
    (select codigo from tb_plataforma where nome = 'Microsoft'),
    'Conta Microsoft', 'defensium.project@outlook.com.br', 'senha-segura', 'https://outlook.live.com/mail/0/?prompt=select_account'
);

-- Cartão Bancário
insert into tb_cartao_bancario (descricao, numero_cartao, nome_titular, data_validade, cvv) values (
    'Cartão de Crédito (Banco do Brasil))',
     '1234567890123456',
     'José Quintinno',
     '2027-08-01',
     '123'
);

select * from tb_plataforma order by nome asc;

select * from tb_credencial order by codigo asc;

select * from tb_cartao_bancario order by codigo asc;
```