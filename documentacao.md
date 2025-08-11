# Defensium

O sistema **Defensium** tem como principal objetivo gerenciar credenciais pessoais e sensíveis de forma segura, centralizada e confiável.  
A plataforma foi projetada para permitir o armazenamento, recuperação e auditoria de informações sigilosas — como senhas, chaves de API, tokens de acesso e outros dados confidenciais — respeitando os mais altos padrões de segurança e conformidade.  

## Principais objetivos
- Reduzir o risco de vazamento de credenciais por meio do uso de criptografia em repouso e em trânsito;
- Fornecer controle de acesso granular e autenticação segura;
- Permitir o versionamento, auditoria e rastreabilidade de alterações em dados sensíveis;
- Facilitar a integração com aplicações externas via API, de forma segura;
- Promover a automação de processos que envolvem credenciais, minimizando o uso manual e a exposição inadvertida.

---

## Índice
1. [Arquitetura](#arquitetura)
2. [Modelo de Dados](#modelo-de-dados)
3. [Configurações de Ambiente](#configurações-de-ambiente)
4. [Exportação de Variáveis](#exportação-de-variáveis)
5. [Implantação](#implantação)
6. [Versionamento com Git](#versionamento-com-git)
7. [Referências Futuras](#referências-futuras)

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

Acesso: [Supabase Dashboard](https://supabase.com/dashboard/project/jrkgryytjyfadavepemq/database/schemas)

---

## Modelo de Dados

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

---

## Configurações de Ambiente

Acesso remoto:  
```bash
ssh quintinno@192.168.1.8 -p 2222

ssh u0_a295@192.168.1.5 -p 8022
```

O sistema depende de variáveis de ambiente para acessar recursos como banco de dados, autenticação, etc.

**Passos:**
1. Remover banner e logs desnecessários (deixar apenas info e error);
2. Exportar dados de acesso ao banco de dados em variáveis de ambiente;
3. Preparar script de build do ambiente.

**Subir ambiente em segundo plano:**
```bash
nohup java -jar target/defensium-service.jar > ../defensiumlog/application.log 2>&1 &
tail -n 500 -f ../defensiumlog/application.log
```

---

## Exportação de Variáveis

As variáveis podem ser adicionadas ao seu `.bashrc`.

**Exemplo de exportação:**
```bash
# Ambiente de Desenvolvimento
export DEFENSIUM_DATABASE_HOST=db.jrkgryytjyfadavepemq.supabase.co
export DEFENSIUM_DATABASE_DATABASE=postgres
export DEFENSIUM_DATABASE_PORT=5432
export DEFENSIUM_DATABASE_USERNAME=postgres
export DEFENSIUM_DATABASE_PASSWORD=defensium

# Ambiente de Homologação (WSL)
export DEFENSIUM_DATABASE_HOST=127.0.0.1
export DEFENSIUM_DATABASE_DATABASE=postgres
export DEFENSIUM_DATABASE_PORT=5432
export DEFENSIUM_DATABASE_USERNAME=postgres
export DEFENSIUM_DATABASE_PASSWORD=postgres
```

---

##### Configurar PG no servidor TERMUX

```bash

pkg update && pkg upgrade -y

pkg install postgresql -y

initdb $PREFIX/var/lib/postgresql

pg_ctl -D $PREFIX/var/lib/postgresql start

createuser -s postgres

psql postgres

> ALTER USER postgres WITH PASSWORD 'postgres';

$ psql -U postgres -d postgres
```

## Implantação

Configurar ambiente de homologação no **WSL (servidor Priscila)** com **NGROK** para a conta `defensium.project@gmail.com`.

**Versão Java:**
```
java 21.0.6 2025-01-21 LTS
Java(TM) SE Runtime Environment (build 21.0.6+8-LTS-188)
Java HotSpot(TM) 64-Bit Server VM (build 21.0.6+8-LTS-188, mixed mode, sharing)

Apache Maven 3.8.7
Maven home: /usr/share/maven
Java version: 21.0.6, vendor: Oracle Corporation, runtime: /opt/jdk-21.0.6
Default locale: pt_BR, platform encoding: UTF-8
OS name: "linux", version: "6.8.0-71-generic", arch: "amd64", family: "unix"
```

**Instalar Java 21 (.bashrc):**
```bash
rm -rf ~/.m2
sudo apt install openjdk-21-jdk

export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH

javac -version
```

**Instalar NGROK:**

###### No TeRMUX

```bash
git clone https://github.com/Yisus7u7/termux-ngrok
```

###### No Ubuntu
```bash
ngrok.asc   | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null   && echo "deb https://ngrok-agent.s3.amazonaws.com bookworm main"   | sudo tee /etc/apt/sources.list.d/ngrok.list   && sudo apt update   && sudo apt install ngrok

ngrok config add-authtoken 317CmR9NGpSYeHA2ZW3FKyNhtAh_3e43mFBYYLCVei82mJEg

ngrok http http://localhost:8080
```

**Disponível em:**  
[DefensiumService](https://9a306c39431d.ngrok-free.app)

---

## Versionamento com Git

**Colaboradores do Projeto:**
```bash
git config --global user.name
git config --global user.email
```

- José Quintinno  
  Email: josequintinno@icloud.com  
  Email alternativo: quintinno.motog@gmail.com  

---

## Referências Futuras
*(Reservado para documentações adicionais e melhorias previstas.)*
