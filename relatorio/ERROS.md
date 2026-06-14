# Registo de Erros, Soluções e Uso de IA — Lab 02 BDD
### Grupo 06 · MercadoKwanza · ISPTEC 2025/2026

> Este documento regista todos os erros encontrados durante a execução do Lab 02,
> as soluções aplicadas, e o registo de uso de IA conforme exigido pelo enunciado.

---

## ERRO 1 — Docker não arrancava
**Fase:** Fase 1 — Levantar o Cluster

**Erro:**
```
unable to get image 'mysql:8.0': failed to connect to the docker API
at npipe:////./pipe/dockerDesktopLinuxEngine
```

**Causa:** O Docker Desktop não estava aberto. O motor do Docker precisa de estar
ativo antes de qualquer comando `docker compose`.

**Solução:** Abrir o Docker Desktop manualmente e aguardar o ícone ficar verde
na barra de tarefas antes de correr `docker compose up -d`.

**Prompt usado com IA (Claude):**
> "Apareceu este erro ao correr docker compose up -d: failed to connect to the
> docker API at npipe. O que significa e como resolver?"

**Resposta da IA:** Identificou que o Docker Desktop não estava em execução.
Sugeriu abrir o Docker Desktop e aguardar inicialização completa.

**O que o grupo fez:** Seguiu a sugestão. Problema resolvido.

---

## ERRO 2 — Sem acesso à internet para descarregar imagens
**Fase:** Fase 1 — Levantar o Cluster

**Erro:**
```
dial tcp: lookup registry-1.docker.io: no such host
```

**Causa:** O Docker Desktop não conseguia resolver o DNS para aceder ao Docker Hub
e descarregar as imagens do MySQL e MongoDB.

**Solução:** Nas definições do Docker Desktop → Docker Engine, adicionar servidores
DNS do Google:
```json
{
  "dns": ["8.8.8.8", "8.8.4.4"]
}
```
Depois clicar "Apply & Restart".

**Prompt usado com IA (Claude):**
> "O Docker não consegue descarregar imagens. Erro: dial tcp: lookup
> registry-1.docker.io: no such host. Como resolver no Windows?"

**Resposta da IA:** Sugeriu configurar DNS manualmente no Docker Engine settings.

**O que o grupo fez:** Aplicou a configuração de DNS. Problema resolvido.

---

## ERRO 3 — Atributo `version` obsoleto no docker-compose.yml
**Fase:** Fase 1 — docker-compose.yml

**Erro (aviso):**
```
the attribute `version` is obsolete, it will be ignored
```

**Causa:** Versões recentes do Docker Compose já não utilizam o campo `version`
no ficheiro `docker-compose.yml`.

**Solução:** Remover a linha `version: '3.8'` do início do ficheiro.

**Prompt usado com IA (Claude):**
> "O docker compose mostra um aviso sobre version obsoleto. Devo remover?"

**Resposta da IA:** Confirmou que é apenas um aviso, não um erro, e que a linha
pode ser removida sem consequências.

**O que o grupo fez:** Removeu a linha. Aviso desapareceu.

---

## ERRO 4 — Conflito de portas com o XAMPP
**Fase:** Fase 1 — Configuração das portas

**Problema:** O XAMPP já utilizava a porta 3307 no computador. O Docker iria
tentar usar a mesma porta para o contentor `no-benguela`, causando conflito.

**Causa:** Dois processos não podem usar a mesma porta no mesmo computador.

**Solução:** Alterar as portas do lado do anfitrião no docker-compose.yml:
- no-luanda:   3306 → 3360
- no-benguela: 3307 → 3361
- no-huambo:   3308 → 3362

Atualizar também o script Python para usar as novas portas:
```python
no_luanda   = conectar(3360)
no_benguela = conectar(3361)
```

**Prompt usado com IA (Claude):**
> "O meu XAMPP usa a porta 3307. Isso afeta a configuração do Docker? O que
> tenho de mudar?"

**Resposta da IA:** Identificou o conflito e sugeriu portas alternativas 3360/3361/3362.

**O que o grupo fez:** Aplicou as novas portas. Sem conflitos.

---

## ERRO 5 — Tabelas não encontradas com nomes em maiúsculas
**Fase:** Fase 1 — Verificação dos dados

**Erro:**
```
ERROR 1146 (42S02): Table 'mercadokwanza.VENDA' doesn't exist
ERROR 1146 (42S02): Table 'mercadokwanza.CLIENTE' doesn't exist
```

**Causa:** O MySQL no Linux (dentro do Docker) é sensível a maiúsculas/minúsculas
nos nomes de tabelas. O dataset gerado pela IA criou as tabelas em minúsculas
(`venda`, `cliente`), mas as queries do enunciado usavam maiúsculas (`VENDA`, `CLIENTE`).

