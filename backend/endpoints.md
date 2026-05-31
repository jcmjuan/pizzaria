## Endpoints da API - Sistema de Pizzaria

### Convenções Gerais

- **Base URL padrão**: `http://localhost:3333`
- Todas as rotas (exceto criação de usuário e login) exigem **autenticação via JWT**.
- Header de autenticação:

```http
Authorization: Bearer <token>
```

---

### 1. Usuários

#### 1.1 POST `/users` – Criar usuário

- **Descrição**: Cria um novo usuário no sistema.
- **Middlewares**:
  - `validateSchema(createUserSchema)`
- **Body (JSON)**:

```json
{
  "name": "João Silva",
  "email": "joao@example.com",
  "password": "senha123"
}
```

---

#### 1.2 POST `/session` – Login

- **Descrição**: Autentica um usuário e retorna um token JWT.
- **Middlewares**:
  - `validateSchema(authUserSchema)`
- **Body (JSON)**:

```json
{
  "email": "joao@example.com",
  "password": "senha123"
}
```

---

#### 1.3 GET `/me` – Detalhes do usuário autenticado

- **Descrição**: Retorna os dados do usuário autenticado.
- **Middlewares**:
  - `isAuthenticated`

---

### 2. Categorias

#### 2.1 POST `/category` – Criar categoria

- **Descrição**: Cria uma nova categoria de produtos.
- **Middlewares**:
  - `isAuthenticated`
  - `isAdmin`
  - `validateSchema(createCategorySchema)`
- **Body (JSON)**:

```json
{
  "name": "Pizzas Salgadas"
}
```

---

#### 2.2 GET `/category` – Listar categorias

- **Descrição**: Lista todas as categorias cadastradas.
- **Middlewares**:
  - `isAuthenticated`

---

### 3. Produtos

#### 3.1 POST `/product` – Criar produto

- **Descrição**: Cria um novo produto vinculado a uma categoria.
- **Middlewares**:
  - `isAuthenticated`
  - `isAdmin`
  - `upload.single("file")`
  - `validateSchema(createProductSchema)`
- **Headers**:
  - `Content-Type: multipart/form-data`
- **Campos (form-data)**:
  - `name` (texto)
  - `price` (texto numérico)
  - `description` (texto)
  - `category_id` (texto)
  - `file` (arquivo de imagem)

---

#### 3.2 GET `/products` – Listar produtos

- **Descrição**: Lista produtos, com filtro opcional por `disabled`.
- **Middlewares**:
  - `isAuthenticated`
  - `validateSchema(listProductSchema)`
- **Query params**:
  - `disabled` (opcional): `"true"` ou `"false"`

---

#### 3.3 DELETE `/product` – Deletar produto

- **Descrição**: Remove um produto (normalmente por ID via query ou body, conforme service interno).
- **Middlewares**:
  - `isAuthenticated`
  - `isAdmin`

---

#### 3.4 GET `/category/product` – Listar produtos por categoria

- **Descrição**: Lista produtos filtrando por categoria.
- **Middlewares**:
  - `isAuthenticated`
  - `validateSchema(listProductByCategorySchema)`
- **Query params** (conforme schema):
  - `category_id`: ID da categoria

---

### 4. Pedidos (Orders)

#### 4.1 POST `/order` – Criar pedido

- **Descrição**: Cria um novo pedido.
- **Middlewares**:
  - `isAuthenticated`
  - `validateSchema(createOrderSchema)`
- **Body (JSON)**:

```json
{
  "table": 10,
  "name": "Mesa 10 - João"
}
```

---

#### 4.2 GET `/orders` – Listar pedidos

- **Descrição**: Lista pedidos (pode considerar rascunhos ou não, conforme service).
- **Middlewares**:
  - `isAuthenticated`
- **Query params** (conforme service `ListOrdersService`):
  - `draft` (opcional): `"true"` ou `"false"`

---

#### 4.3 GET `/order/detail` – Detalhar pedido

- **Descrição**: Retorna os detalhes completos de um pedido (itens + produtos).
- **Middlewares**:
  - `isAuthenticated`
  - `validateSchema(detailOrderSchema)`
- **Query params**:
  - `order_id`: ID do pedido

---

#### 4.4 POST `/order/add` – Adicionar item ao pedido

- **Descrição**: Adiciona um novo item a um pedido existente.
- **Middlewares**:
  - `isAuthenticated`
  - `validateSchema(addItemSchema)`
- **Body (JSON)**:

```json
{
  "order_id": "uuid-do-pedido",
  "product_id": "uuid-do-produto",
  "amount": 2
}
```

---

#### 4.5 DELETE `/order/remove` – Remover item do pedido

- **Descrição**: Remove um item de um pedido.
- **Middlewares**:
  - `isAuthenticated`
  - `validateSchema(removeItemSchema)`
- **Query params**:
  - `item_id`: ID do item a ser removido

---

#### 4.6 DELETE `/order` – Deletar pedido

- **Descrição**: Remove um pedido completo.
- **Middlewares**:
  - `isAuthenticated`
  - `validateSchema(deleteOrderSchema)`
- **Query params**:
  - `order_id`: ID do pedido

---

#### 4.7 PUT `/order/send` – Enviar pedido

- **Descrição**: Altera o estado do pedido, enviando-o (por exemplo, da situação de rascunho para produção/cozinha).
- **Middlewares**:
  - `isAuthenticated`
  - `validateSchema(sendOrderSchema)`
- **Body (JSON)**:

```json
{
  "order_id": "uuid-do-pedido",
  "name": "Mesa 10 - João"
}
```

---

#### 4.8 PUT `/order/finish` – Finalizar pedido

- **Descrição**: Marca um pedido como finalizado/concluído.
- **Middlewares**:
  - `isAuthenticated`
  - `validateSchema(finishOrderSchema)`
- **Body (JSON)**:

```json
{
  "order_id": "uuid-do-pedido"
}
```

