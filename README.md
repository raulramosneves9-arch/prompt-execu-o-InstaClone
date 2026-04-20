# 🚀 PROMPT DE EXECUÇÃO — InstaClone (Vue.js 3)
> Cole este prompt em uma nova conversa com a IA para construir o projeto do zero, módulo por módulo.

---

## 🎯 MISSÃO

Você é um desenvolvedor frontend sênior especialista em Vue.js 3, com experiência em arquitetura de SPAs e boas práticas de produção. Vamos construir juntos o **InstaClone**: uma rede social inspirada no Instagram, que é o projeto final da minha disciplina. O objetivo não é apenas fazer funcionar — é fazer **do jeito certo**, com código limpo, organizado e escalável.

---

## 🧱 STACK E RESTRIÇÕES TÉCNICAS

| Tecnologia | Detalhe |
|---|---|
| **Vue.js 3** | Obrigatório usar Composition API com `<script setup>` em todos os componentes |
| **Vue Router 4** | Navigation guards, rotas dinâmicas, lazy loading em todas as views |
| **Axios** | Instância centralizada com interceptors de request e response |
| **Vite** | Já configurado. Usar `import.meta.env` para variáveis de ambiente |
| **CSS** | Mobile-first. Com frameworks CSS externos (Bootstrap APENAS). Variáveis CSS nativas para o tema |
| **JavaScript** | ES2022+. `async/await` em todas as chamadas assíncronas. Sem `.then/.catch` encadeados |

**Restrições adicionais — não negocie nenhuma:**
- Nenhuma lógica de negócio dentro de componentes `.vue` — toda chamada à API vai em `src/services/`
- Lógica reutilizável entre componentes vai em `src/composables/` (Composition API)
- Nenhuma string de URL hardcoded — tudo via `import.meta.env.VITE_API_URL`
- Componentes com mais de 150 linhas devem ser divididos em sub-componentes
- Toda chamada assíncrona envolvida em `try/catch/finally`

---

## 📁 ARQUITETURA DE PASTAS (definida antes de qualquer código)

```
src/
├── assets/
│   └── main.css              # variáveis CSS globais + reset
├── components/
│   ├── ui/                   # componentes genéricos reutilizáveis (Avatar, Badge, Spinner...)
│   └── [feature]/            # componentes específicos de cada feature
├── composables/              # lógica reutilizável com Composition API
├── layouts/
│   └── MainLayout.vue
├── router/
│   └── index.js
├── services/
│   ├── api.js                # instância Axios centralizada
│   ├── authService.js
│   ├── postService.js
│   ├── storyService.js
│   ├── userService.js
│   ├── notificationService.js
│   └── exploreService.js
├── utils/
│   ├── date.js               # timeAgo(), formatDate()
│   └── format.js             # formatCount() ex: 1200 → "1,2k"
├── views/                    # uma view por rota
└── App.vue
```

---

## ⚙️ REGRAS DE EXECUÇÃO — SIGA RIGOROSAMENTE

1. **Um módulo por vez.** Não avance sem minha confirmação explícita ("pode continuar" ou "próximo").
2. **Antes de cada módulo**, mostre a lista exata de arquivos que serão criados ou modificados.
3. **Código 100% completo.** Proibido usar `// ...`, `/* resto do código */` ou qualquer omissão. Se um arquivo criado anteriormente precisar de atualização, mostre-o inteiro.
4. **Comentário obrigatório** em cada bloco não óbvio: explique O QUÊ e POR QUÊ aquela escolha foi feita (não apenas o que o código faz, mas a razão da decisão).
5. **Ao final de cada módulo**, entregue obrigatoriamente:
   - ✅ Checklist do que foi implementado
   - 🧪 Passo a passo de como testar manualmente no browser
   - ⚠️ Alertas: dependências do próximo módulo ou pontos de atenção
6. **Se uma decisão técnica tiver alternativas**, mencione qual foi escolhida e por quê (ex: "usei `URL.createObjectURL` ao invés de `FileReader` porque é mais rápido e não serializa o arquivo").
7. **Antes de iniciar**, pergunte sobre o `.env` e a estrutura de pastas existente.

---

## 📋 MÓDULOS — ORDEM OBRIGATÓRIA

---

### MÓDULO 0 — Fundação (`.env`, CSS e Axios)

**Por que começa aqui:** tudo depende dessas três peças. Sem elas, nenhum módulo funciona.

- Criar `.env.example`:
  ```
  VITE_API_URL=http://localhost:3000/api
  ```