**Solução:** Usar sempre minúsculas nas queries:
- `FROM VENDA` → `FROM venda`
- `FROM CLIENTE` → `FROM cliente`
- `FROM PRODUTO` → `FROM produto`
- `FROM ITEM_VENDA` → `FROM item_venda`
- `FROM STOCK` → `FROM stock`
- `FROM LOJA` → `FROM loja`

**Prompt usado com IA (Claude):**
> "O MySQL dentro do Docker diz que a tabela VENDA não existe mas o SHOW TABLES
> mostra 'venda' em minúsculas. Como resolver?"

**Resposta da IA:** Explicou que o Linux é case-sensitive nos nomes de tabelas
e sugeriu usar minúsculas consistentemente.

**O que o grupo fez:** Adaptou todas as queries para minúsculas.

---

## ERRO 6 — Tabela `categoria` separada não prevista no enunciado
**Fase:** Fase 0 / Fase 3 — Dataset e INSERT de produtos

**Erro:**
```
ERROR 1364 (HY000): Field 'categoria_id' doesn't have a default value
```

**Causa:** A IA que gerou o dataset criou uma tabela `categoria` separada
(não prevista no esquema do enunciado) e adicionou uma coluna `categoria_id`
à tabela `produto` como chave estrangeira.

**Solução:** Verificar as categorias existentes com `SELECT * FROM categoria`
e incluir o `categoria_id` nos INSERTs:
```sql
INSERT INTO produto (descricao, categoria_id, preco, activo) VALUES
  ('Sabao Protex 200g', 2, 850, 1),
  ('Arroz Precioso 5kg', 1, 3500, 1),
  ('Pilha AA Energizer x4', 3, 1200, 1);
```

**Prompt usado com IA (Claude):**
> "O INSERT na tabela produto dá erro de categoria_id sem valor por defeito.
> O dataset tem uma tabela categoria separada. Como adaptar o INSERT?"

**Resposta da IA:** Sugeriu consultar a tabela categoria primeiro e incluir
o campo categoria_id no INSERT.

**O que o grupo fez:** Consultou a tabela e adaptou o INSERT com os IDs corretos.

---

## ERRO 7 — Base de dados não selecionada antes de criar views
**Fase:** Fase 2 — Fragmentação

**Erro:**
```
ERROR 1046 (3D000): No database selected
```

**Causa:** O comando `USE mercadokwanza` não foi executado antes de criar as views.
O MySQL não sabe em que base de dados criar os objetos.

**Solução:** Executar sempre `USE mercadokwanza;` após entrar no MySQL,
ou passar a base de dados no comando de ligação:
```powershell
docker exec -it no-luanda mysql -uroot -pkwanza2024 mercadokwanza
```

**Prompt usado com IA:** Não foi necessário — o grupo identificou e resolveu
autonomamente.

---

## ERRO 8 — Autenticação SSL entre Master e Slave
**Fase:** Fase 3 — Replicação

**Erro:**
```
Authentication plugin 'caching_sha2_password' reported error:
Authentication requires secure connection.
Last_IO_Errno: 2061
```

**Causa:** O MySQL 8.0 usa por defeito o plugin `caching_sha2_password` que
exige SSL para autenticação entre servidores. Em ambiente de laboratório local
sem SSL configurado, a ligação é recusada.

**Solução:** Adicionar `GET_MASTER_PUBLIC_KEY=1` ao comando `CHANGE MASTER TO`:
```sql
CHANGE MASTER TO
  MASTER_HOST='no-luanda',
  MASTER_USER='root',
  MASTER_PASSWORD='kwanza2024',
  MASTER_LOG_FILE='mysql-bin.000003',
  MASTER_LOG_POS=2047,
  GET_MASTER_PUBLIC_KEY=1;
```

**Prompt usado com IA (Claude):**
> "O Slave dá erro: Authentication plugin caching_sha2_password requires secure
> connection. Como configurar a replicação MySQL 8.0 sem SSL em ambiente local?"

**Resposta da IA:** Identificou o problema do plugin de autenticação e sugeriu
o parâmetro `GET_MASTER_PUBLIC_KEY=1`.

**O que o grupo fez:** Aplicou o parâmetro. Replicação ficou ativa em ambos os Slaves.

---

## ERRO 9 — Slave configurado antes de ter os dados do Master
**Fase:** Fase 3 — Replicação

