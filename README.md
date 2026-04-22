# 🚀 PROMPT DE EXECUÇÃO — InstaClone (Vue.js 3 + Pinia + API Real)
> Cole este prompt em uma nova conversa com a IA para construir o projeto do zero, módulo por módulo.
> Este prompt é uma especificação de build: nomes de rota, endpoints e limites são contratos com o backend e devem ser respeitados exatamente.

---

## 🎯 MISSÃO

Você é um desenvolvedor frontend sênior especialista em Vue.js 3. Vamos construir o **InstaClone**: uma SPA inspirada no Instagram, projeto final de disciplina, que **consome uma API RESTful real**. Cada endpoint, rota e validação descrita aqui é um contrato fixo com o backend — não invente, não renomeie, não substitua.

---

## 🧱 STACK E RESTRIÇÕES TÉCNICAS

| Tecnologia | Detalhe |
|---|---|
| **Vue.js 3 + Vite** | Composition API com `<script setup>` em todos os componentes |
| **Vue Router 4** | History mode. Guards `requiresAuth` e `requiresGuest`. Lazy loading em todas as views |
| **Pinia** | Stores: `auth` e `feed`. Estado global apenas aqui |
| **Axios** | Instância centralizada em `src/services/api.js`. baseURL de `import.meta.env.VITE_API_URL` |
| **Bootstrap** | Importado no `main.js` **antes** do tema customizado |
| **CSS** | Tema global em `src/assets/styles/theme.css`, importado no `main.js` **após** o Bootstrap |
| **Docker** | Dockerfile multi-stage + nginx + compose.yaml entregues ao final |

**Restrições inegociáveis:**
- Estado global → sempre em Pinia stores (`auth`, `feed`)
- Estado local (form fields, preview, feedback momentâneo) → `ref`/`reactive` na view, nunca no store
- Nenhuma URL de API hardcoded — sempre `import.meta.env.VITE_API_URL`
- Token salvo no `localStorage` com chave fixa `instaclone.token`
- Componentes com mais de 150 linhas devem ser divididos
- Toda operação assíncrona envolve `try/catch/finally`
- Validações de arquivo feitas no **cliente** antes de qualquer requisição

---

## 📁 ARQUITETURA DE PASTAS

```
src/
├── assets/
│   └── styles/
│       └── theme.css           # tema global (importado após Bootstrap no main.js)
├── components/
│   ├── ui/                     # Avatar, Spinner, ConfirmModal — genéricos
│   └── [feature]/              # componentes específicos de cada feature
├── composables/                # lógica reutilizável (sem estado global)
├── layouts/
│   ├── AuthLayout.vue          # layout para /login e /cadastro
│   └── AppLayout.vue           # layout autenticado com navbar
├── router/
│   └── index.js
├── services/
│   └── api.js                  # instância Axios centralizada
├── stores/
│   ├── auth.js                 # Pinia: sessão, user, token, actions de auth
│   └── feed.js                 # Pinia: posts normalizados, curtidas, comentários
├── utils/
│   ├── date.js                 # timeAgo()
│   └── format.js               # formatCount()
├── views/
│   ├── auth/
│   │   ├── LoginView.vue
│   │   └── CadastroView.vue
│   ├── FeedView.vue
│   ├── DescubrirView.vue
│   ├── CriarPostView.vue
│   ├── PerfilView.vue
│   ├── EditarPerfilView.vue
│   ├── ListaConexoesView.vue
│   ├── PostDetailView.vue
│   └── NotFoundView.vue
└── App.vue
```

---

## ⚙️ REGRAS DE EXECUÇÃO — SIGA RIGOROSAMENTE

1. **Um módulo por vez.** Não avance sem minha confirmação explícita.
2. **Antes de cada módulo**, liste exatamente quais arquivos serão criados ou modificados.
3. **Código 100% completo.** Proibido usar `// ...` ou qualquer omissão. Se um arquivo anterior precisar ser atualizado, mostre-o inteiro.
4. **Comentário obrigatório** em cada decisão não óbvia: O QUÊ e POR QUÊ.
5. **Ao final de cada módulo**, entregue:
   - ✅ Checklist do que foi implementado
   - 🧪 Como testar manualmente no browser (passo a passo)
   - ⚠️ Alertas e dependências do próximo módulo
6. **Endpoints são contratos** — use exatamente o método HTTP e a rota especificados. Não adapte.
7. **Antes de iniciar**, pergunte se o projeto já tem estrutura de pastas ou é do zero, e se o `.env` já está configurado.

