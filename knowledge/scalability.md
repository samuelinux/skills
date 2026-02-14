---
description: Guia de boas práticas e estratégias para garantir a performance e escalabilidade de aplicações TALL Stack.
---

# Base de Conhecimento: Escalabilidade e Performance TALL Stack

O projeto tem como pilar central o **Desempenho Extremo**. O Livewire 3 é **Stateful por padrão**, o que exige atenção redobrada à hidratação de dados e ao consumo de memória.

## 1. Práticas Recomendadas para Performance

| Prática                      | Configuração                                          |
| ---------------------------- | ----------------------------------------------------- |
| **Session Driver**           | Usar `database` ou `redis` (nunca `file` em produção) |
| **Eager Loading**            | Sempre usar `with()` em queries para evitar N+1       |
| **Paginação**                | Sempre paginar listagens volumosas                    |
| **Cache de Dados Estáticos** | Usar `Cache::remember()` para selects e configurações |
| **Hidratação Mínima**        | Manter apenas IDs e strings em variáveis `public`     |

## 2. Hidratação de Estado (Conceito)

A **Hidratação** é o processo onde o Livewire reconstrói o estado do componente a cada requisição.

- **Impacto:** Propriedades `public` pesadas (como Models completos) aumentam drasticamente o payload JSON, gerando latência perceptível em servidores compartilhados (como Hostinger).
- **Solução Antigravity:** Utilize `Computed Properties` (`#[Computed]`). Elas não são enviadas de volta para o cliente, economizando banda e processamento.

## 3. Técnicas de Carregamento Otimizado

- **Lazy Loading de Componentes:** Usar `wire:init` para carregar dados pesados após o render inicial.
- **Defer Loading:** Usar `wire:model.blur` ao invés de `.live` quando a atualização em tempo real não for estritamente necessária.
- **Componentes Pesados:** Considerar `Livewire::lazy()` para componentes que processam muitos dados.

## 4. Compatibilidade: Hostinger (Shared) vs VPS

Para garantir que a mesma base de código rode em ambos os ambientes:

| Recurso      | Hostinger (Shared)             | VPS (Full Control)    | Estratégia Antigravity                                                        |
| :----------- | :----------------------------- | :-------------------- | :---------------------------------------------------------------------------- |
| **Storage**  | Disco Local (Limitado)         | Local / S3 / Block    | Usar `Storage::put()`, `Storage::get()`, etc. sem especificar disco no código |
| **Caminhos** | `/home/u123/...`               | `/var/www/html/...`   | Usar `storage_path()` e `base_path()` (nunca caminhos absolutos)              |
| **Drivers**  | `file` / `database`            | `redis` / `memcached` | Definir via `.env` sem alterar o código PHP                                   |
| **Symlinks** | Às vezes restrito              | Totalmente livre      | Usar entrega via rotas (`web.php`) para evitar dependência de `storage:link`  |
| **Queues**   | Cron Job (`--stop-when-empty`) | Supervisor + Redis    | Código sempre usa Jobs; o Worker muda por ambiente                            |

> **Nota sobre `Storage`:** A facade `Storage::` sem argumento em `disk()` já utiliza automaticamente o driver definido em `FILESYSTEM_DISK` no `.env`. Chamar `Storage::disk(config('filesystems.default'))` é redundante. Basta usar `Storage::put()`, `Storage::url()`, etc.

## 5. Segurança de Arquivos e Rotas de Entrega

Para blindar a aplicação e garantir a portabilidade:

- **Pasta `/public` é Restrita:** Nunca salve documentos ou fotos de usuários em `/public` ou `/public_html`. Use estas pastas **apenas para assets estáticos** (CSS, JS, logos do sistema).
- **`public_path()` é para Assets Estáticos:** O helper `public_path()` deve ser usado **exclusivamente** para referenciar CSS, JS e imagens do sistema. **Nunca** use-o para uploads ou arquivos de usuários.
- **Armazenamento Seguro:** Salve arquivos em `storage/app/private/` (ou pastas similares fora da raiz web).
- **Entrega via Rotas:** Crie rotas no `web.php` que entreguem o arquivo via PHP (ex: `return Storage::download()`).
  - **Segurança:** Permite anexar Middlewares de verificação de permissão na rota.
  - **Portabilidade:** Funciona em qualquer servidor sem depender de links simbólicos (`storage:link`).

