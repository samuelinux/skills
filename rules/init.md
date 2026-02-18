---
description: Regras globais e inicialização do agente
---

# 1. Regras de Comunicação e Operação

- **Idioma:** Sempre fale em **Português** em todas as interações, na criação de arquivos e na documentação, assim também como nos comentários do código. Evite usar inglês em qualquer circunstancia, exceto quando for estritamente necessário.
- **Protocolo de Inicialização:** Antes de qualquer resposta, execução de comando ou criação de arquivos, você deve **obrigatoriamente analisar e memorizar** todos os arquivos contidos na pasta `.agent`.

# 2. Referências de Módulos

- **Regras:** [TALL Stack Rules](/rules/tall-stack.md)
- **Workflows:** [CRUD TALL Stack](/workflows/crud-tall.md)
- **Knowledge:** [Escalabilidade](/knowledge/scalability.md)

# 3. Filosofia do Projeto

- **Foco em Desempenho:** O projeto é focado em desempenho extremo. Todo código, arquitetura e sugestão deve ser otimizado para garantir o menor tempo de resposta e o uso mais eficiente de recursos.
- **Portabilidade e Abstração:** O código deve ser totalmente agnóstico ao servidor (multi-servidor), funcionando perfeitamente tanto em **Hostinger (Shared)** quanto em **VPS**. É proibido o uso de caminhos absolutos fixos ou a definição rígida (hardcoding) de drivers. Utilize sempre as camadas de abstração do Laravel (Helpers e Configurações via `.env`).
- **Segurança e Validação:** A segurança é prioridade absoluta. É **obrigatório** implementar validações robustas em todas as rotas e em todas as interações com o banco de dados (Model e Migration). Nunca confie em dados vindos do cliente sem validação prévia.
- **Arquitetura Limpa (Clean Architecture):** Siga rigorosamente os princípios de arquitetura limpa. A lógica de negócio (Domain) deve ser totalmente isolada da infraestrutura (Infrastructure). O código deve ser organizado em camadas bem definidas, com dependências fluindo em uma única direção (geralmente de fora para dentro).
- **Princípios SOLID:** Aplique os princípios SOLID em todo o código. Evite acoplamento forte, favoreça a composição sobre a herança e garanta que cada classe tenha uma única responsabilidade.
- **Testabilidade:** Escreva código que seja fácil de testar. Evite efeitos colaterais e dependências globais que dificultem a criação de testes unitários e de integração.
- **Manutenibilidade:** O código deve ser fácil de ler, entender e manter. Use nomes de variáveis e funções descritivos, evite complexidade desnecessária e mantenha o código organizado em funções e classes coesas, sempre seguindo as boas práticas de programação, comente o código sempre que necessário para que ele possa ser entendido por outros desenvolvedores.
- **Padronização de Código:** Siga rigorosamente as convenções de nomenclatura e formatação do Laravel. Use snake_case para variáveis e funções, PascalCase para classes e métodos, e mantenha um estilo de código consistente em todo o projeto.
- **Documentação:** Documente todas as decisões importantes de arquitetura e as configurações necessárias. Use comentários claros e concisos para explicar o "porquê" por trás das escolhas técnicas.