---

## 📋 MÓDULOS — ORDEM OBRIGATÓRIA DE IMPLEMENTAÇÃO

---

### MÓDULO 0 — Setup do Projeto

**O que criar:**

- `.env.example`:
  ```
  VITE_API_URL=http://localhost:8000/api
  ```

- `.dockerignore`: excluindo `node_modules`, `dist`, `.env`; preservando `.env.example`

- `src/assets/styles/theme.css`: variáveis CSS do tema Instagram:
  ```css
  :root {
    --color-gradient-start: #833ab4;
    --color-gradient-mid:   #fd1d1d;
    --color-gradient-end:   #f77737;
    --color-primary:        #0095f6;
    --color-bg:             #fafafa;
    --color-surface:        #ffffff;
    --color-text:           #262626;
    --color-text-muted:     #8e8e8e;
    --color-border:         #dbdbdb;
    --radius-sm:            4px;
    --radius-md:            8px;
    --radius-full:          9999px;
  }
  ```
  Reset box-sizing, margin/padding zero, `font-family: system-ui`, `.visually-hidden`.

- `main.js`: importa Bootstrap **primeiro**, depois `./assets/styles/theme.css`, depois monta o app com Pinia e Router

- `src/services/api.js`:
  - Instância Axios: `baseURL: import.meta.env.VITE_API_URL`
  - Interceptor de **request**: se `localStorage.getItem('instaclone.token')` existir → injeta `Authorization: Bearer <token>`
  - Interceptor de **response**: se status `401` → remove `instaclone.token` do localStorage → `window.location.href = '/login'`
  - Exportar como `export default`

- `src/utils/date.js`: `timeAgo(dateString)` → "agora", "há X min", "há Xh", "há X dias"
- `src/utils/format.js`: `formatCount(n)` → "1,2k", "4,5M"

---

### MÓDULO 1 — Stores Pinia

**`src/stores/auth.js`:**
- Estado: `user` (objeto|null), `token` (string|null)
- `isAuthenticated`: computed → `!!token`
- `init()`: lê `instaclone.token` do localStorage → se existir, chama `fetchMe()`
- `login(email, password)`:
  - `POST /auth/login` → recebe `{ access_token, user }`
  - Salva token em `localStorage` com chave `instaclone.token`
  - Atualiza `token` e `user` no estado
- `register(name, username, email, password, password_confirmation)`:
  - `POST /auth/register` → cria conta e já autentica
  - Salva token e user igual ao login
- `logout()`:
  - `POST /auth/logout` (mesmo se token inválido, captura o erro silenciosamente)
  - Remove `instaclone.token` do localStorage
  - Zera `user` e `token` no estado
- `fetchMe()`:
  - `GET /auth/me` → atualiza `user` no estado
  - Se falhar → chama `logout()`
- `updateProfile(data)`: atualiza `user` no estado local após edição de perfil bem-sucedida

**`src/stores/feed.js`:**
- Estado: `postsById` (objeto/dicionário `{ [id]: post }`), `feedOrder` (array de ids ordenado), `nextCursor` (string|null), `isLoading` (boolean)
- `fetchFeed()`: `GET /feed` → normaliza posts em `postsById`, preenche `feedOrder`, salva `nextCursor`
- `loadMoreFeed()`: `GET /feed?cursor=<nextCursor>` → acumula em `postsById` e `feedOrder`, atualiza `nextCursor`
- `toggleLike(postId)`:
  - Atualização otimista: inverte `isLiked` e ajusta `likesCount` em `postsById[postId]` imediatamente
  - Se `isLiked` → `POST /posts/:id/like`. Se não → `DELETE /posts/:id/unlike`
  - Em caso de erro: reverte o estado
- `addComment(postId, body)`: `POST /posts/:id/comments` com `{ body }` → adiciona comentário ao post em `postsById`
- `createPost(formData)`: `POST /posts` com FormData → adiciona post ao início de `feedOrder` e `postsById`
- `removePost(postId)`: remove de `postsById` e `feedOrder`
- Getter `getPostById(id)`: retorna `postsById[id]`
- Getter `feedPosts`: retorna `feedOrder.map(id => postsById[id])`

---

### MÓDULO 2 — Roteamento

**`src/router/index.js`:**

Rotas com seus layouts:

