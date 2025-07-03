# **Integração Frontend + Backend**  
### (React + Spring Boot REST API)

<div align="center">
  <img src="https://img.icons8.com/color/96/000000/react-native.png" alt="React" title="React"/> 
  <img src="https://img.icons8.com/color/96/000000/spring-logo.png" alt="Spring Boot" title="Spring Boot"/>
  <img src="https://img.icons8.com/fluency/96/000000/api.png" alt="API" title="API"/>
  <img src="https://img.icons8.com/color/96/000000/docker.png" alt="Docker" title="Docker"/>
  <img src="https://img.icons8.com/external-tal-revivo-color-tal-revivo/96/external-postman-is-the-only-complete-api-development-environment-logo-color-tal-revivo.png" alt="Postman" title="Postman"/>
</div>

<br>

Neste tutorial, será explicar como integrar um projeto <img src="https://img.icons8.com/color/96/000000/react-native.png" alt="React" title="React" width="20" />  React (frontend) com <img src="https://img.icons8.com/color/96/000000/spring-logo.png" alt="Spring Boot" title="Spring Boot" width="20"/> Spring Boot (backend), utilizando:
- <img src="https://img.icons8.com/external-tal-revivo-color-tal-revivo/96/external-postman-is-the-only-complete-api-development-environment-logo-color-tal-revivo.png" alt="Postman" title="Postman" width="20" /> Postman (ferramenta para testar APIs, enviar requisições HTTP e verificar respostas)
- <img src="https://img.icons8.com/color/96/000000/docker.png" alt="Docker" title="Docker" width="20" /> Docker (plataforma para criar, executar e gerenciar aplicações em containers isolados)
- <img src="https://img.icons8.com/fluency/96/000000/api.png" alt="API" title="API" width="20" /> API REST (conjunto de regras que permitem a comunicação entre frontend e backend, seguindo o padrão REST)

---

## **🔧 Arquitetura do Sistema**  

O sistema segue uma arquitetura **cliente-servidor**, onde:  
- <img src="https://img.icons8.com/color/96/000000/react-native.png" alt="React" title="React" width="20" /> **Frontend (React):**  
  - Consome a API via **Axios** (biblioteca para requisições HTTP).  
  - Armazena dados temporários no **LocalStorage** (para sessão do usuário, tokens JWT, etc.).  
  - Renderiza a interface com base nas respostas da API.  

- <img src="https://img.icons8.com/color/96/000000/spring-logo.png" alt="Spring Boot" title="Spring Boot" width="20"/> **Backend (Spring Boot):**  
  - Expõe endpoints REST (**GET, POST, PUT, DELETE**).  
  - Processa requisições, aplica regras de negócio e se comunica com o banco de dados.  
  - Retorna respostas em **JSON**, que são consumidas pelo frontend.  

- **Ferramentas de Apoio:**  
  - **Postman:** Testa endpoints da API antes da integração com o frontend.  
  - **Docker:** Containeriza a aplicação para facilitar deploy e ambiente de desenvolvimento.  

```mermaid
graph LR
    A[Frontend React] -->|Axios| B[Spring Boot API]
    B -->|JSON| A
    A --> C[(LocalStorage)]
    B --> D[(Database)]
```

---

## **📂 Estrutura de Pastas (Tree View)**  

```mermaid
graph TD
    A[Projeto] --> B[frontend]
    A --> C[backend]
    B --> D[public]
    B --> E[src]
    E --> F[components]
    E --> G[services]
    E --> H[pages]
    E --> I[contexts]
    C --> J[src/main/java]
    J --> K[com/example/projeto]
    K --> L[config]
    K --> M[controller]
    K --> N[model]
    K --> O[repository]
    K --> P[security]
    K --> Q[service]
```

```bash
PROJETO/
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── assets/            # Imagens/fontes
│   │   ├── components/        # Componentes reutilizáveis
│   │   ├── contexts/          # Contextos React (AuthContext)
│   │   ├── hooks/             # Custom hooks
│   │   ├── pages/             # Telas da aplicação
│   │   ├── services/          # Conexão com API (axios)
│   │   ├── styles/            # CSS global
│   │   └── App.js             # Roteamento principal
│   └── Dockerfile
└── backend/
    ├── src/main/java/
    │   └── com/seuprojeto/
    │       ├── config/        # SecurityConfig, Swagger
    │       ├── controller/    # RestControllers
    │       ├── dto/           # Objetos de transferência
    │       ├── exception/     # Handlers de erro
    │       ├── model/         # Entidades JPA
    │       ├── repository/    # JPA Repositories
    │       ├── security/      # JWT filters
    │       └── service/       # Lógica de negócio
    ├── src/main/resources/
    │   ├── application.yml    # Configurações
    │   └── data.sql          # Dados iniciais
    └── Dockerfile
```

