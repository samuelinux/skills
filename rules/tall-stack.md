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
- **JS Externo:** Proibido o uso de JavaScript puro (Vanilla) em arquivos `.js` separados. Toda lógica de interatividade deve usar as diretivas do AlpineJS diretamente no HTML.

# 2. Arquitetura Modular e Portabilidade

## 2.1 Estrutura de Pastas (3 Níveis)

O projeto segue uma hierarquia organizacional rígida: **Sistema ➝ Perfil (Role) ➝ Funcionalidade (Menu)**.

## 2.2 Regras de Portabilidade (Agnóstico ao Servidor)

- **Proibido Hardcoding:** Nunca use drivers específicos (ex: `disk('local')`) ou caminhos absolutos fixos (ex: `/var/www/...`).
- **Abstração de Arquivos:** Use sempre `Storage::` sem especificar o driver no código (deixe o Laravel usar o padrão do `.env`) ou use drivers configuráveis.
- **Helpers de Caminho:** Utilize obrigatoriamente helpers como `base_path()`, `storage_path()`, `public_path()` ou `config()` para referenciar diretórios.
- **Drivers de Serviço:** Serviços como Mail, Cache e Session devem usar o driver definido no `.env`, garantindo que o sistema funcione em ambientes limitados (Hostinger) apenas trocando a configuração.

# 3. Comunicação e Feedback (SweetAlert2)

O feedback ao usuário deve ser disparado pelo Livewire (Backend) e executado pelo Browser (Frontend).

### No Componente (PHP):

$this->dispatch('swal:modal', icon: 'success', title: 'Sucesso!', text: 'Operação realizada.');

### Estilos de UI (Tailwind):

- **Design:** Minimalista e focado em usabilidade.
- **Loading States:** Uso extensivo de `wire:loading` e `wire:target`.
- **Flickering:** Uso obrigatório de `x-cloak`.
