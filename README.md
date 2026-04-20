# 🚀 PROMPT DE EXECUÇÃO — InstaClone (Vue.js 3 + Pinia · Local-First)
> Cole este prompt em uma nova conversa com a IA para construir o projeto do zero, módulo por módulo.

---

## 🎯 MISSÃO

Você é um desenvolvedor frontend sênior especialista em Vue.js 3. Vamos construir juntos o **InstaClone**: uma SPA inspirada no Instagram, projeto final de disciplina.

**Modelo de funcionamento atual:** a aplicação opera em modo **local-first**. Autenticação, sessão, feed, curtidas, comentários e perfil ficam persistidos no `localStorage`. A integração com API RESTful real é preparada na camada de serviços (Axios + interceptors JWT), mas ainda não está ativa — o objetivo agora é ter uma aplicação 100% funcional rodando localmente, com dados mock realistas.

---

## 🧱 STACK E RESTRIÇÕES TÉCNICAS

| Tecnologia | Detalhe |
|---|---|
| **Vue.js 3** | Composition API com `<script setup>` em todos os componentes |
| **Vue Router 4** | Navigation guards, rotas dinâmicas, lazy loading em todas as views |
| **Pinia** | Gerenciamento de estado global. `src/stores/auth.js` e `src/stores/feed.js` |
| **Axios** | Instância centralizada configurada com interceptors JWT — pronta para API futura |
| **CSS** | Mobile-first. Sem frameworks CSS externos. Variáveis CSS nativas para o tema Instagram |
| **JavaScript** | ES2022+. `async/await` em todo código assíncrono. Sem `.then/.catch` encadeados |

**Restrições inegociáveis:**
- Estado **global** (sessão, feed, posts) → sempre em Pinia stores
- Estado **local** (campos de formulário, preview de upload, mensagens de feedback momentâneas) → `ref`/`reactive` dentro da view, nunca no store
- Nenhuma URL de API hardcoded — usar `import.meta.env.VITE_API_URL`
- Componentes com mais de 150 linhas devem ser divididos em sub-componentes
- Toda operação assíncrona envolve `try/catch/finally`

---

## 📁 ARQUITETURA DE PASTAS

```
src/
├── assets/
│   └── main.css              # variáveis CSS globais + reset
├── components/
│   ├── ui/                   # Avatar, Badge, Spinner — genéricos e reutilizáveis
│   └── [feature]/            # componentes específicos de cada feature
├── composables/              # lógica reutilizável sem estado global (useInfiniteScroll, useDebounce...)
├── layouts/
│   └── MainLayout.vue        # slots: header, main, footer + Navbar
├── router/
│   └── index.js
├── services/
│   └── api.js                # instância Axios centralizada (pronta para API futura)
├── stores/
│   ├── auth.js               # Pinia: sessão, usuário atual, contas locais
│   └── feed.js               # Pinia: posts, curtidas, comentários, criação
├── utils/
│   ├── date.js               # timeAgo(), formatDate()
│   └── format.js             # formatCount() ex: 1200 → "1,2k"
├── views/
└── App.vue
```

---

## ⚙️ REGRAS DE EXECUÇÃO — SIGA RIGOROSAMENTE

1. **Um módulo por vez.** Não avance sem minha confirmação explícita.
2. **Antes de cada módulo**, liste exatamente quais arquivos serão criados ou modificados.
3. **Código 100% completo.** Proibido usar `// ...` ou qualquer omissão. Se um arquivo anterior precisar ser atualizado, mostre-o inteiro.
4. **Comentário obrigatório** em cada decisão não óbvia: explique O QUÊ e POR QUÊ aquela escolha foi feita.
5. **Ao final de cada módulo**, entregue:
   - ✅ Checklist do que foi implementado
   - 🧪 Como testar manualmente no browser (passo a passo concreto)
   - ⚠️ Alertas e dependências do próximo módulo