- `src/assets/main.css` — variáveis CSS do tema Instagram + reset:
  ```css
  :root {
    --color-gradient-start: #833ab4;
    --color-gradient-mid: #fd1d1d;
    --color-gradient-end: #f77737;
    --color-primary: #0095f6;
    --color-bg: #fafafa;
    --color-surface: #ffffff;
    --color-text: #262626;
    --color-text-muted: #8e8e8e;
    --color-border: #dbdbdb;
    --radius-sm: 4px;
    --radius-md: 8px;
    --radius-full: 9999px;
  }
  ```
  Incluir: reset box-sizing, margin/padding zero, classe `.visually-hidden` para acessibilidade
- `src/services/api.js`:
  - Instância Axios com `baseURL: import.meta.env.VITE_API_URL`
  - Interceptor de **request**: se `localStorage.getItem('token')` existir, injeta `Authorization: Bearer <token>`
  - Interceptor de **response**: se status 401, remove token do localStorage e redireciona para `/login` usando `window.location.href`
  - Exportar como `export default`

---

### MÓDULO 1 — Roteamento Base

- `src/router/index.js`:
  - Todas as views com lazy loading: `component: () => import('@/views/...')`
  - Rotas **públicas** (sem token): `/login`, `/register`
  - Rotas **protegidas** (requerem token): `/feed`, `/explore`, `/create`, `/notifications`, `/post/:id`, `/profile/:username`, `/profile/:username/followers`, `/profile/:username/following`, `/stories/create`, `/profile/edit`
  - `beforeEach`: verifica token. Se rota protegida sem token → redireciona `/login`. Se rota pública com token → redireciona `/feed`
  - Rota `404`: redireciona para `/feed` ou exibe página "não encontrado"
- `src/App.vue`: apenas `<RouterView />` com transição CSS `fade` (opacity 0→1 em 200ms)
- `src/layouts/MainLayout.vue`:
  - `<slot name="header" />` — área opcional de header customizável por view
  - `<slot />` — conteúdo principal com padding correto
  - Inclui `<Navbar />` fixo
  - Padding-bottom no mobile para não sobrepor a navbar (`padding-bottom: 60px`)

---

### MÓDULO 2 — Autenticação

**Arquivos:** `src/services/authService.js`, `src/composables/useAuth.js`, `src/views/LoginView.vue`, `src/views/RegisterView.vue`

- `authService.js`:
  - `login(email, password)` → `POST /auth/login` → salva `token` e `user` no localStorage
  - `register(name, username, email, password)` → `POST /auth/register`
  - `logout()` → limpa localStorage (`token`, `user`) e redireciona para `/login`
  - `getToken()` → string ou `null`
  - `getCurrentUser()` → objeto usuário parseado do localStorage ou `null`
  - `isAuthenticated()` → booleano

- `useAuth.js` (composable — singleton compartilhado):
  - `user` como `ref` inicializado com `getCurrentUser()`
  - `isLoading` e `error` como `ref`
  - Funções: `login()`, `logout()` (delegam para authService e atualizam `user`)
  - Exportado como instância criada **fora** da função (singleton)

- `LoginView.vue` e `RegisterView.vue`:
  - Validação client-side antes de qualquer requisição
  - Botão com `Spinner` durante loading (desabilitado)
  - Erro da API exibido como mensagem abaixo do formulário
  - Design: centralizado, logo do InstaClone (texto estilizado), visual limpo inspirado no Instagram
  - `RegisterView`: senha mínimo 6 chars, confirmação, username só `[a-zA-Z0-9_]`

---

### MÓDULO 3 — Navbar e Componentes UI Base

**Arquivos:** `src/components/Navbar.vue`, `src/components/ui/Avatar.vue`, `src/components/ui/Badge.vue`, `src/components/ui/Spinner.vue`

- `Avatar.vue`: props `src`, `size` (sm=32px / md=44px / lg=80px), `alt`. Fallback: exibe inicial do nome em fundo colorido se imagem falhar (`@error`)
- `Badge.vue`: props `count`. Exibe número se `1-99`, "+99" se maior, nada se `0`
- `Spinner.vue`: animação CSS `spin`, tamanhos sm/md/lg, cor via `currentColor`

