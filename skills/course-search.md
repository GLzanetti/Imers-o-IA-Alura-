# Course Search Skill (Curator)

## Objetivo

Descrever o fluxo completo que o agente **Curator** deve seguir para buscar cursos na Alura que preencham as lacunas de habilidades do usuário.

## Passos Detalhados

1. **Leitura do perfil do usuário**
   - Usar `read_file` para abrir `data/user-profile.md`.
   - Extrair as seções:
     - `Habilidades atuais`
     - `Funções alvo` (opcional)
   - Se o arquivo não existir ou estiver incompleto, retornar erro.

2. **Identificar habilidades faltantes**
   - Ler `data/job-search-results.md` (gerado pelo agente Scout).
   - Para cada vaga, coletar a lista de `habilidades_faltantes`.
   - Unir todas as habilidades faltantes em um conjunto (case‑insensitive) removendo duplicatas.
   - Se o conjunto ficar vazio, gerar resposta de sucesso indicando que não há cursos necessários.

3. **Construir a query de busca**
   - Basear a query na **área de interesse** do usuário (campo `Área de interesse` em `user-profile.md`).
   - Incluir as habilidades faltantes como termos de busca.
   - Exemplo de query: `"site:alura.com.br" "Python" "Data Science" "Machine Learning"` (todos os termos separados por espaço).
   - Limitar a query a no máximo 8 termos para evitar sobrecarga.

4. **Executar busca no site da Alura**
   - Usar o comando:
     ```sh
     firecrawl search "site:alura.com.br <query>" --json
     ```
   - Capturar a saída JSON que contém, para cada resultado, pelo menos:
     - `url`
     - `title`
     - `description` (snippet)
   - Se o comando falhar, retornar envelope de erro.

5. **Selecionar até 5 resultados**
   - Ordenar os resultados pela relevância retornada pelo Firecrawl (já incluída no JSON).
   - Manter apenas os primeiros 5.

6. **Scrape detalhado de cada URL**
   - Para cada URL selecionada, executar:
     ```sh
     firecrawl scrape <url> --format markdown
     ```
   - Se o scrape falhar, registrar a falha e usar apenas o `title` e `url` obtidos na busca.
   - Do markdown obtido, extrair:
     - **Título do curso** (geralmente o primeiro cabeçalho `#`)
     - **Descrição resumida** – primeiro parágrafo após o título.
     - **Habilidades abordadas** – procurar por palavras‑chave que correspondam a habilidades (mesmo conjunto usado nas faltantes). Pode ser feita uma busca simples por termos no texto.
   - Normalizar as habilidades extraídas (lowercase, trim).

7. **Correspondência de habilidades**
   - Para cada curso, comparar a lista de habilidades extraídas com o conjunto de habilidades faltantes.
   - Produzir duas listas:
     - `habilidades_cobertas` – interseção.
     - `habilidades_faltantes` – diferença (faltantes que ainda não são cobertas por este curso).
   - Ordenar os cursos por tamanho da lista `habilidades_cobertas` (decrescente).

8. **Construir o Envelope de Resposta**
   - Formato (conforme `plano-aula-3.md`):
     ```markdown
     ## RESPOSTA: CURATOR
     ### estado
     sucesso

     ### resumo
     Encontrados X cursos relevantes na Alura que cobrem Y das habilidades faltantes.

     ### dados
     1. titulo: <Título>
        link: <URL>
        habilidades_cobertas: <lista>
        habilidades_faltantes: <lista>
        descricao_resumida: <texto>
     2. ... (até 5)

     ### erros
     ```
   - Se houver falhas parciais (algum scrape falhou), incluir a informação na descrição do curso e listar a falha em `erros`.
   - Se nenhum curso for encontrado, ainda retornar `estado: sucesso` com resumo indicando que não há cursos relevantes.

## Regras de Erro

- Qualquer falha de comando (`firecrawl search` ou `scrape`) deve ser capturada e incluída no campo `erros`.
- Não interrompa o fluxo por falha em um único curso; continue processando os demais.
- Se o arquivo de perfil ou de vagas não existir, retornar erro imediatamente.

## Persistência

- O agente Curator deve gravar o envelope completo em `data/curator-results.md` usando `write_file`.
- O conteúdo deve iniciar com um cabeçalho de data e parâmetros de busca, por exemplo:
  ```
  Data da Busca: 2026-05-30 14:22
  Query utilizada: <query>
  ```
  seguido pelo envelope de resposta.

## Notas Técnicas

- **Correspondência de habilidades**: usar comparação case‑insensitive e remover acentos.
- **Limite de 5 cursos**: garantir que nunca mais que 5 itens sejam incluídos em `dados`.
- **Performance**: o número de chamadas `scrape` deve ser limitado ao número de resultados selecionados (máx 5).
- **Formato JSON**: ao chamar `firecrawl search`, garantir que a flag `--json` seja usada para facilitar o parsing.