6. **Se uma decisão técnica tiver alternativas**, mencione a escolhida e a razão (ex: "usei `URL.createObjectURL` ao invés de `FileReader` porque é mais performático para preview").
7. **Antes de iniciar**, pergunte sobre a estrutura de pastas atual do repositório.

---

## 📋 MÓDULOS — ORDEM OBRIGATÓRIA DE IMPLEMENTAÇÃO

---

### MÓDULO 0 — Fundação (CSS, Axios e Dados Mock)

**Por que começa aqui:** o tema visual, o cliente HTTP e os dados de teste são a base de tudo. Sem eles, nenhum módulo funciona corretamente.

**CSS Global — `src/assets/main.css`:**
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
Incluir: reset box-sizing, margin/padding zero, `font-family: system-ui`, classe `.visually-hidden`.

**Axios — `src/services/api.js`:**
- Instância com `baseURL: import.meta.env.VITE_API_URL`
- Interceptor de request: injeta `Authorization: Bearer <token>` se existir no `localStorage`
- Interceptor de response: se status 401, limpa token e redireciona para `/login`
- Exportar como `export default`
- Comentar claramente: *"Esta instância está pronta para quando a API real for conectada"*

**Dados Mock — `src/data/mockData.js`:**
- Array `MOCK_USERS` com 4 usuários: `{ id, username, name, avatar, bio, followersCount, followingCount }`
  - Usuário principal de teste: username `"usuario_teste"`, senha `"senha123"`
  - 3 usuários adicionais para popular o feed
- Array `MOCK_POSTS` com 6 posts distribuídos entre os usuários: `{ id, authorId, imageUrl, caption, location, likesCount, createdAt, comments: [] }`
  - Usar imagens de `https://picsum.photos/600/600?random=N` (N = 1 a 6)
- Exportar ambos

---

### MÓDULO 1 — Stores Pinia (auth + feed)

**Por que antes do roteamento:** os guards de rota dependem do store de auth. O feed depende do store de feed. Ambos precisam existir antes de qualquer view.

**`src/stores/auth.js`:**
- Estado: `user` (objeto ou null), `token` (string mock ou null), `localAccounts` (array de contas cadastradas)
- Inicialização (`init()`): hidrata `user` e `token` do `localStorage`. Se nenhuma conta existir, insere os `MOCK_USERS` como contas seed
- `login(username, password)`: busca em `localAccounts`, gera token mock (`"mock-token-" + Date.now()`), salva no localStorage, atualiza `user` e `token`
- `register(name, username, email, password)`: valida unicidade de username, cria conta, faz login automático
- `logout()`: limpa `user` e `token` do estado e do localStorage
- `updateProfile(data)`: atualiza `user` no estado e no localStorage. Se `data.avatar` for base64, salva junto
- `isAuthenticated`: computed que retorna `!!token`
- `currentUserId`: computed que retorna `user?.id`

**`src/stores/feed.js`:**
- Estado: `posts` (array), `page` (number), `hasMore` (boolean)
- Inicialização (`init()`): carrega posts do localStorage; se vazio, usa `MOCK_POSTS` como seed
- `getPaginatedFeed(page, perPage = 5)`: retorna slice do array de posts, ordenado por `createdAt` decrescente
- `getPostById(id)`: retorna post por id
- `getPostsByUser(userId)`: filtra posts por autor
- `createPost(imageBase64, caption, location)`: cria objeto de post com `id` único (`Date.now()`), adiciona ao início do array, persiste no localStorage
- `deletePost(id)`: remove do array e persiste
- `toggleLike(postId, userId)`: adiciona ou remove userId do array `likedBy` do post; atualiza `likesCount`; persiste
- `addComment(postId, text, userId)`: cria objeto `{ id, userId, username, text, createdAt }`, adiciona ao array de comentários do post, persiste
- `persistPosts()`: helper privado que salva posts no localStorage
- Expor getter `isLikedByUser(postId, userId)` para verificar curtida atual

---

### MÓDULO 2 — Roteamento e App Base

