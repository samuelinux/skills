---
description: Regras globais e inicialização do agente
---

# 1. Regras de Comunicação e Operação

- **Idioma:** Sempre fale em **Português** em todas as interações.
- **Protocolo de Inicialização:** Antes de qualquer resposta, execução de comando ou criação de arquivos, você deve **obrigatoriamente analisar e memorizar** todos os arquivos contidos na pasta `.agent`.

# 2. Referências de Módulos

- **Regras:** [TALL Stack Rules](/rules/tall-stack.md)
- **Workflows:** [CRUD TALL Stack](/workflows/crud-tall.md)
- **Knowledge:** [Escalabilidade](/knowledge/scalability.md)

# 3. Filosofia do Projeto

- **Foco em Desempenho:** O projeto é focado em desempenho extremo. Todo código, arquitetura e sugestão deve ser otimizado para garantir o menor tempo de resposta e o uso mais eficiente de recursos.
- **Portabilidade e Abstração:** O código deve ser totalmente agnóstico ao servidor (multi-servidor), funcionando perfeitamente tanto em **Hostinger (Shared)** quanto em **VPS**. É proibido o uso de caminhos absolutos fixos ou a definição rígida (hardcoding) de drivers. Utilize sempre as camadas de abstração do Laravel (Helpers e Configurações via `.env`).
