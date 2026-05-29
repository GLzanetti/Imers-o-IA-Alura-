# job-search.md

## Descrição

Skill responsável por buscar vagas de emprego usando o CLI **Firecrawl**, analisar os resultados e comparar as habilidades requeridas com as habilidades atuais do usuário.

## Ferramentas do Zed

- `terminal` – executar comandos `firecrawl search` e `firecrawl scrape`.

## Fluxo de Busca de Vagas

1. **Construir a query**
   - Utilizar os campos `Área de interesse`, `Localização` e `Nível de experiência` do arquivo `data/user-profile.md`.
   - Formatar a query como:
     ```
     vagas [area_de_interesse] [localizacao]
     ```
   - Exemplo: `vagas Backend São Paulo`.

2. **Executar a busca**
   - Comando:
     ```
     firecrawl search "vagas [area] [localizacao]" --json
     ```
   - O comando devolve um JSON contendo, para cada resultado, ao menos os campos `url`, `title`, `description`.

3. **Processar o JSON**
   - Parsear o JSON e manter apenas os primeiros **10** resultados (para limitar tempo).
   - Para cada resultado, executar:
     ```
     firecrawl scrape <url> --format markdown
     ```
   - O markdown retornado deve ser analisado para extrair:
     - **Título da vaga** (geralmente o `title`).
     - **Empresa** (pode ser inferida a partir da URL ou do título).
     - **Localização** (buscar palavras como "Remoto", nomes de cidades ou estados).
     - **Habilidades requeridas** – procurar listas ou frases que contenham palavras‑chave de tecnologias (ex.: Python, React, SQL, Docker, etc.).
   - Caso o `scrape` falhe, usar apenas o `title` e `description` do resultado da busca e registrar a falha.

4. **Comparar habilidades**
   - Ler o arquivo `data/user-profile.md` e obter a lista de **Habilidades atuais**.
   - Normalizar ambas as listas (minúsculas, remover pontuação) e comparar usando correspondência exata de strings.
   - Produzir duas listas:
     - `habilidades_correspondentes` – presentes tanto na vaga quanto nas habilidades do usuário.
     - `habilidades_faltantes` – presentes na vaga mas não no usuário.
   - Calcular `contagem_correspondencia` como `X de Y`, onde `X` é o número de correspondências e `Y` o total de habilidades requeridas encontradas.

5. **Filtrar por nível de experiência**
   - Se a descrição da vaga mencionar explicitamente "Júnior", "Pleno" ou "Sênior", priorizar vagas que coincidam com o nível do usuário.
   - Caso não existam vagas do nível exato nos primeiros resultados, incluir vagas de nível adjacente e anotar a discrepância no campo `nivel_discrepancia`.

6. **Selecionar até 5 vagas**
   - Ordenar os resultados por **contagem_correspondencia** (decrescente) e, em caso de empate, por proximidade do nível de experiência.
   - Retornar no máximo 5 vagas.

## Formato de Resposta Esperado (Envelope de Resposta)

```
## RESPOSTA: SCOUT
### estado
[sucesso | erro]

### resumo
[Resumo legível de 2‑3 frases]

### dados
1. titulo: <título>
   empresa: <empresa>
   localizacao: <localização>
   link: <url>
   habilidades_correspondentes: <lista separada por vírgulas>
   habilidades_faltantes: <lista separada por vígulas>
   contagem_correspondencia: <X de Y>
   nivel_discrepancia: <texto opcional>
2. ...

### erros
[Somente se estado for erro]
```

## Tratamento de Erros

- **Falha no comando `firecrawl search`** – registrar o erro e retornar `estado: erro` com mensagem no campo `erros`.
- **Falha no `firecrawl scrape` de uma URL específica** – usar título/descrição do resultado da busca, preencher os campos faltantes com "N/A" e incluir a observação "scrape falhou".
- **JSON inválido** – reportar erro e abortar a skill.
- **Arquivo `data/user-profile.md` ausente** – retornar erro indicando que o perfil do usuário não foi encontrado.

## Observações

- Todos os caminhos de arquivo devem ser referenciados com o prefixo `data/` conforme a política do projeto.
- Não gerar tabelas markdown; usar listas numeradas com pares chave‑valor.
- O agente que consome esta skill (Scout) deve incluir o envelope de despacho conforme definido em `skills/dispatch.md`.