**`src/router/index.js`:**
- Todas as views com lazy loading: `component: () => import('@/views/...')`
- Rotas **públicas**: `/login`, `/register`
- Rotas **protegidas**: `/feed`, `/create`, `/post/:id`, `/profile/:username`, `/profile/:username/followers`, `/profile/:username/following`, `/profile/edit`
- `beforeEach`: importa `useAuthStore`. Se rota protegida e `!authStore.isAuthenticated` → redireciona `/login`. Se rota pública e `authStore.isAuthenticated` → redireciona `/feed`
- Rota catch-all `/:pathMatch(.*)*` → redireciona `/feed`

**`src/App.vue`:**
- Chama `authStore.init()` e `feedStore.init()` no `onMounted` para hidratar dados do localStorage
- `<RouterView />` com transição `fade` (opacity 0→1, 200ms)

**`src/utils/date.js`:** função `timeAgo(dateString)` → "agora", "há X minutos", "há X horas", "há X dias"

**`src/utils/format.js`:** função `formatCount(n)` → "1,2k", "4,5M" para números grandes

---

### MÓDULO 3 — Componentes UI Base e Layout

**`src/components/ui/Avatar.vue`:**
- Props: `src` (string), `size` (sm=32px / md=44px / lg=80px), `alt` (string)
- Se `src` for vazio ou falhar (`@error`): exibe círculo colorido com a inicial do `alt`

**`src/components/ui/Spinner.vue`:**
- Animação CSS `spin`, tamanhos sm/md/lg via prop, cor via `currentColor`

**`src/components/ui/ConfirmModal.vue`:**
- Props: `title`, `message`, `confirmLabel` (default "Confirmar"), `cancelLabel` (default "Cancelar")
- Emite: `confirm`, `cancel`
- Overlay fullscreen com `position: fixed`, `z-index: 1000`
- Usado no delete de post

**`src/layouts/MainLayout.vue`:**
- `<slot name="header" />` — header opcional customizável por view
- `<slot />` — conteúdo principal
- `<Navbar />` fixo
- Padding-bottom `60px` no mobile para não sobrepor a navbar

**`src/components/Navbar.vue`:**
- **Mobile** (`< 768px`): `position: fixed`, `bottom: 0`, largura 100%, 3 ícones SVG: Home (`/feed`), Criar (`/create`), Perfil (`/profile/:username` do usuário logado)
- **Desktop** (`≥ 768px`): sidebar `position: fixed`, `left: 0`, altura 100vh, ícone + label
- Ícone de Perfil usa `<Avatar size="sm" />` com foto do usuário logado (vindo de `authStore.user`)
- Link ativo visualmente diferenciado (ícone preenchido vs outline, usando `router-link-active`)
- SVG inline para todos os ícones (sem dependência externa)

---

### MÓDULO 4 — Autenticação (Login e Cadastro)

**`src/views/LoginView.vue`:**
- Formulário: username + senha
- Validação client-side: campos obrigatórios antes de qualquer ação
- Submit: chama `authStore.login(username, password)`
- Sucesso: redireciona para `/feed` com `router.replace`
- Erro: mensagem abaixo do formulário ("Usuário ou senha incorretos")
- Estado de loading no botão durante o processo
- Link para `/register`
- Design: centralizado, nome "InstaClone" estilizado com gradiente, visual limpo

**`src/views/RegisterView.vue`:**
- Formulário: nome completo, username (só `[a-zA-Z0-9_]`), email, senha (mín. 6 chars), confirmar senha
- Validação: todos os campos, username único (verifica em `authStore.localAccounts`), senhas iguais
- Submit: chama `authStore.register(...)` → faz login automático → redireciona `/feed`
- Mesmo padrão visual do Login

---

### MÓDULO 5 — Feed

**`src/composables/useInfiniteScroll.js`:**
- Parâmetros: `fetchFn` (função que recebe página, retorna `{ items, hasMore }`), `options`
- Retorna: `items` (array acumulado), `isLoading`, `hasMore`, `error`, `sentinel` (ref para elemento DOM), `reset()`
- Usa `IntersectionObserver` no elemento sentinela para chamar `loadMore` automaticamente
- `onUnmounted`: desconecta o observer