- `Navbar.vue`:
  - **Mobile** (`< 768px`): `position: fixed`, `bottom: 0`, largura 100%, 5 ícones SVG
  - **Desktop** (`≥ 768px`): sidebar `position: fixed`, `left: 0`, altura 100vh, ícone + label
  - Ícone de notificações com `<Badge :count="unreadCount" />` (valor vindo do `useNotifications` — pode ser `0` por ora)
  - Ícone de perfil usa `<Avatar />` com foto do usuário logado
  - Link ativo visualmente diferenciado (ícone preenchido vs outline)
  - SVG inline para os ícones (Home, Search, Plus, Heart, User)

---

### MÓDULO 4 — Feed + Scroll Infinito

**Arquivos:** `src/services/postService.js`, `src/composables/useInfiniteScroll.js`, `src/utils/date.js`, `src/components/PostCard.vue`, `src/views/FeedView.vue`

- `postService.js`: `getFeed(page)`, `likePost(id)`, `unlikePost(id)`, `addComment(postId, text)`, `getPostById(id)`

- `useInfiniteScroll.js`:
  - Parâmetros: `fetchFn` (função que recebe página e retorna `{ data, hasMore }`), `options` (opcional)
  - Retorna: `items` (array acumulado), `isLoading`, `hasMore`, `error`, `reset()`
  - Usa `IntersectionObserver` em um elemento sentinela (`ref` exposto como `sentinel`)
  - Limpa o observer no `onUnmounted`

- `src/utils/date.js`: função `timeAgo(dateString)` → "agora", "há 3 minutos", "há 2 horas", "há 4 dias", "há 2 semanas"

- `PostCard.vue` — componente completo:
  - **Header**: `<Avatar />` + username clicável → `/profile/:username` + ícone "..." (placeholder)
  - **Imagem**: `loading="lazy"`, `aspect-ratio: 1/1`, `object-fit: cover`
  - **Actions**: botão curtir com animação de escala (❤️ vazio/preenchido), botão comentar
  - **Contagem**: "X curtidas"
  - **Legenda**: `<strong>username</strong> texto` — trunca com "... mais" se > 120 chars, expande ao clicar
  - **Preview de comentários**: até 2 comentários abreviados
  - **Campo de comentário inline**: input + botão "Publicar" (aparece quando input tem foco)
  - **Data**: `timeAgo(post.createdAt)` em cinza
  - **Curtir com atualização otimista**: inverte `isLiked` e `likesCount` imediatamente; reverte em caso de erro da API

- `FeedView.vue`:
  - `<StoriesBar />` no topo (placeholder que será substituído no Módulo 5)
  - Lista de `<PostCard />` via `useInfiniteScroll`
  - Elemento `<div ref="sentinel">` no fim da lista para trigger do observer
  - Skeleton loader (3 cards) durante carregamento inicial
  - Mensagem vazia: "Siga pessoas para ver posts aqui 📸"

---

### MÓDULO 5 — Stories

**Arquivos:** `src/services/storyService.js`, `src/components/StoriesBar.vue`, `src/components/StoryViewer.vue`, `src/views/CreateStoryView.vue`

- `storyService.js`: `getStories()`, `markStoryViewed(id)`, `createStory(formData)`

- `StoriesBar.vue`:
  - Scroll horizontal sem scrollbar visível
  - Primeiro item: "Seu story" com Avatar + ícone `+` azul. Clica → navega para `/stories/create`
  - Stories não vistos: borda gradiente (`--color-gradient-start` → `--color-gradient-end`) via `border` + `background-clip`
  - Stories vistos: borda `--color-border`
  - Ao clicar em story: emite `@open-story` com índice do story
  - `FeedView` recebe o evento e abre `StoryViewer`

- `StoryViewer.vue`:
  - `position: fixed`, fullscreen, `z-index: 9999`, fundo preto
  - Barra de progresso: `n` segmentos (um por story), preenchimento via CSS `transition: width linear 5s`
  - Navegação: área esquerda clicável (anterior) + área direita clicável (próximo)
  - Botão `X` para fechar
  - `onMounted`: inicia timer de 5s para avançar. `onUnmounted`: `clearTimeout`
  - Chama `markStoryViewed(id)` ao exibir cada story
  - Ao fechar: emite `@close`

- `CreateStoryView.vue`:
  - Input `type="file"` `accept="image/*"` + botão estilizado que o dispara
  - Preview em `aspect-ratio: 9/16` (vertical, como stories reais)
  - Botão "Compartilhar" → `FormData` → `POST /stories`
  - Loading + feedback de sucesso/erro
  - Redireciona para `/feed` após 1.5s de sucesso

---

### MÓDULO 6 — Criar Post

**Arquivos:** `src/views/CreatePostView.vue`