```
/login          → AuthLayout > LoginView        (requiresGuest)
/cadastro       → AuthLayout > CadastroView     (requiresGuest)
/feed           → AppLayout  > FeedView         (requiresAuth)
/descobrir      → AppLayout  > DescubrirView    (requiresAuth)
/criar          → AppLayout  > CriarPostView    (requiresAuth)
/perfil         → AppLayout  > PerfilView       (requiresAuth)  [?user=<username>]
/perfil/editar  → AppLayout  > EditarPerfilView (requiresAuth)
/perfil/lista/:type → AppLayout > ListaConexoesView (requiresAuth)
/posts/:postId  → AppLayout  > PostDetailView   (requiresAuth)
/:pathMatch(.*)*→ NotFoundView  (sem layout, acessível a todos)
```

- Todas as views com lazy loading: `component: () => import('@/views/...')`
- Guard `requiresAuth`: sem token (`authStore.isAuthenticated === false`) → `next('/login')`
- Guard `requiresGuest`: com token → `next('/feed')`
- Rota raiz `/` redireciona para `/feed`

**`src/App.vue`:**
- Chama `authStore.init()` no `onMounted`
- `<RouterView />` com transição `fade` (opacity 0→1, 200ms)

---

### MÓDULO 3 — Layouts e Navbar

**`src/layouts/AuthLayout.vue`:**
- Layout centralizado para telas de visitante
- Logo "InstaClone" estilizado com gradiente no topo
- `<slot />` para o formulário
- Sem navbar

**`src/layouts/AppLayout.vue`:**
- `<Navbar />` responsivo + `<slot />`
- Padding-bottom `60px` no mobile para não sobrepor navbar

**`src/components/Navbar.vue`:**
- **Mobile** (`< 768px`): `position: fixed`, `bottom: 0`, largura 100%, 4 ícones SVG
- **Desktop** (`≥ 768px`): sidebar `position: fixed`, `left: 0`, altura 100vh, ícone + label
- 4 entradas de navegação (exatamente estas, nesta ordem):
  1. **Home** → `/feed`
  2. **Buscar** → `/descobrir`
  3. **Criar** → `/criar`
  4. **Perfil** → `/perfil` (sem query param — perfil próprio)
- Ícone de Perfil: `<Avatar size="sm" />` com foto do `authStore.user`
- Link ativo: ícone preenchido vs outline via `.router-link-active`
- SVG inline para todos os ícones (Home, Search, Plus, User)

**`src/RouterView` no AppLayout:**
- Usar `v-slot="{ Component }"` + `<component :is="Component" />` para troca de views (conforme especificado no README)

**Componentes UI base:**
- `src/components/ui/Avatar.vue`: props `src`, `size` (sm=32/md=44/lg=80px), `alt`. Fallback: inicial em círculo colorido se imagem falhar
- `src/components/ui/Spinner.vue`: animação CSS spin, tamanhos sm/md/lg, cor via `currentColor`
- `src/components/ui/ConfirmModal.vue`: props `title`, `message`, `confirmLabel`, `cancelLabel`. Emite `confirm`, `cancel`. Overlay `position: fixed`, `z-index: 1000`

---

### MÓDULO 4 — Autenticação

**`src/views/auth/LoginView.vue`:**
- Campos: `email` (type email) + `password`
- Validação client-side: ambos obrigatórios, formato de email
- Submit: `authStore.login(email, password)`
- Sucesso: `router.replace('/feed')`
- Erro: mensagem inline abaixo do formulário (texto vindo da API)
- Botão desabilitado + `<Spinner />` durante loading
- Link para `/cadastro`

**`src/views/auth/CadastroView.vue`:**
- Campos: `name`, `username`, `email`, `password`, `password_confirmation`
- Validações client-side:
  - Todos obrigatórios
  - `username`: regex `^[A-Za-z0-9._]+$`, máx 30 chars
  - `password`: mínimo 6 chars
  - `password_confirmation`: deve ser igual a `password`
- Submit: `authStore.register(name, username, email, password, password_confirmation)`
- Sucesso: `router.replace('/feed')`
- Erros por campo vindos do backend exibidos inline abaixo de cada input
- Link para `/login`

---

### MÓDULO 5 — Feed (`/feed`)

**`src/views/FeedView.vue`:**
- `onMounted`: chama `feedStore.fetchFeed()`
- Lista `feedStore.feedPosts` com `<PostCard />` para cada post
- Skeleton loader (3 cards) durante `feedStore.isLoading` no carregamento inicial
- Botão "Carregar mais" visível enquanto `feedStore.nextCursor !== null`
  - Ao clicar: chama `feedStore.loadMoreFeed()`
