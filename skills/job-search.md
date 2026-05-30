# Job Search – Skill para o Scout

## Visão Geral
Esta skill define o fluxo completo que o agente **Scout** deve executar para buscar vagas de emprego, extrair requisitos e comparar com as habilidades do usuário.

## Ferramentas Utilizadas
- **terminal** – para executar os comandos CLI do Firecrawl (`firecrawl search` e `firecrawl scrape`).
- **read_file** – ler `data/user-profile.md` e obter as habilidades, nível de experiência, área de interesse e localização do usuário.
- **write_file** – gravar o resultado final em `data/job-search-results.md`.

## Passos do Fluxo
1. **Carregar perfil do usuário**
   - Ler `data/user-profile.md`.
   - Extrair os campos:
     - `area_de_interesse`
     - `localizacao`
     - `nivel_de_experiencia`
     - `habilidades` (lista separada por vírgulas).
   - Se o arquivo não existir ou algum campo estiver ausente, retornar `estado: erro` com mensagem apropriada.

2. **Construir query de busca**
   - Formatar a query como: `vagas {area_de_interesse} {localizacao}`.
   - Exemplo: `vagas backend São Paulo`.

3. **Executar busca inicial**
   - Comando: `firecrawl search "{query}" --json`.
   - Capturar a saída JSON. Cada item contém ao menos `url`, `title`, `description`.
   - Se o comando falhar, retornar erro.

4. **Processar resultados**
   - Para cada resultado (até 10 inicialmente):
     - Executar `firecrawl scrape <url> --format markdown` para obter a descrição completa da vaga.
     - Se o scrape falhar, usar apenas `title` e `description` do resultado da busca.
     - Extrair **habilidades requeridas** procurando por palavras‑chave típicas (ex.: "requisitos", "necessário", "conhecimentos", "skills").
       - Simplificação: considerar todas as palavras separadas por vírgulas ou listadas em linhas que contenham a palavra "skill" ou "conhecimento".
       - Normalizar para minúsculas e remover pontuação.
     - Determinar **empresa** a partir da URL (domínio) ou do título.
     - Determinar **localização** da vaga (se presente no texto; caso contrário usar a localização da query).
     - Detectar **nível de experiência** mencionado (palavras "júnior", "pleno", "sênior").

5. **Correspondência de habilidades**
   - Comparar a lista de habilidades requeridas com a lista do usuário (case‑insensitive).
   - `habilidades_correspondentes` = interseção.
   - `habilidades_faltantes` = requisitos que não estão na lista do usuário.
   - `contagem_correspondencia` = "X de Y habilidades correspondem" onde X = |correspondentes| e Y = |requeridas|.

6. **Filtragem por nível**
   - Priorizar vagas cujo nível mencionado coincide com `nivel_de_experiencia` do usuário.
   - Se nas primeiras 5 vagas não houver coincidência, expandir a busca incluindo vagas de nível adjacente (ex.: para Júnior, incluir Pleno) e marcar a discrepância no campo opcional `nivel_discrepancia`.

7. **Ordenação**
   - Ordenar primeiro por **contagem de correspondência** (descendente) e depois por **relevância de nível**.
   - Selecionar até **5 vagas**.

8. **Construir Envelope de Resposta**
   - Formatar conforme definido em `skills/dispatch.md` (ver seção *Envelope de Resposta*).
   - Campo `estado` = `sucesso`.
   - Campo `resumo` = texto curto indicando quantas vagas foram encontradas e o grau de correspondência geral.
   - Campo `dados` = lista numerada das vagas com os pares chave‑valor:
     ```
     1. titulo: <título>
        empresa: <empresa>
        localizacao: <cidade ou Remoto>
        link: <URL>
        habilidades_correspondentes: [h1, h2]
        habilidades_faltantes: [h3, h4]
        contagem_correspondencia: <X de Y habilidades correspondem>
        nivel_discrepancia: <texto opcional>
     2. ...
     ```
   - Campo `erros` vazio.

9. **Persistir resultados**
   - Escrever o mesmo conteúdo (sem o cabeçalho `## RESPOSTA`) em `data/job-search-results.md` seguindo o modelo descrito em `plano-aula-2.md`.
   - Incluir data/hora da busca e os parâmetros usados.

## Tratamento de Erros
- **Falha no `firecrawl search`**: capturar a mensagem de erro, definir `estado: erro`, preencher `resumo` com "Busca de vagas falhou" e colocar a mensagem em `erros`.
- **Falha no `firecrawl scrape`** de uma URL específica: usar título/descrição da busca, definir `habilidades_faltantes` como "informação não disponível" e anotar a falha no campo `erros` (mas não abortar o fluxo).
- **JSON inválido**: abortar a skill e retornar erro.
- **Arquivo de perfil inexistente**: retornar erro indicando que o usuário ainda não completou o quiz.
- **Nenhum resultado encontrado**: retornar `estado: sucesso` com `resumo` indicando que nenhuma vaga foi encontrada e deixar `dados` vazio.

## Observações
- Todos os caminhos de arquivo devem ser prefixados com `data/`.
- Não usar tabelas markdown; usar listas numeradas como especificado.
- Não gerar dados fictícios; se alguma informação não puder ser obtida, usar "não disponível".
- O agente **Scout** deve chamar esta skill como seu único ponto de entrada; toda a lógica descrita acima deve ser implementada dentro da skill.