- Fluxo em 3 etapas visuais (não são rotas separadas, são estados internos via `ref`):

  **Etapa 1 — Seleção:**
  - Área de drag-and-drop estilizada com ícone e texto "Arraste ou clique para selecionar"
  - Input `type="file"` `accept="image/*"` oculto, ativado pelo clique na área
  - Ao selecionar: avança para Etapa 2

  **Etapa 2 — Edição:**
  - Preview quadrado (`aspect-ratio: 1/1`) gerado com `URL.createObjectURL()`
  - Textarea para legenda: contador `X / 2200` caracteres
  - Botão "Voltar" (retorna à Etapa 1 e limpa seleção) + Botão "Compartilhar"
  - Validação: imagem obrigatória, legenda máximo 2200 chars

  **Etapa 3 — Publicando:**
  - `POST /posts` com `FormData` (campo `image` + campo `caption`)
  - Exibe `<Spinner />` durante upload
  - Sucesso: ícone ✅ + "Post publicado!" + redireciona para `/feed` após 1.5s
  - Erro: mensagem + botão "Tentar novamente" (volta para Etapa 2 com dados preservados)

---

### MÓDULO 7 — Perfil de Usuário

**Arquivos:** `src/services/userService.js`, `src/composables/useProfile.js`, `src/views/ProfileView.vue`, `src/views/EditProfileView.vue`, `src/views/FollowersView.vue`, `src/views/FollowingView.vue`

- `userService.js`: `getProfile(username)`, `followUser(id)`, `unfollowUser(id)`, `updateProfile(formData)`, `getFollowers(username, page)`, `getFollowing(username, page)`, `getUserPosts(username, page)`

- `useProfile.js`:
  - Recebe `username` reativo (da rota)
  - Expõe: `profile`, `posts`, `isLoading`, `isOwnProfile` (compara com `getCurrentUser().id`), `isFollowing`, `toggleFollow()`
  - `watch(() => route.params.username, fetchAll, { immediate: true })` — recarrega ao trocar de perfil sem sair do componente

- `ProfileView.vue`:
  - Header: `<Avatar size="lg">` + nome + username + bio + contadores (`posts` / `seguidores` / `seguindo` — os dois últimos são links clicáveis)
  - Botão "Seguir / Seguindo" com atualização otimista se perfil alheio
  - Botão "Editar perfil" se perfil próprio
  - Guia de tabs: "Posts" (grid) — pode deixar só esta por ora
  - Grid 3 colunas de miniaturas com `aspect-ratio: 1/1` e `object-fit: cover`
  - Skeleton durante carregamento

- `EditProfileView.vue`:
  - Avatar clicável → abre `input[type=file]` oculto → preview imediato
  - Campos: nome, username, bio (textarea 150 chars)
  - `PATCH /users/me` com `FormData` se houver nova imagem, ou `application/json` se não houver
  - Atualiza `localStorage` com dados do usuário após sucesso

- `FollowersView.vue` / `FollowingView.vue`:
  - Recebe `:username` da rota
  - Lista paginada: `<Avatar />` + nome + username + botão "Seguir/Seguindo"

---

### MÓDULO 8 — Explorar + Busca com Debounce

**Arquivos:** `src/services/exploreService.js`, `src/composables/useDebounce.js`, `src/views/ExploreView.vue`

- `useDebounce.js`:
  - `function useDebounce(value, delay)` — recebe um `ref` e retorna um `ref` debounced
  - Implementar com `watch` + `setTimeout/clearTimeout`
  - Limpar timeout em `onUnmounted`

- `exploreService.js`: `getPopularPosts(page)`, `searchUsers(query)`

- `ExploreView.vue`:
  - Barra de busca sticky no topo (input com ícone de lupa e `×` para limpar)
  - `searchQuery` como `ref`, debounce de 400ms via `useDebounce`
  - `watch(debouncedQuery, ...)` dispara busca quando valor muda

  **Estado A — sem busca** (searchQuery vazio):
  - Grid de posts populares com scroll infinito via `useInfiniteScroll`
  - Colunas variadas (2 ou 3, com destaque aleatório a cada 3 itens)

  **Estado B — com busca** (searchQuery com texto):
  - Lista de usuários encontrados: `<Avatar />` + nome + username + botão seguir
  - Estado vazio: "Nenhum usuário encontrado para '[query]'"
  - Transição animada entre estados A e B

---

### MÓDULO 9 — Notificações + Polling

**Arquivos:** `src/services/notificationService.js`, `src/composables/useNotifications.js`, `src/views/NotificationsView.vue`

