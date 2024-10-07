# Frontend
Para o Frontend nós utilizaremos o React com Vite, crie um projeto normalmente utilizando o Vite e o Javascript

## Primeiro Passo:
- Configuração do proxy para redirecionar as requisições para o Gateway  `vite.config.js`
```Javascript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()], // Plugin para suportar React
  server: {

    // Configuração do proxy para redirecionar as requisições para o Gateway

    proxy: {
      '/oauth2/authorization/azure': {
        target: 'http://localhost:8000', // URL de destino para redirecionamento
        changeOrigin: true, // Altera a origem da requisição para o URL de destino
        secure: false,
        rewrite: (path) => path.replace(/^\/gateway/, ''), // Reescreve o caminho removendo '/gateway'
      },

      // Proxy para a rota de informações do usuário

      '/userinfo': {
        target: 'http://localhost:8000', 
        changeOrigin: true, 
        secure: false,
        rewrite: (path) => path.replace(/^\/gateway/, ''),
      },

      // Proxy para a rota de recursos protegidos

      '/resource': {
        target: 'http://localhost:8000',
        changeOrigin: true,
        secure: false,
        rewrite: (path) => path.replace(/^\/gateway/, ''),
      },

      // Proxy para a rota de callback

      '/callback': {
        target: 'http://localhost:8000/login/oauth2/code/azure',
        changeOrigin: true,
        secure: false,
        rewrite: (path) => path.replace(/^\/callback/, ''),
      },
      '/logout': {
        target: URL,
        changeOrigin: true,
        secure: false,
        rewrite: (path) => path.replace(/^\/gateway/, ''),
    }
  }
});

```
Aqui nós configuramos o proxy do servidor para que ele redirecione as rotas para o nosso Gateway!

## Segundo Passo
- Criação das rotas
`Routes.jsx`
```Javascript
import React from 'react';
import { Routes, Route } from 'react-router-dom';
import ResourceFetcher from './ResourceFetcher';
import Profile from './Profile';

const AppRoutes = () => (
  <Routes>
    <Route path="/resource" element={<ResourceFetcher />} />
    <Route path="/profile" element={<Profile />} />
  </Routes>
);

export default AppRoutes;
```
## Terceiro Passo
- `App.jsx`
```Javascript
import React from 'react';
import { BrowserRouter as Router, Link } from 'react-router-dom';
import AppRoutes from './components/Routes';

const App = () => {
  const handleLogin = () => {
    window.location.href = 'http://localhost:3000/oauth2/authorization/azure'; // Redireciona para o Login (SSO)
  };

  return (
    <Router>
      <div>
        <nav>
          <button onClick={handleLogin}>Login</button>
          <Link to="/resource">Go to Resource</Link>
          <Link to="/profile">Go to Profile</Link>
        </nav>
        <AppRoutes />
      </div>
    </Router>
  );
};

export default App;

```
## Quarto Passo
- `Profile.jsx`
  
```Javascript
import React, { useEffect, useState } from "react";
import axios from "axios";

const API_BASE_URL = "http://localhost:3000"; // Base URL para as requisições

const Profile = () => {
  const [userInfo, setUserInfo] = useState(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUserInfo = async () => {
      try {
        const response = await axios.get(`${API_BASE_URL}/userinfo`); 
        const userData = response.data;

        setUserInfo(userData);
        localStorage.setItem("id_token", userData.id_token); // Armazena o Token no LocalStorage
        
      } catch (error) {
        console.error("Erro ao buscar informações do usuário:", error);
        setError("Erro ao buscar informações do usuário.");
      }
    };

    fetchUserInfo();
  }, []);

// Apenas renderiza as informações

  return (
    <div>
      {error && <p>{error}</p>}
      {userInfo ? (
        <div>
          <h1>Profile</h1>
          <span style={{ fontWeight: "bold" }}> Tokens: </span>
          <ul>
            <li>
              <span style={{ fontWeight: "bold" }}>id_token</span>:{" "}
              <span> {userInfo.id_token} </span>
            </li>
            <li>
              <span style={{ fontWeight: "bold" }}>access_token</span>:{" "}
              <span> {userInfo.access_token} </span>
            </li>
          </ul>
          <div>
            <span style={{ fontWeight: "bold" }}> User Attributes: </span>
            <ul>
              {userInfo &&
                userInfo.user_attributes &&
                Object.entries(userInfo.user_attributes).map(([key, value]) => (
                  <li key={key}>
                    <span style={{ fontWeight: "bold" }}> {key} </span>:{" "}
                    <span> {value} </span>
                  </li>
                ))}
            </ul>
          </div>
        </div>
      ) : (
        <p>Loading...</p>
      )}
    </div>
  );
};

export default Profile;
```


## Quinto Passo
- Criar Componente para fazer a requisição ao recurso protegido de nosso backend `ResourceFetcher.jsx`
```Javascript
import React, { useEffect, useState } from "react";
import axios from "axios";

const API_BASE_URL = "http://localhost:3000"; // Base URL para as requisições

const ResourceFetcher = () => {
  const [data, setData] = useState(null);

  useEffect(() => {
    const fetchResource = async () => {
      const token = localStorage.getItem("id_token"); // Pegar o token anteriormente armazenado no localStorage

      if (token) {
        try {
          const response = await axios.get(`${API_BASE_URL}/resource`, { // Fazer a requisição passando o token no header
            headers: {
              Authorization: `Bearer ${token}`,
            },
          });

          setData(response.data);
          
        } catch (error) {
          console.error(
            "Erro ao acessar recurso protegido:",
            error.response ? error.response.data : error.message
          );
        }
      } else {
        console.error("Token não encontrado no localStorage.");
      }
    };

    fetchResource();
  }, []);

  // Exibir Recurso Protegido

  return (
    <div>
      <h1>Recurso Protegido</h1>
      <div>
        <ul>
          <li>
            <span> {data} </span>
          </li>
        </ul>
      </div>
    </div>
  );
};

export default ResourceFetcher;

```

### Caso tenha alguma duvida sobre qualquer um dos processos, entre em contato conosco!
#### Caio Farias & Amber Forte