- Mensagem vazia: "Ainda não há posts para exibir." se feed vazio e não loading

**`src/components/PostCard.vue`:**
- **Header**: `<Avatar size="md" />` + username clicável → `/perfil?user=<username>` + nome
- **Imagem**: `loading="lazy"`, `aspect-ratio: 1/1`, `object-fit: cover`
- **Actions**: botão curtir (❤️ vazio/preenchido com animação de escala), contador de likes com `formatCount()`
- **Legenda**: `<strong>username</strong> texto` — trunca com "... mais" se > 120 chars
- **Contador de comentários**: "X comentários" como link para `/posts/:id`
- **Campo inline de comentário**: input simples → Enter envia → chama `feedStore.addComment(postId, body)`
- **Data**: `timeAgo(post.createdAt)` em `--color-text-muted`
- **Curtir**: chama `feedStore.toggleLike(postId)` — atualização otimista já no store

---

### MÓDULO 6 — Descobrir (`/descobrir`)

**`src/views/DescubrirView.vue`:**
- `onMounted`: em paralelo (Promise.all):
  - `GET /users/suggestions` → lista de perfis sugeridos
  - `GET /users/:viewerId/following` → ids de quem o viewer já segue (para marcar estado do botão)
- Grid ou lista de cards de usuários sugeridos
- Cada card: `<Avatar />` + nome + username + botão "Seguir" / "Seguindo"
  - Seguir: `POST /users/:id/follow` → atualiza estado local
  - Deixar de seguir: `DELETE /users/:id/unfollow` → atualiza estado local
- Clique no card (fora do botão): navega para `/perfil?user=<username>` — se for o próprio usuário, navega para `/perfil`
- Paginação por página: botão "Carregar mais" → `GET /users/suggestions?page=<n>`

---

### MÓDULO 7 — Criar Post (`/criar`)

**`src/views/CriarPostView.vue`** — fluxo em 3 estados internos (`ref` local):

**Estado 1 — Seleção:**
- Área de drag-and-drop estilizada
- `<input type="file" accept="image/jpeg,image/jpg,image/png,image/webp">` oculto, ativado pelo clique
- Validações no cliente **antes** de avançar:
  - Tipo: apenas `image/jpeg`, `image/jpg`, `image/png`, `image/webp`
  - Tamanho: máximo **5 MB** (`file.size <= 5 * 1024 * 1024`)
  - Exibir mensagem de erro se inválido, não avançar
- Ao passar validação: gerar preview com `URL.createObjectURL(file)`, avançar para Estado 2

**Estado 2 — Edição:**
- Preview quadrado (`aspect-ratio: 1/1`) via `object-url`
- Textarea legenda: contador `X / 2200` chars, máx 2200
- Botão "Compartilhar": **desabilitado** enquanto imagem ou legenda ausentes
- Botão "Voltar": revoga blob (`URL.revokeObjectURL`), limpa seleção, volta ao Estado 1
- `onUnmounted`: revogar blob se ainda existir

**Estado 3 — Publicando:**
- Monta `FormData` com campos `image` (File) e `caption` (string)
- Chama `feedStore.createPost(formData)`
- Exibe `<Spinner />` durante upload
- Sucesso: "Post publicado!" → `router.replace('/feed')` após 1.5s
- Erro: mensagem + botão "Tentar novamente" (preserva dados do Estado 2)

---

### MÓDULO 8 — Perfil (`/perfil` e `/perfil?user=<username>`)

**`src/composables/useProfile.js`:**
- Lê `route.query.user` para determinar o username alvo. Se ausente, usa `authStore.user.username`
- `isOwnProfile`: computed → username alvo === `authStore.user.username`
- Em paralelo ao montar (Promise.all):
  - `GET /users/{username}` → dados do perfil
  - `GET /users/{id}/posts` → posts do grid
  - `GET /users/{id}/followers` → contagem
  - `GET /users/{id}/following` → contagem
  - Se `!isOwnProfile`: `GET /users/{id}/is-following` → estado do botão
- `toggleFollow()`:
  - Se seguindo → `DELETE /users/:id/unfollow`
  - Se não → `POST /users/:id/follow`
  - Atualização otimista do contador e estado
- `watch(() => route.query.user, fetchAll, { immediate: true })` — recarrega ao trocar `?user=`