**`src/components/PostCard.vue`** — componente completo:
- **Header**: `<Avatar size="md" />` + username clicável → `/profile/:username` + nome
- **Imagem**: `loading="lazy"`, `aspect-ratio: 1/1`, `object-fit: cover`
- **Localização** (se existir): ícone de pin + texto abaixo do header
- **Actions**: botão curtir (❤️ vazio/preenchido com animação de escala via CSS), contagem de likes com `formatCount()`
- **Legenda**: `<strong>username</strong> texto` — trunca com "... mais" se > 120 chars
- **Preview de comentários**: até 2 comentários (`username: texto`)
- **Campo de comentário inline**: input simples, envio com Enter, chama `feedStore.addComment()`
- **Data**: `timeAgo(post.createdAt)` em cinza
- **Curtir com atualização otimista**: chama `feedStore.toggleLike(postId, userId)` — store atualiza imediatamente e persiste

**`src/views/FeedView.vue`:**
- Busca posts via `feedStore.getPaginatedFeed(page)` com paginação simples (botão "Carregar mais" ou scroll infinito com `useInfiniteScroll`)
- Lista de `<PostCard />`, cada um recebe o objeto `post` completo como prop
- Skeleton loader (3 cards estilizados) durante o carregamento inicial
- Mensagem vazia: "Ainda não há posts. Crie o primeiro! 📸" com link para `/create`

---

### MÓDULO 6 — Criar Post

**`src/views/CreatePostView.vue`** — fluxo em 3 estados internos:

**Estado 1 — Seleção:**
- Área de drag-and-drop estilizada com ícone e texto "Arraste ou clique para selecionar"
- Input `type="file"` `accept="image/*"` oculto, acionado pelo clique na área
- Ao selecionar arquivo: converte para base64 com `FileReader`, avança para Estado 2

**Estado 2 — Edição:**
- Preview quadrado (`aspect-ratio: 1/1`) da imagem selecionada
- Textarea para legenda (contador `X / 2200` chars)
- Campo de localização (texto livre, opcional)
- Botão "Voltar" (limpa seleção, volta ao Estado 1)
- Botão "Compartilhar"

**Estado 3 — Publicando:**
- Chama `feedStore.createPost(imageBase64, caption, location)`
- Exibe `<Spinner />` durante o processo
- Sucesso: ícone ✅ + "Post publicado!" + redireciona para `/feed` após 1.5s
- Erro: mensagem + botão "Tentar novamente" (preserva dados)

---

### MÓDULO 7 — Perfil de Usuário

**`src/composables/useProfile.js`:**
- Recebe `username` reativo (da rota via `useRoute()`)
- Busca usuário em `authStore.localAccounts` pelo username
- Expõe: `profile`, `posts` (via `feedStore.getPostsByUser`), `isLoading`, `isOwnProfile` (compara com `authStore.currentUserId`), `isFollowing`, `toggleFollow()`
- `toggleFollow()`: atualiza `followersCount` no store e persiste localmente
- `watch(() => route.params.username, fetchAll, { immediate: true })` — recarrega ao trocar de perfil

**`src/views/ProfileView.vue`:**
- Header: `<Avatar size="lg" />` + nome + username + bio + contadores clicáveis (posts / seguidores → `/followers` / seguindo → `/following`)
- Botão "Seguir / Seguindo" com atualização imediata se perfil alheio
- Botão "Editar perfil" → navega para `/profile/edit` se perfil próprio
- Grid 3 colunas (`grid-template-columns: repeat(3, 1fr)`) de miniaturas dos posts (`aspect-ratio: 1/1`, `object-fit: cover`)
- Clicar em miniatura → navega para `/post/:id`
- Mensagem "Nenhum post ainda" se grid vazio

**`src/views/EditProfileView.vue`:**
- Avatar clicável → abre `input[type=file]` oculto → preview imediato (base64)
- Campos: nome, username, bio (textarea, max 150 chars)
- Submit: chama `authStore.updateProfile(data)` → redireciona para `/profile/:username` atualizado
- Botão cancelar retorna ao perfil sem salvar

