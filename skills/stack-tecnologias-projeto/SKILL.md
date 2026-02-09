---
name: tall-stack-modular-master-rules
description: Protocolo mestre para desenvolvimento TALL Stack (Laravel 11, Livewire 3, AlpineJS, Tailwind). Arquitetura modular de 3 níveis, sem controllers, sem Vite, e CRUD single-component.
---

# 1. Core Stack & Restrições de Build

- **Framework:** Laravel 11+ e Livewire 3 (Sem uso de Controllers tradicionais).
- **Front-end:** Alpine.js (nativo do Livewire 3) e Tailwind CSS.
- **Banco de Dados:** MySQL 8.0+ com as seguintes práticas:
    - **Soft Deletes:** Usar `deleted_at` em tabelas principais.
    - **Timestamps:** Sempre incluir `created_at` e `updated_at`.
    - **Índices:** Criar índices em colunas de busca frequente e foreign keys.
- **Zero Compilação:** **NÃO utilizar VITE ou NPM.** CSS e JS devem ser vinculados via links `<link>` e `<script>` no layout principal apontando para a pasta `public/`.
- **SweetAlert2:** Proibido uso de CDN ou pacotes de config PHP. Utilizar o arquivo localizado em `public/js/sweetalert2.all.min.js`.
- **JS Externo:** Proibido o uso de JavaScript puro (Vanilla) em arquivos `.js` separados. Toda lógica de interatividade deve usar as diretivas do AlpineJS(nativo do Livewire 3) diretamente no HTML.

# 2. Arquitetura Modular (Espelhamento de 3 Níveis)

O projeto segue uma hierarquia organizacional rígida: **Sistema ➝ Perfil (Role) ➝ Funcionalidade (Menu)**.

- **Regra de Nome:** O nome do arquivo final NÃO deve repetir os nomes das pastas anteriores.
- **Backend (Componentes):** `App/Livewire/{Sistema}/{Perfil}/{Funcionalidade}.php`
- **Frontend (Views):** `resources/views/livewire/{sistema}/{perfil}/{funcionalidade}.blade.php` (Pastas em caixa baixa).
- **Rotas:** Utilizar grupos de prefixos e nomes que reflitam a estrutura de pastas acima.

# 3. Padrão de Desenvolvimento CRUD (Single Component)

Todas as operações de um módulo devem ser centralizadas em um **único componente Livewire** para reatividade instantânea.

## 3.1 Estrutura do Componente (Traits Obrigatórias)

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
- Facilita testes unitários por operação.

## 3.2 Estrutura das Views (Partials Obrigatórias)

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
- Mantém views pequenas e legíveis (DRY).

## 3.3 Comportamentos Padrão

- **Listagem Dinâmica:** Tabela Tailwind com busca em tempo real usando `wire:model.live.debounce.500ms="search"`.
- **Paginação Personalizável:** Variável `public $perPage` com opções [10, 25, 50, 100] vinculada via `wire:model.live`.
- **Estado do Formulário:** Uso de uma única variável de estado (ex: `public $isModalOpen = false`) para alternar entre lista e formulário (Create/Edit).
- **Sincronização de Estado:** Uso obrigatório de `@entangle` para que o AlpineJS controle a visibilidade de modais/formulários baseado no estado do Livewire.

# 4. Comunicação e Feedback (SweetAlert2)

O feedback ao usuário deve ser disparado pelo Livewire (Backend) e executado pelo Browser (Frontend).

### No Componente (PHP):

$this->dispatch('swal:modal', icon: 'success', title: 'Sucesso!', text: 'Operação realizada.');

### No Layout (JS dentro do Blade):

window.addEventListener('swal:modal', event => {
Swal.fire({
icon: event.detail.icon,
title: event.detail.title,
text: event.detail.text,
confirmButtonColor: '#3085d6',
});
});

para criação, edição e erro podemos usar algo simples como:

session()->flash('success', 'Tipo de serviço <strong>'.e($this->name).'</strong> atualizado com sucesso!');
session()->flash('error', 'Não foi possível realizar a operação. Tente novamente.');

<main class="flex-grow py-8">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                @if (session()->has('success'))
                    <div class="mb-4 bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded relative">
                        {{ session('success') }}
                    </div>
                @endif

                @if (session()->has('error'))
                    <div class="mb-4 bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded relative">
                        {{ session('error') }}
                    </div>
                @endif

                {{ $slot }}
            </div>
        </main>

# 5. Padrão de Estilo e UI (Tailwind)

- **Design:** Minimalista, focado em usabilidade e direto ao ponto.
- **Loading States:** Uso extensivo de `wire:loading` e `wire:target` para dar feedback visual em botões e processos de busca/filtro.
- **Flickering:** Uso obrigatório de `x-cloak` em todos os elementos controlados pelo AlpineJS para evitar que apareçam antes do carregamento do script.
- **Estilização:** Apenas classes nativas do Tailwind CSS; evitar a criação de arquivos CSS customizados.

# 6. Preparação para Escalabilidade

O Livewire 3 é **Stateful por padrão** (mantém estado no servidor). Para projetos em fase inicial, isso é adequado. As práticas abaixo preparam o projeto para crescimento sem reescrever código.

## 6.1 Práticas Obrigatórias (Aplicar Agora)

| Prática                      | Configuração                                               |
| ---------------------------- | ---------------------------------------------------------- |
| **Session Driver**           | Usar `database` ou `redis` (nunca `file` em produção)      |
| **Eager Loading**            | Sempre usar `with()` em queries para evitar N+1            |
| **Paginação**                | Sempre paginar listagens (já definido na seção 3.3)        |
| **Cache de Dados Estáticos** | Usar `Cache::remember()` para selects, configurações, etc. |

## 6.2 Práticas Recomendadas (Aplicar Conforme Necessário)

- **Lazy Loading de Componentes:** Usar `wire:init` para carregar dados pesados após o render inicial.
- **Defer Loading:** Usar `wire:model.blur` ao invés de `.live` quando atualização em tempo real não for necessária.
- **Componentes Pesados:** Considerar `Livewire::lazy()` para componentes com muitos dados.

## 6.3 Quando Considerar Stateless

Migrar para arquitetura stateless quando:

- Memória do servidor constantemente acima de 80%
- Necessidade de load balancer / múltiplos servidores
- Requests lentos por excesso de estado hidratado

**Soluções para esse estágio:**

- Redis para cache de sessão e estado
- Laravel Echo + Pusher para eventos push (substituir `wire:poll`)
- Filas para processamento pesado