---

## **🚦 Fluxo de Autenticação**  

### **1. Login (POST)**
<div align="center">
  <img src="https://img.icons8.com/fluency/48/000000/login-rounded.png" alt="Login"/>
</div>

**Frontend Code** (`authService.js`):  
```jsx
// 🚀 Método de Login
async function login(email, senha) {
  const response = await axios.post(
    'http://localhost:4201/api/auth/login',
    { email, senha } // 📦 Body JSON
  );
  return response.data; // 🔑 Token + User Data
}
```

**Request/Response Example:**  
```http
POST /api/auth/login HTTP/1.1
Content-Type: application/json

{
  "email": "user@example.com",
  "senha": "123456"
}
```
```json
{
  "token": "eyJhbGciOiJ...",  // 🔐 JWT Token
  "user": { "id": 1, "nome": "Fulano" }
}
```

---

### **2. Token JWT Explained**  
<div align="center">
  <img src="https://img.icons8.com/color/48/000000/jwt.png" alt="JWT"/>
</div>

**Estrutura do Token:**  
```json
{
  "sub": "user@email.com",  // 📧 Email
  "id": 1,                  // 🆔 User ID
  "userlevel": true,        // ✅ Email verificado?
  "admlevel": false,        // 👑 Admin?
  "ativo": true             // 🟢 Conta ativa?
}
```

**Validações Importantes:**  
- `userlevel: false` → Mostrar alerta: *"E-mail não verificado!"* ❌  
- `ativo: false` → Bloquear acesso: *"Conta desativada!"* ⛔  

---

### **3. Buscar Dados do Usuário (GET)**  
<div align="center">
  <img src="https://img.icons8.com/fluency/48/000000/get-rejected.png" alt="GET Request"/>
</div>

**Requisição com Token:**  
```jsx
const getUser = async (id) => {
  const response = await axios.get(
    `http://localhost:4201/api/usuario/${id}`,
    { headers: { 
        Authorization: `Bearer ${token}`  // 🔑 Token no Header
      } 
    }
  );
  return response.data;
};
```

**Exemplo de Resposta:**  
```json
{
  "id": 1,
  "nome": "Fulano",
  "email": "fulano@email.com",
  "admlevel": false,
  "_links": { /* ... */ }
}
```

---

## **📦 Armazenamento no Frontend**  
<div align="center">
  <img src="https://img.icons8.com/fluency/48/000000/save.png" alt="Save Data"/>
</div>

**Salvando em LocalStorage:**  
```jsx
localStorage.setItem('auth', JSON.stringify({
  token: "eyJhbGci...",  // 🔐
  user: { id: 1, nome: "Fulano" }  // 👤
}));
```

---

## **🔧 Ferramentas Utilizadas**  

| Tecnologia       | Ícone | Descrição          |
|------------------|-------|--------------------|
| **React**        | <img src="https://img.icons8.com/color/48/000000/react-native.png" width="20"/> | Biblioteca Frontend |
| **Axios**        | <img src="https://img.icons8.com/ios/50/000000/axios.png" width="20"/> | Cliente HTTP |
| **Spring Boot**  | <img src="https://img.icons8.com/color/48/000000/spring-logo.png" width="20"/> | Backend Java |
| **JWT**          | <img src="https://img.icons8.com/color/48/000000/jwt.png" width="20"/> | Autenticação |

---

## **🎨 Diagrama Completo**  
```mermaid
sequenceDiagram
    participant Frontend
    participant Backend
    participant Database

    Frontend->>Backend: POST /login (email, senha)
    Backend->>Database: Valida credenciais
    Backend-->>Frontend: Retorna Token JWT
    Frontend->>Backend: GET /usuario (com Token)
    Backend-->>Frontend: Dados do Usuário
    Frontend->>LocalStorage: Salva Token + Dados
```

---
