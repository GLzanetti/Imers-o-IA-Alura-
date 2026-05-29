# Dispatch – Protocolo de Despacho e Handoff

## Tabela de Roteamento
| Código | Agente   |
|--------|----------|
| A      | Scout    |
| B      | Curator  |
| C      | Coach    |
| D      | Maestro (quiz) |

## Envelope de Despacho (usado pelo Maestro)
```
## DESPACHO: [NOME_DO_AGENTE]
### referencia_persona
[Conteúdo completo de personas/<nome_do_agente_minusculo>.md]

### tarefa
[Uma frase descrevendo o que o agente deve fazer]

### perfil_usuario
[Conteúdo de data/user-profile.md]

### contexto
[Contexto específico: ex: qual vaga para entrevistar, quais habilidades buscar cursos]

### saida_esperada
[Formato exato que o agente deve retornar]
```

## Envelope de Resposta (agente despachado)
```
## RESPOSTA: [NOME_DO_AGENTE]
### estado
sucesso | erro

### resumo
[Resumo legível de 2‑3 frases]

### dados
[Lista numerada de pares chave‑valor]

### erros
[Se estado for erro]
```

## Especificações de Handoff por Agente
- **Scout**: recebe `perfil_usuario`, gera URLs, busca vagas, grava `data/scout-results.md` e devolve envelope de resposta.
- **Curator**: (não implementado ainda) receberá perfil e retornará recomendações de cursos.
- **Coach**: (não implementado ainda) conduzirá entrevista simulada em 6 despachos sequenciais.
- **Maestro**: gerencia quiz, gera `data/user-profile.md` e apresenta menu.

## Despacho Sequencial do Coach (6 passos)
1. Pergunta inicial de introdução.
2. Pergunta de experiência técnica.
3. Pergunta de comportamento.
4. Pergunta de resolução de problemas.
5. Pergunta de cultura e valores.
6. Feedback final.

## Regras de Tratamento de Erros
- Se um agente retornar `estado: erro`, o Maestro deve incluir a mensagem em `erros` ao usuário.
- O Maestro pode tentar re‑despachar o agente uma única vez se o erro for recuperável (ex.: falha de rede).
- Erros não recuperáveis devem ser reportados ao usuário com instruções de ação.