**`src/views/PerfilView.vue`:**
- Usa `useProfile()`
- Header: `<Avatar size="lg" />` + nome + username + bio
- Contadores clicáveis:
  - Posts: número (sem link)
  - Seguidores → `/perfil/lista/seguidores?user=<username>` (ou sem `?user=` se próprio)
  - Seguindo → `/perfil/lista/seguindo?user=<username>`
- Botão "Seguir / Seguindo" se `!isOwnProfile`, com atualização otimista
- Botão "Editar perfil" se `isOwnProfile` → navega para `/perfil/editar`
- Grid 3 colunas (`grid-template-columns: repeat(3, 1fr)`) de miniaturas dos posts
  - `aspect-ratio: 1/1`, `object-fit: cover`
  - Clique → `/posts/:postId`
- Skeleton durante carregamento

---

### MÓDULO 9 — Editar Perfil (`/perfil/editar`)

**`src/views/EditarPerfilView.vue`:**

**Seção avatar:**
- `<Avatar size="lg" />` clicável → dispara `<input type="file" accept="image/*">` oculto
- Validação cliente: máximo **2 MB** (`file.size <= 2 * 1024 * 1024`)
- Preview imediato com `URL.createObjectURL`
- Ao salvar avatar separadamente: `POST /users/me/avatar` com `FormData` contendo `avatar`

**Seção dados:**
- Campos pré-preenchidos com dados de `authStore.user`:
  - `name`: máx 255 chars
  - `username`: máx 30 chars, regex `^[A-Za-z0-9._]+$`
  - `bio`: textarea, máx 500 chars, contador visível
- Validações acima executadas no cliente antes do submit
- Submit: `PUT /users/me` com `{ name, username, bio }`
- Após sucesso: `authStore.updateProfile(responseData)` → `router.replace('/perfil')`
- Erros por campo vindos do backend exibidos inline abaixo de cada input
- Botão "Cancelar" → `router.back()`

---

### MÓDULO 10 — Listas de Conexão (`/perfil/lista/:type`)

**`src/views/ListaConexoesView.vue`:**
- `:type` aceita `seguidores` ou `seguindo` (qualquer outro valor → redireciona para `/perfil`)
- Lê `route.query.user` para saber de qual perfil listar. Se ausente, usa o próprio usuário
- `GET /users/{id}/followers` ou `GET /users/{id}/following` conforme `:type`
- Paginação por página: botão "Carregar mais" → `?page=<n>` acumulando itens
- Cada linha: `<Avatar size="md" />` + nome + username (link para `/perfil?user=<username>`) + botão "Seguir / Seguindo"
  - Seguir: `POST /users/:id/follow`
  - Deixar de seguir: `DELETE /users/:id/unfollow`
- Header da tela: título "Seguidores" ou "Seguindo" + botão voltar → `/perfil?user=<username>` (ou `/perfil` se próprio)

---

### MÓDULO 11 — Detalhes do Post (`/posts/:postId`)

**`src/components/CommentInput.vue`:**
- Input placeholder "Adicione um comentário..."
- Botão "Publicar" desabilitado quando input vazio
- Enter (sem Shift) também envia
- Emite `comment-added` com o body e limpa o campo

**`src/views/PostDetailView.vue`:**
- `onMounted` em paralelo:
  - `GET /posts/:id` → dados do post
  - `GET /posts/:id/comments` → primeira página de comentários
- Se post não encontrado → redireciona para `/feed`

- **Layout mobile** (vertical): imagem → header autor → actions → legenda → comentários → input
- **Layout desktop** (lado a lado): imagem esquerda | painel direito (header + legenda + lista comentários com scroll + input fixo no bottom)

- Header: `<Avatar />` + username (link → `/perfil?user=<username>`) + nome
- Actions: botão curtir (via `feedStore.toggleLike`) + contagem de likes com `formatCount()`
- Legenda completa (sem truncamento)
- Lista de comentários paginada por página:
  - Item: `<Avatar size="sm" />` + `<strong>username</strong>` + texto + `timeAgo()`
  - **Botão deletar comentário** visível apenas para o autor do comentário (`comment.userId === authStore.user.id`):
    - `DELETE /comments/:id` → remove da lista local
  - Botão "Carregar mais" enquanto houver próxima página