**`src/views/FollowersView.vue` / `src/views/FollowingView.vue`:**
- Recebe `:username` da rota
- Busca lista de seguidores/seguindo do usuário
- Lista: `<Avatar />` + nome + username + link para o perfil

---

### MÓDULO 8 — Detalhes do Post

**`src/components/CommentInput.vue`:**
- Input: placeholder "Adicione um comentário..."
- Botão "Publicar" desabilitado quando input vazio
- Enter (sem Shift) também envia
- Emite `comment-added` com o texto após envio e limpa o campo

**`src/components/CommentList.vue`:**
- Props: `postId`, `initialCount` (quantos mostrar inicialmente, default 3)
- Busca `post.comments` do `feedStore.getPostById(postId)`
- Exibe os N primeiros comentários
- Botão "Ver mais comentários" acumula mais (`+ 5` por clique), some quando não há mais
- Item: `<Avatar size="sm" />` + `<strong>username</strong>` + texto + `timeAgo()`

**`src/views/PostDetailView.vue`** (rota `/post/:id`):
- Carrega post via `feedStore.getPostById(id)` — se não encontrado, redireciona para `/feed`
- **Layout mobile** (vertical): imagem → header do autor → localização → actions → legenda → comentários → input
- **Layout desktop** (lado a lado): imagem à esquerda | painel direito (header + localização + legenda + `<CommentList />` com scroll + `<CommentInput />` fixo no bottom do painel)
- Header: `<Avatar />` + username (link pro perfil)
- Localização: ícone de pin + texto (se existir)
- Actions: botão curtir (mesmo padrão do PostCard, via `feedStore.toggleLike`) + contagem com `formatCount()`
- Legenda completa (sem truncamento)
- `<CommentInput />` no final: ao emitir `comment-added`, chama `feedStore.addComment(postId, text, userId)`
- Botão "Deletar post" (visível só se `isOwnProfile`):
  - Abre `<ConfirmModal />` com texto "Tem certeza que deseja excluir este post?"
  - Confirmar: chama `feedStore.deletePost(id)` → redireciona para `/profile/:username`
  - Cancelar: fecha modal sem ação

---

### MÓDULO FINAL — Revisão, Polimento e Persistência

Após todos os módulos prontos, fazer um sweep completo:

**Persistência:**
- Verificar que todas as ações (curtir, comentar, criar post, deletar, editar perfil, seguir) persistem corretamente ao recarregar a página (F5)
- Verificar que o `App.vue` está chamando `authStore.init()` e `feedStore.init()` no `onMounted`
- Verificar que usuários criados via cadastro persistem entre sessões

**Fluxo completo a testar:**
1. Abrir o app sem sessão → deve cair em `/login`
2. Fazer login com `usuario_teste` / `senha123` → redireciona para `/feed`
3. Ver posts no feed, curtir um, comentar
4. Recarregar a página → curtida e comentário ainda aparecem
5. Criar um post novo → aparece no topo do feed
6. Acessar o post pelo detalhe → comentar → deletar
7. Acessar o próprio perfil → editar → ver mudança refletida na Navbar
8. Acessar perfil de outro usuário → seguir → ver contador atualizar
9. Fazer logout → tentar acessar `/feed` → redireciona para `/login`

**Visual e acessibilidade:**
- Responsividade em 375px (iPhone SE) e 1440px (desktop)
- Todos os botões de ícone com `aria-label`
- Imagens com `alt` descritivo
- Estados de loading e erro cobertos em todas as views

---

## 🚦 INICIE AGORA

Antes de escrever qualquer código, faça estas perguntas:
1. O projeto já tem alguma estrutura de pastas ou é do zero?
2. O Pinia já está instalado (`npm install pinia`)?
3. Quer que eu crie o `src/data/mockData.js` com usuários e imagens realistas logo no Módulo 0?

Após minhas respostas, comece pelo **MÓDULO 0**.