### 5.1 Por Que `storage:link` Falha em Shared Hosting

O comando `php artisan storage:link` cria um symlink de `public/storage` para `storage/app/public`. Em hospedagens compartilhadas como Hostinger, isso pode falhar por dois motivos:

1. **Pasta pública diferente:** A raiz web é `public_html`, não `public`. O Laravel espera `public/`, causando links quebrados.
2. **Permissão SSH restrita:** O servidor pode bloquear a criação de symlinks por questões de segurança.

**Solução:** Sempre use **entrega via rotas** (Seção 5 acima) como padrão. Isso elimina a dependência de symlinks e funciona em 100% dos ambientes.

## 6. Armadilhas de Escalabilidade (Pitfalls)

- **Helper `public_path()` para uploads:** Proibido. Em multi-servidores, ele aponta para o disco local de uma máquina específica. Use `Storage::url()` ou caminhos gerenciados pelo driver.
- **Sessões em Arquivos:** Nunca use o driver `file` em produção. Use `database` (Hostinger) ou `redis` (VPS) para que o estado do usuário seja compartilhado entre servidores.
- **Cache Local em Arquivo:** Evite o driver de cache `file` se o projeto for escalar para múltiplas VPS. Use `database` ou `redis`.
- **Funções de Sistema (`exec`, `shell_exec`, `system`):** Proibido. Elas são bloqueadas por padrão em hospedagens compartilhadas. Se precisar processar arquivos (ex: redimensionar imagem), use bibliotecas PHP puras (como Intervention Image) ou mova para uma Job que rode em uma VPS com acesso ao binário.
- **Sticky Sessions em Load Balancer:** Se usar múltiplos servidores atrás de um Load Balancer sem sessão compartilhada (Redis/database), o usuário perderá a sessão ao ser redirecionado para outro servidor.

## 7. Rate Limiting e Throttling

Escalabilidade não é só infraestrutura — é também **proteção contra abuso**.

### 7.1 Rate Limiting de Rotas

O Laravel possui rate limiting nativo via middleware `throttle`. Configure no `AppServiceProvider`:

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

RateLimiter::for('global', function ($request) {
    return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
});
```

### 7.2 Boas Práticas

| Cenário                        | Limite Sugerido            |
| :----------------------------- | :------------------------- |
| **Login / Autenticação**       | 5 tentativas / minuto      |
| **API Pública**                | 60 requests / minuto       |
| **Uploads de Arquivo**         | 10 uploads / minuto        |
| **Livewire (requests padrão)** | Confiar no throttle global |

### 7.3 Proteção Adicional

- **CSRF Token:** Já nativo no Laravel, protege contra ataques de formulários externos.
- **Cloudflare (Recomendado):** Usar como proxy reverso para proteção contra DDoS, bot detection e firewall de aplicação (WAF).
8465465
## 8. Atualizações em Tempo Real (Real-time)

### 8.1 Progressão Recomendada

Adote a complexidade de real-time de forma **gradual**, conforme a necessidade:

| Fase  | Tecnologia                    | Quando Usar                                         | Custo  |
| :---- | :---------------------------- | :-------------------------------------------------- | :----- |
| **1** | `wire:poll.15s`               | Dashboards simples, listas que precisam atualizar   | Zero   |
| **2** | Livewire Events (`$dispatch`) | Comunicação entre componentes na mesma página       | Zero   |
| **3** | Laravel Echo + Pusher/Soketi  | Push real do servidor para o browser (notificações) | Pago\* |

> \*Soketi é uma alternativa open-source e self-hosted ao Pusher.

### 8.2 Cuidados com `wire:poll`

- **Nunca use intervalos < 5 segundos.** Isso gera carga excessiva no servidor.
- **Prefira `wire:poll.visible`** para que o polling pause quando a aba não estiver ativa.
- **Em shared hosting:** Use intervalos de 15s ou mais para não estourar limites de request.

## 9. CDN e Cache de Assets

Como o projeto usa **zero compilação** (CSS/JS servidos direto de `/public`), é crucial otimizar a entrega desses arquivos.

### 9.1 Cloudflare como CDN (Recomendado)

- **Custo:** Gratuito no plano básico.
- **Benefícios:** Cache automático de assets estáticos, compressão Brotli/gzip, proteção DDoS.
- **Configuração:** Apontar o DNS do domínio para o Cloudflare e ativar o proxy (nuvem laranja).

### 9.2 Cache Headers no Servidor

Garanta que o servidor (Apache/Nginx) envie headers de cache corretos para assets estáticos:

**Apache (.htaccess):**

```apache
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType text/css "access plus 1 month"
    ExpiresByType application/javascript "access plus 1 month"
    ExpiresByType image/png "access plus 1 month"
    ExpiresByType image/jpeg "access plus 1 month"
    ExpiresByType image/svg+xml "access plus 1 month"