- `<CommentInput />` → ao emitir `comment-added`: `POST /posts/:id/comments` com `{ body }` → adiciona ao início da lista local
- **Botão deletar post** visível apenas para o autor do post:
  - Abre `<ConfirmModal title="Excluir post" message="Tem certeza que deseja excluir este post? Essa ação não pode ser desfeita." />`
  - Confirmar: `DELETE /posts/:id` → `feedStore.removePost(id)` → `router.replace('/feed')`

---

### MÓDULO 12 — 404 (`/:pathMatch(.*)*`)

**`src/views/NotFoundView.vue`:**
- Sem layout (renderiza direto no `App.vue`)
- Mensagem "Página não encontrada"
- Link condicional:
  - Se `authStore.isAuthenticated` → "Voltar ao feed" → `/feed`
  - Se não autenticado → "Fazer login" → `/login`

---

### MÓDULO 13 — Docker e Entrega

**`Dockerfile`** — multi-stage:
```dockerfile
# Stage 1: build
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL
RUN npm run build

# Stage 2: runtime
FROM nginx:1.27-alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY docker/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

**`docker/nginx.conf`:**
```nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```
Explicar por que `try_files ... /index.html` é necessário para o history mode do Vue Router.

**`compose.yaml`:**
```yaml
services:
  frontend:
    build:
      context: .
      args:
        VITE_API_URL: ${VITE_API_URL}
    ports:
      - "3000:80"
```

Verificar que `.dockerignore` exclui `node_modules`, `dist`, `.env` e preserva `.env.example`.

---

## 📊 TABELA DE ENDPOINTS (referência — não altere nenhum)

| Módulo | Método | Rota |
|---|---|---|
| Auth | POST | `/auth/login` |
| Auth | POST | `/auth/register` |
| Auth | POST | `/auth/logout` |
| Auth | GET | `/auth/me` |
| Feed | GET | `/feed?cursor=<c>` |
| Posts | POST | `/posts` (multipart) |
| Posts | GET | `/posts/:id` |
| Posts | DELETE | `/posts/:id` |
| Posts | POST | `/posts/:id/like` |
| Posts | DELETE | `/posts/:id/unlike` |
| Comments | GET | `/posts/:id/comments` |
| Comments | POST | `/posts/:id/comments` |
| Comments | DELETE | `/comments/:id` |
| Users | GET | `/users/suggestions` |
| Users | GET | `/users/:username` |
| Users | GET | `/users/:id/posts` |
| Users | GET | `/users/:id/followers` |
| Users | GET | `/users/:id/following` |
| Users | GET | `/users/:id/is-following` |
| Users | POST | `/users/:id/follow` |
| Users | DELETE | `/users/:id/unfollow` |
| Users | PUT | `/users/me` |
| Users | POST | `/users/me/avatar` (multipart) |

---

## MÓDULO FINAL — Revisão e Fluxo Completo

Após todos os módulos prontos, fazer um sweep completo:

**Fluxo obrigatório a testar:**
1. Acessar `/feed` sem token → redireciona para `/login`
2. Cadastrar nova conta → redireciona para `/feed`
3. Fazer logout → tenta acessar `/feed` → redireciona para `/login`
4. Fazer login → feed carrega posts
5. Curtir post → coração muda imediatamente (otimista) → requisição vai para API
6. Comentar inline no card → comentário aparece
7. Recarregar página (F5) → `authStore.init()` chama `GET /auth/me` → sessão é restaurada
8. Criar post com imagem > 5MB → erro de validação client-side, sem requisição
9. Criar post válido → aparece no feed
10. Abrir detalhe do post → comentar → deletar comentário → deletar o post
11. Acessar perfil alheio via `/perfil?user=<username>` → seguir → contador atualiza
12. Editar perfil (nome, bio, avatar) → Navbar atualiza com novos dados
13. Acessar `/qualquer-coisa-invalida` → NotFoundView com link correto conforme autenticação
14. Rodar `docker compose up` → app disponível em `http://localhost:3000`

**Acessibilidade mínima:**
- Todos os botões de ícone com `aria-label`
- Imagens com `alt` descritivo
- Inputs com `label` associado (mesmo que visualmente oculto via `.visually-hidden`)
- Responsividade em 375px (iPhone SE) e 1440px (desktop)

---

## 🚦 INICIE AGORA

Antes de escrever qualquer código, faça estas perguntas:
1. O projeto já tem estrutura de pastas ou é do zero?
2. O `.env` já está configurado com `VITE_API_URL`?
3. Bootstrap já está instalado (`npm install bootstrap`)?

Após minhas respostas, comece pelo **MÓDULO 0**.
