# recoloca-ia

Assistente de desenvolvimento de carreira baseado em agentes, projetado para ajudar profissionais tech na jornada de recolocação ou transição profissional.

## 🎯 Sobre o Projeto

O **recoloca-ia** funciona como um sistema de orquestração de agentes conversacionais. A persona principal, **Maestro**, conduz o usuário por um processo estruturado de definição de perfil profissional e, a partir disso, coordena subagentes especializados para auxiliar em:

- **Busca de Vagas:** Identificação de oportunidades alinhadas ao perfil (`Scout`).
- **Curadoria de Cursos:** Sugestão de capacitação técnica (`Curator`).
- **Simulação de Entrevistas:** Treinamento prático com feedback (`Coach`).

## 🏗️ Como Funciona

O sistema opera através de personas e habilidades (skills) que interagem com um estado persistido em arquivos Markdown na pasta `data/`.

- **Maestro (`personas/maestro.md`):** É a interface central. Ele gerencia o quiz inicial de perfil e despacha os demais agentes.
- **Estado do Usuário:** As respostas e o perfil consolidado são salvos em `data/personality-quiz.md` e `data/user-profile.md`, garantindo que a conversa possa ser retomada a qualquer momento.
- **Skills (`skills/`):** Contêm as definições de como os subagentes devem executar suas tarefas específicas, seguindo um protocolo de despacho definido em `skills/dispatch.md`.

## 🚀 Como Iniciar

1. Certifique-se de estar utilizando o ambiente de agentes do Zed.
2. Inicie a interação com a persona **Maestro**.
3. O Maestro guiará você por um quiz de 7 perguntas para entender seu objetivo, nível de experiência e interesses.
4. Após concluir o quiz, você terá acesso ao menu de serviços para buscar vagas, encontrar cursos ou praticar entrevistas.

## 🛠️ Estrutura do Repositório

```text
recoloca-ia/
├── data/              # Estado e dados do usuário (quiz, perfil)
├── personas/          # Definições das personas (Maestro, Scout, Curator, etc.)
├── skills/            # Lógica das habilidades dos agentes
└── README.md          # Este arquivo
```

---
*Este projeto é uma ferramenta de auxílio à carreira baseada em agentes.*