</IfModule>
```

### 9.3 Versionamento de Assets

Para invalidar cache quando um asset mudar, use query strings com versão:

```html
<link
  rel="stylesheet"
  href="/css/app.css?v={{ filemtime(public_path('css/app.css')) }}"
/>
<script src="/js/app.js?v={{ filemtime(public_path('js/app.js')) }}"></script>
```

> **Nota:** Este é um uso correto de `public_path()` — referenciando assets estáticos do sistema, não uploads de usuário.

## 10. Transição para Stateless

### 10.1 Quando Migrar

Migrar para arquitetura stateless quando **pelo menos dois** dos critérios abaixo forem verdadeiros:

- Memória do servidor estiver constantemente acima de 80%.
- Houver necessidade de Load Balancer com múltiplos servidores.
- Requests ficarem lentos por excesso de estado hidratado.
- O número de usuários simultâneos ultrapassar a capacidade de um único servidor.

### 10.2 Como Monitorar

| Ferramenta              | O que Monitora                      | Ambiente       |
| :---------------------- | :---------------------------------- | :------------- |
| `php artisan about`     | Versões, drivers, cache, session    | Todos          |
| Laravel Telescope       | Queries, requests, jobs, exceptions | Dev / Staging  |
| Logs do servidor (htop) | CPU, memória, processos             | VPS            |
| Painel Hostinger        | Uso de recursos, limites            | Shared Hosting |

### 10.3 Passos de Migração (Gradual)

1. **Fase 1 — Sessão Compartilhada:** Mover `SESSION_DRIVER` de `file` para `database`.
2. **Fase 2 — Cache Centralizado:** Mover `CACHE_STORE` de `file` para `database` ou `redis`.
3. **Fase 3 — Redis:** Quando tiver VPS, migre sessão e cache para Redis.
4. **Fase 4 — Load Balancer:** Adicione um segundo servidor e configure Nginx como proxy reverso.
5. **Fase 5 — Broadcast:** Implemente Laravel Echo para eliminar `wire:poll` e reduzir requests.

### 10.4 Soluções por Camada

| Camada        | Solução                                         |
| :------------ | :---------------------------------------------- |
| **Sessão**    | Redis (`SESSION_DRIVER=redis`)                  |
| **Cache**     | Redis (`CACHE_STORE=redis`)                     |
| **Real-time** | Laravel Echo + Soketi / Pusher                  |
| **Jobs**      | Redis + Supervisor (Queue Worker)               |
| **Arquivos**  | S3 / DigitalOcean Spaces (`FILESYSTEM_DISK=s3`) |