**Problema:** Os Slaves foram configurados para replicação depois de os dados
já terem sido carregados no Master. A replicação MySQL só copia operações
que acontecem **após** a configuração — os dados anteriores não são copiados.
Resultado: Slaves ficaram com as tabelas vazias.

**Causa:** Ordem incorreta de operações — o correto seria:
1. Configurar replicação
2. Fazer dump do Master
3. Importar nos Slaves
4. Iniciar replicação a partir da posição correta do dump

**Solução aplicada:** Exportar dados do Master e importar diretamente nos Slaves
usando pipe no CMD (o PowerShell não suporta `<` para redirecionamento):
```cmd
docker exec no-luanda mysqldump -uroot -pkwanza2024 --routines --single-transaction mercadokwanza | docker exec -i no-benguela mysql -uroot -pkwanza2024 mercadokwanza
```

**Prompt usado com IA (Claude):**
> "Os Slaves ficaram sem dados porque a replicação foi configurada depois de
> carregar os dados no Master. Como copiar os dados existentes para os Slaves?"

**Resposta da IA:** Sugeriu usar mysqldump para exportar e reimportar nos Slaves,
e depois reconfigurar a replicação com a posição atualizada do binlog.

---

## ERRO 10 — Redirecionamento `<` não funciona no PowerShell
**Fase:** Fase 3 — Importação de dados nos Slaves

**Erro:**
```
The '<' operator is reserved for future use.
ParserError: RedirectionNotSupported
```

**Causa:** O PowerShell não suporta o operador `<` para redirecionar ficheiros
como input. Este operador só funciona no CMD clássico do Windows.

**Solução:** Usar o CMD em vez do PowerShell para comandos com redirecionamento,
ou usar o pipe `|` que funciona em ambos:
```cmd
docker exec no-luanda mysqldump ... | docker exec -i no-benguela mysql ...
```

**Prompt usado com IA (Claude):**
> "No PowerShell dá erro ao usar < para redirecionar ficheiro para docker exec.
> Como importar um dump SQL para um contentor Docker no Windows?"

**Resposta da IA:** Explicou a limitação do PowerShell e sugeriu usar CMD
ou pipe direto entre comandos docker.

---

## ERRO 11 — Ficheiro dump.sql com encoding UTF-16 (caracteres nulos)
**Fase:** Fase 3 — Importação de dados nos Slaves

**Erro:**
```
ERROR: ASCII '\0' appeared in the statement, but this is not allowed
unless option --binary-mode is enabled.
Query: '��-'
```

**Causa:** O CMD do Windows guardou o ficheiro `dump.sql` com encoding UTF-16
(com BOM e caracteres nulos `\0`), que o MySQL não consegue interpretar.

**Solução:** Evitar criar o ficheiro intermédio. Usar pipe direto para transferir
os dados do Master para os Slaves sem passar pelo sistema de ficheiros do Windows:
```cmd
docker exec no-luanda mysqldump -uroot -pkwanza2024 --routines --single-transaction mercadokwanza | docker exec -i no-benguela mysql -uroot -pkwanza2024 mercadokwanza
```

**Prompt usado com IA (Claude):**
> "O dump.sql importado dá erro de ASCII null character. O ficheiro foi criado
> pelo CMD do Windows. Como resolver?"

**Resposta da IA:** Identificou o problema de encoding UTF-16 do CMD e sugeriu
usar pipe direto sem ficheiro intermédio.

**Solução final aplicada:**
```cmd
docker exec no-luanda mysqldump -uroot -pkwanza2024 --routines --single-transaction mercadokwanza | docker exec -i no-benguela mysql -uroot -pkwanza2024 mercadokwanza

docker exec no-luanda mysqldump -uroot -pkwanza2024 --routines --single-transaction mercadokwanza | docker exec -i no-huambo mysql -uroot -pkwanza2024 mercadokwanza
```

**Estado:** ✅ Resolvido — dados importados com sucesso nos dois Slaves.

---

## RESULTADO FINAL — Fase 3: Replicação

### Configuração dos Slaves (segunda tentativa — bem sucedida)

Após importar os dados via pipe, os Slaves foram reconfigurados com a posição
atualizada do binlog do Master:

```
File: mysql-bin.000003
Position: 2458
```

Comando usado em ambos os Slaves:
```sql
STOP SLAVE;
CHANGE MASTER TO
  MASTER_HOST='no-luanda',
  MASTER_USER='root',
  MASTER_PASSWORD='kwanza2024',
  MASTER_LOG_FILE='mysql-bin.000003',
  MASTER_LOG_POS=2458,
  GET_MASTER_PUBLIC_KEY=1;
START SLAVE;
```
---

*Documento gerado durante a execução do Lab 02 — ISPTEC BDD II 2025/2026*
