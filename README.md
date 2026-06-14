# BDD2 Lab 02 — Exploração de Bases de Dados Distribuídas
### MercadoKwanza · ISPTEC · Licenciatura em Engenharia Informática · 2025/2026

> Simulação de um cluster de bases de dados distribuídas com MySQL (replicação Master-Slave),
> fragmentação horizontal/vertical e transações distribuídas com Two-Phase Commit.

---

## Grupo

| Nome | Função | Cenário |
|------|--------|---------|
| Nome do Membro 1 | Líder | — |
| Nome do Membro 2 | Desenvolvedor | — |
| Nome do Membro 3 | Desenvolvedor | — |
| Nome do Membro 4 | Desenvolvedor | — |

**Grupo:** X · **Cenário:** A ou B · **Turma:** XX

---

## Estrutura do Repositório

```
bdd2-lab02-grupoX/
│
├── docker-compose.yml          # Definição do cluster (4 contentores)
│
├── dados/
│   ├── mercadokwanza_p1.sql    # Dataset da Parte 1
│   └── mercadokwanza_p2.sql    # Dataset da Parte 2
│
├── scripts/
│   ├── transacao.py            # Transação distribuída (Two-Phase Commit)
│   ├── migracao.py             # Migração MySQL → MongoDB
│   ├── transferencia.py        # Transferência de stock entre nós (Grupo 4)
│   └── fragmentacao.sql        # Views de fragmentação horizontal e vertical
│
├── relatorio/
│   └── Lab02_GrupoX.pdf        # Relatório final em PDF
│
└── README.md                   # Este ficheiro
```

---

## Pré-Requisitos

Antes de começar, certifica-te de que tens instalado:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (versão 26+)
- [Docker Compose](https://docs.docker.com/compose/) (versão 2+)
- [Python 3.10+](https://www.python.org/downloads/)
- Bibliotecas Python:

```bash
pip install mysql-connector-python pymongo
```

> **Utilizadores de XAMPP:** se o MySQL do XAMPP estiver ativo, as portas 3306/3307
> podem estar ocupadas. Verifica as portas no `docker-compose.yml` e ajusta conforme
> necessário (o repositório já usa 3360/3361/3362 por defeito).

---

## Como Executar

### 1. Clonar o repositório

```bash
git clone https://github.com/teu-utilizador/bdd2-lab02-grupoX.git
cd bdd2-lab02-grupoX
```

### 2. Levantar o cluster

```bash
docker compose up -d
```

Na primeira execução, o Docker vai descarregar as imagens do MySQL 8.0 e MongoDB 7.0
(~500 MB). Pode demorar alguns minutos.

### 3. Verificar que os contentores estão ativos

```bash
docker ps
```

Deves ver 4 contentores com `STATUS = Up`:

| Contentor | Imagem | Porta (PC) |
|-----------|--------|------------|
| no-luanda | mysql:8.0 | 3360 |
| no-benguela | mysql:8.0 | 3361 |
| no-huambo | mysql:8.0 | 3362 |
| mongo-kwanza | mongo:7.0 | 27017 |

### 4. Verificar os dados no nó Luanda

```bash
docker exec -it no-luanda mysql -uroot -pkwanza2024 mercadokwanza \
  -e "SELECT COUNT(*) AS vendas FROM VENDA; SELECT COUNT(*) AS clientes FROM CLIENTE;"
```

### 5. Executar as transações distribuídas

```bash
python scripts/transacao.py
```

### 6. Executar a migração para MongoDB

```bash
python scripts/migracao.py
```

---

## Arquitectura do Cluster

```
┌─────────────────────────────────────────────────┐
│                  O teu Computador                │
│                                                   │
│  ┌──────────────┐    replicação    ┌───────────┐ │
│  │  no-luanda   │ ───────────────► │no-benguela│ │
│  │  (Master)    │                  │ (Slave 1) │ │
│  │  porta 3360  │ ───────────────► │ porta 3361│ │
│  └──────────────┘        │         └───────────┘ │
│                           │                       │
│                           ▼                       │
│                    ┌──────────────┐               │
│                    │  no-huambo   │               │
│                    │  (Slave 2)   │               │
│                    │  porta 3362  │               │
│                    └──────────────┘               │
│                                                   │
│  ┌──────────────┐                                 │
│  │ mongo-kwanza │  ← usado na Parte 2             │
│  │  porta 27017 │                                 │
│  └──────────────┘                                 │
└─────────────────────────────────────────────────┘
```

**Fluxo de replicação:** todas as escritas acontecem no Master (Luanda). Os Slaves
(Benguela e Huambo) leem o binary log do Master e repetem as operações automaticamente.

---

## Credenciais de Acesso

| Campo | Valor |
|-------|-------|
| Utilizador | `root` |
| Palavra-passe | `kwanza2024` |
| Base de dados | `mercadokwanza` |

> ⚠️ Estas credenciais são apenas para ambiente de laboratório local.
> Nunca uses senhas simples em ambientes de produção.

---

## Ligar via MySQL Workbench

Cria uma nova conexão para cada nó:

| Nó | Host | Porta |
|----|------|-------|
| Luanda (Master) | 127.0.0.1 | 3360 |
| Benguela (Slave 1) | 127.0.0.1 | 3361 |
| Huambo (Slave 2) | 127.0.0.1 | 3362 |

---

## Parar e Limpar o Cluster

```bash
# Parar os contentores (dados ficam guardados)
docker compose stop

# Parar e remover os contentores (dados ficam guardados nos volumes)
docker compose down

# Parar, remover contentores E apagar todos os dados
docker compose down -v
```

> `down` sem `-v` preserva os dados entre sessões.
> `down -v` apaga tudo — útil para recomeçar do zero.

---

## Uso de Inteligência Artificial

Este laboratório exige o registo de todos os usos de IA. Ver secção correspondente
no relatório (`relatorio/Lab02_GrupoX.pdf`) para o registo completo de:

- Ferramenta usada
- Prompt submetido
- Resposta obtida
- Correcções e decisões do grupo

---

## Referências

- ÖZSU, M. T.; VALDURIEZ, P. (2020). *Principles of Distributed Database Systems*. 4.ª ed. Springer.
- KLEPPMANN, M. (2017). *Designing Data-Intensive Applications*. O'Reilly Media.
- BREWER, E. A. (2000). *Towards Robust Distributed Systems*. ACM PODC.
- MySQL 8.0 Reference Manual — https://dev.mysql.com/doc/refman/8.0/en/
- MongoDB Manual 7.0 — https://www.mongodb.com/docs/manual/
- Docker Documentation — https://docs.docker.com

---

*ISPTEC · Departamento de Engenharia e Tecnologias · Base de Dados II · 2025/2026*