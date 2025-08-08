-- Cadastrar Credencial

Código              0001
Tipo Cofre          Credencial
Descrição           Conta Google
Usuário             email@gmail.com 
Senha               * * *
Link                http://www.account.google.com/auth

drop table if exists flyway_schema_history cascade;
drop table if exists tb_plataforma cascade;
drop table if exists tb_credencial cascade;
drop table if exists tb_cartao_credito cascade;

create extension if not exists pgcrypto;

create table if not exists tb_plataforma (
    codigo bigserial not null,
    nome varchar (100) not null,
    ativa boolean not null default true,
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
    constraint pk_credencial primary key (codigo),
    constraint pk_plataforma foreign key (id_plataforma) references tb_plataforma (codigo),
    constraint un_credencial unique (id_plataforma, descricao)
);

create table if not exists tb_cartao_bancario (
    codigo bigserial not null,
    codigo_publico uuid default gen_random_uuid (),
    id_plataforma bigserial not null,
    descricao varchar(100) not null,
    numero_cartao bytea not null,
    nome_titular varchar(100) not null,
    data_validade date not null,
    cvv bytea not null,
    constraint fk_plataforma foreign key (codigo) references tb_plataforma (codigo),
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

SELECT * FROM pg_extension WHERE extname = 'pgcrypto';

-- Cartão Bancário
insert into tb_cartao_bancario (id_plataforma, descricao, numero_cartao, nome_titular, data_validade, cvv) values (
    (select codigo from tb_plataforma where nome = 'Google'),
    'Cartão de Crédito (Banco do Brasil))',
     '1234567890123456',
     'José Quintinno',
     '2027-08-01',
     '123'
);

select * from tb_plataforma order by nome asc;

select * from tb_credencial order by codigo asc;

select * from tb_cartao_bancario order by codigo asc;