# Plano Aula 3 – Curator (Busca de Cursos)

## Visão Geral

Este plano descreve a criação do terceiro agente **Curator**, responsável por buscar cursos na Alura (alura.com.br) que preencham as lacunas de habilidades identificadas para o usuário. O Curator utiliza o **Firecrawl** da mesma forma que o agente Scout (busca de vagas) para navegar, extrair e analisar conteúdo do site da Alura.

## Objetivo

- Navegar no site da Alura usando Firecrawl.
- Identificar cursos que complementem as habilidades faltantes do usuário, de acordo com o *quiz* e as vagas encontradas.
- Retornar um envelope de resposta estruturado que o Maestro pode apresentar ao usuário e armazenar em `data/curator-results.md`.

## Pré‑requisitos

- Firecrawl instalado e configurado (`FIRECRAWL_API_KEY` definida).
- O usuário já completou o quiz e tem o perfil em `data/user-profile.md`.
- As vagas já foram buscadas pelo agente **Scout** e estão disponíveis em `data/job-search-results.md` (para cruzar habilidades requeridas nas vagas).

## Diretrizes MoE (mesmas do plano anterior)

- Saídas estruturadas como **listas numeradas** com pares *chave‑valor* (nenhuma tabela markdown).
- Todos os caminhos de arquivo começam com `data/`.
- Relatar falhas no campo `erros` e interromper a execução.
- Não gerar código que implemente a persona; o agente **Curator** será personificado nas respostas.

## Arquitetura Atualizada

```
┌─────────────────────────────────────────────────┐
│                 Usuário                         │
└────────────────────┬────────────────────────────┘
                     ▼
┌─────────────────────────────────────────────────┐
│                 MAESTRO (Orquestrador)          │
│  - Coordena agentes e consolida resultados       │
└──┬───────────────┬───────────────┬───────────────┘
   │               │               │
   ▼               ▼               ▼
┌───────┐   ┌───────────┐   ┌───────────────┐
│SCOUT │   │ CURATOR   │   │ COACH         │
│(Vagas)│   │(Cursos)   │   │(Entrevista)  │
└───────┘   └───────────┘   └───────────────┘
```

## Fluxo de Trabalho do Curator

1. **Leitura do perfil do usuário** – `data/user-profile.md` para obter:
   - Habilidades atuais
   - Habilidades faltantes (calculadas a partir das vagas em `data/job-search-results.md`).
2. **Construção da query** – gerar termos de busca que combinem as habilidades faltantes e a área de interesse.
3. **Busca no site da Alura** – usar `firecrawl search "site:alura.com.br <termos>" --json`.
4. **Scrape dos resultados** – para cada URL retornada, executar `firecrawl scrape <url> --format markdown` e extrair:
   - Título do curso
   - Descrição resumida
   - Habilidades abordadas (identificadas a partir da descrição)
   - Link direto para o curso
5. **Correspondência de habilidades** – comparar as habilidades abordadas pelo curso com as habilidades faltantes do usuário.
6. **Ordenação** – priorizar cursos que cubram o maior número de lacunas.
7. **Construção do Envelope de Resposta** – formato definido abaixo.
8. **Persistência** – salvar a resposta em `data/curator-results.md`.
9. **Retorno ao Maestro** – o Maestro apresenta ao usuário a lista de cursos e volta ao menu.

## Envelope de Despacho (Construído pelo Maestro)

```
## DESPACHO: CURATOR
### referencia_persona
[conteúdo completo de personas/curator.md]

### tarefa
Buscar cursos na Alura que preencham as lacunas de habilidades do usuário.

### perfil_usuario
[conteúdo de data/user-profile.md]

### contexto
Habilidades faltantes: <lista>
Área de interesse: <valor>
Nível de experiência: <valor>
Localização: <valor>

### saida_esperada
Envelope de resposta com estado, resumo, dados (lista de cursos) e erros se houver.
```

## Envelope de Resposta (Curator)

```
## RESPOSTA: CURATOR
### estado
[sucesso | erro]

### resumo
[texto curto de 2‑3 frases]

### dados
1. titulo: <Título do Curso>
   link: <URL>
   habilidades_cobertas: <lista>
   habilidades_faltantes: <lista>
   descricao_resumida: <texto>
2. ... (até 5 cursos)

### erros
[apenas se estado for erro]
```

## Estrutura de Arquivos

```
recoloca-ia/
├── personas/
│   └── curator.md          # Persona do agente Curator
├── skills/
│   └── course-search.md    # Skill que descreve o fluxo de busca de cursos (similar ao job-search.md)
└── data/
    ├── curator-results.md  # Resultado mais recente da busca de cursos
    └── ... (outros arquivos existentes)
```

## Tasks a Serem Implementados

1. **Criar `personas/curator.md`** – descrição da persona, ferramentas disponíveis (`terminal` + Firecrawl) e referência ao skill `skills/course-search.md`.
2. **Criar `skills/course-search.md`** – instruções detalhadas para:
   - montar a query de busca;
   - usar `firecrawl search` e `firecrawl scrape`;
   - extrair habilidades dos cursos;
   - comparar com as lacunas do usuário;
   - formatar a resposta.
3. **Atualizar o Maestro** (não parte deste plano, mas o Maestro precisará montar o envelope de despacho e chamar `spawn_agent` com a label "Curator").
4. **Persistir resultados** em `data/curator-results.md`.
5. **Testar** a execução completa: selecionar a opção **B** no menu, o Maestro despacha o Curator, o Curator devolve até 5 cursos relevantes e o Maestro exibe ao usuário.

## Regras de Erro

- Se `firecrawl search` falhar, preencher `estado: erro` e detalhar o motivo em `erros`.
- Se o `scrape` de um curso específico falhar, usar apenas o título e a URL do resultado da busca, marcando a extração como parcial.
- Se nenhuma habilidade faltante for encontrada (ou seja, o usuário já possui todas as habilidades), retornar `estado: sucesso` com um resumo informando que não há cursos necessários.

## Notas Técnicas

- **Correspondência de habilidades**: usar comparação case‑insensitive e considerar sinônimos simples (ex.: "Python" ≈ "python").
- **Limite de resultados**: máximo de 5 cursos para evitar sobrecarga de informação.
- **Persistência**: o arquivo `data/curator-results.md` seguirá o mesmo padrão de cabeçalho de data e parâmetros de busca usado pelo Scout.

---

**Próximos passos** – após concluir este plano, avançaremos para a implementação do agente **Coach** (simulação de entrevista) conforme descrito no plano‑aula‑4.