- `notificationService.js`: `getNotifications(page)`, `markAsRead(id)`, `markAllAsRead()`, `getUnreadCount()`

- `useNotifications.js` (singleton — instância criada fora do composable, compartilhada entre Navbar e NotificationsView):
  - `unreadCount` como `ref`
  - `startPolling(intervalMs = 30000)`: `setInterval` chamando `getUnreadCount()`, salva o id do interval
  - `stopPolling()`: `clearInterval`
  - `refreshCount()`: atualiza `unreadCount` imediatamente

- **Integração na Navbar**: `onMounted` → `startPolling()`, `onUnmounted` → `stopPolling()`. `<Badge :count="unreadCount" />`

- `NotificationsView.vue`:
  - Ao entrar na view: chama `markAllAsRead()` e zera `unreadCount` localmente
  - Botão "Marcar todas como lidas" no header
  - Lista de notificações paginada ("carregar mais")
  - Ícone e texto por tipo:
    - ❤️ curtida: "[username] curtiu sua foto." → clica → `/post/:postId`
    - 💬 comentário: "[username] comentou: '[preview 30 chars]...'" → clica → `/post/:postId`
    - 👤 seguidor: "[username] começou a seguir você." → clica → `/profile/:username`
  - Não lidas: fundo `#fafafa`, lidas: fundo branco
  - Data relativa com `timeAgo()`

---

### MÓDULO 10 — Detalhes do Post

**Arquivos:** `src/views/PostDetailView.vue`, `src/components/CommentList.vue`, `src/components/CommentInput.vue`

- `PostDetailView.vue` (rota `/post/:id`):
  - Carrega post via `getPostById(id)` no `onMounted`
  - Layout:
    - **Mobile**: vertical (imagem → info autor → actions → legenda → comentários → input)
    - **Desktop**: lado a lado (imagem esquerda | painel direito com info + comentários + input)
  - Header do painel: `<Avatar />` + username (link pro perfil) + botão "..." (se dono: abre opção deletar)
  - Actions: curtir (atualização otimista) + contagem
  - Legenda completa
  - `<CommentList :postId="id" />`
  - `<CommentInput :postId="id" @comment-added="prependComment" />` — fixo no bottom do painel
  - Modal de confirmação antes de deletar: "Tem certeza que deseja excluir este post?" + botões Cancelar/Excluir

- `CommentList.vue`:
  - Carrega comentários paginados (`getComments(postId, page)` — adicionar ao `postService`)
  - Acumula no array com `push(...newItems)` (não substitui)
  - Botão "Ver mais comentários" (hidden quando `hasMore = false`)
  - Item: `<Avatar size="sm" />` + `<strong>username</strong>` + texto + `timeAgo()`

- `CommentInput.vue`:
  - Input simples, placeholder "Adicione um comentário..."
  - Botão "Publicar" desabilitado quando vazio
  - Enter (sem Shift) também envia
  - Após envio: limpa input, emite `comment-added` com o objeto do novo comentário
  - O pai (`PostDetailView`) adiciona o comentário no início do array local (`unshift`) — sem refetch

---

### MÓDULO FINAL — Revisão, Polimento e Acessibilidade

Após todos os módulos prontos, faça um sweep completo:

**Funcional:**
- Verificar se todos os `onUnmounted` têm limpeza de timers e observers
- Verificar se todas as rotas protegidas realmente bloqueiam acesso sem token
- Verificar navegação entre `/profile/joao` e `/profile/maria` sem bugs (o `watch` no `useProfile`)
- Testar fluxo completo: registro → login → criar post → ver no feed → entrar no detalhe → comentar → deletar

**Visual:**
- Responsividade em 375px (iPhone SE) e 1440px (desktop)
- Consistência de espaçamentos usando as variáveis CSS definidas no Módulo 0
- Todos os estados de loading e erro cobertos visualmente

**Acessibilidade:**
- Todos os botões de ícone com `aria-label`
- Imagens com `alt` descritivo
- Inputs com `label` associado (mesmo que visualmente oculto com `.visually-hidden`)

---

## 🚦 INICIE AGORA

Antes de escrever qualquer código, me faça estas perguntas:
1. Já tenho o `.env` configurado com `VITE_API_URL`?
2. A estrutura de pastas já existe ou precisa ser criada do zero?
3. Qual é a URL base da API que vou usar durante o desenvolvimento?

Após minhas respostas, comece pelo **MÓDULO 0**.
