---
description: Fluxo de trabalho para desenvolvimento de CRUDs utilizando o padrão de componente único (Single Component) da TALL Stack.
---

# Fluxo de Trabalho: CRUD TALL Stack

Todas as operações de um módulo devem ser centralizadas em um **único componente Livewire** para reatividade instantânea.

## 1. Estrutura do Componente (Traits Obrigatórias)

O componente principal orquestra as operações e importa Traits separadas por responsabilidade:

```
App/Livewire/{Sistema}/{Perfil}/
├── {Entidade}.php              ← Componente principal (orquestra, usa Traits)
└── Traits/
    ├── {Entidade}Criar.php     ← Métodos: create(), store(), resetForm()
    ├── {Entidade}Editar.php    ← Métodos: edit(), update()
    └── {Entidade}Excluir.php   ← Métodos: confirmDelete(), destroy()
```

**Regras:**

- Cada Trait tem uma única responsabilidade (SRP).
- O componente principal apenas importa as Traits e define propriedades comuns.

## 2. Estrutura das Views (Partials Obrigatórias)

A view principal inclui partials para cada formulário/modal:

```
resources/views/livewire/{sistema}/{perfil}/{entidade}/
├── index.blade.php              ← View principal (tabela + @includes)
└── partials/
    ├── _form-criar.blade.php    ← Modal/Form de criação
    ├── _form-editar.blade.php   ← Modal/Form de edição
    └── _modal-excluir.blade.php ← Modal de confirmação de exclusão
```

**Regras:**

- Prefixo `_` indica partial (convenção Laravel).
- Partials incluídas via `@include('livewire.{sistema}.{perfil}.{entidade}.partials._form-criar')`.

## 3. Comportamentos Padrão

- **Listagem Dinâmica:** Tabela Tailwind com busca em tempo real usando `wire:model.live.debounce.500ms="search"`.
- **Paginação Personalizável:** Variável `public $perPage` com opções [10, 25, 50, 100] vinculada via `wire:model.live`.
- **Sincronização de Estado:** Uso obrigatório de `@entangle` para que o AlpineJS controle a visibilidade de modais/formulários baseado no estado do Livewire.
