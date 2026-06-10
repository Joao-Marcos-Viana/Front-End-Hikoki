# Front-End Hikoki — VDM TSEA

Interface web do sistema de Visualização de Desenhos de Máquinas (VDM). Desenvolvida em HTML, CSS e JavaScript puro (sem frameworks), voltada para uso industrial em tablets e computadores na rede interna da fábrica.

---

## Visão Geral

O VDM TSEA é uma SPA (Single Page Application) contida em um único arquivo `index.html`. Ela se comunica com o back-end Spring Boot via chamadas `fetch` para a API REST e apresenta desenhos técnicos CAD com controle de versões, perfis de acesso e upload de novos arquivos.

---

## Pré-requisitos

- Navegador moderno (Chrome, Edge, Firefox)
- Back-end Hikoki rodando em **http://localhost:8085**
- Nenhuma dependência de build — o projeto roda diretamente no navegador

---

## Como Executar

1. Extraia a pasta `Front-End-Hikoki`
2. Abra o arquivo `static/index.html` diretamente no navegador

   **Ou**, se preferir usar um servidor local (recomendado para evitar restrições de CORS):

   ```bash
   # Com Python (já instalado na maioria dos sistemas)
   cd Front-End-Hikoki/static
   python -m http.server 5500
   ```

   Acesse: **http://localhost:5500**

3. Certifique-se de que o back-end esteja rodando em `http://localhost:8085` antes de usar o sistema.

---

## Estrutura do Projeto

```
Front-End-Hikoki/
├── static/
│   ├── index.html    ← aplicação completa (HTML + CSS + JS em um único arquivo)
│   └── favicon.png   ← ícone da aplicação
└── README.md
```

---

## Funcionalidades

### Tela de Login

- Seleção de perfil: **Funcionário** ou **Supervisor**
- Autenticação via API (`POST /api/login`)
- Campos: ID / Matrícula e Senha

**Credenciais padrão:**

| Perfil | Matrícula | Senha | Acesso |
|---|---|---|---|
| Funcionário | `00123` | `1234` | Visualização — setor Montagem |
| Supervisor | `admin` | `admin1234` | Todos os setores + gerenciamento |

---

### Dashboard

- Sidebar com lista de setores carregada dinamicamente da API (`GET /api/setores`)
- Funcionários veem apenas o setor **Montagem**
- Supervisores têm acesso a todos os setores: Usinagem, Estamparia, Solda, Montagem, Controle de Qualidade
- Cards de desenhos com nome, setor, revisão atual e botão de exclusão (apenas supervisores)
- Campo de busca para filtrar desenhos por nome em tempo real
- Botão **+ Nova Versão** visível apenas para supervisores

---

### Visualizador de Desenhos

- Abre ao clicar em um card de desenho
- Exibe a revisão mais recente por padrão
- Suporte a zoom (scroll do mouse / botões)
- Arrastar para navegar na imagem
- Painel de **Histórico de Versões** com todas as revisões disponíveis
- Botão de download do arquivo
- Suporte a múltiplos formatos:

| Formato | Visualização | Upload | Download |
|---|---|---|---|
| PNG | ✅ direto | ✅ | ✅ |
| JPG | ✅ direto | ✅ | ✅ |
| SVG | ✅ direto | ✅ | ✅ |
| PDF | ✅ direto | ✅ | ✅ |
| DWG | ⚠️ só download | ✅ | ✅ |
| DXF | ⚠️ só download | ✅ | ✅ |

> Para visualizar DWG/DXF diretamente, utilize AutoCAD ou LibreCAD (gratuito).

---

### Upload de Nova Versão (apenas Supervisor)

Modal de upload acessível pelo botão **+ Nova Versão** no dashboard ou no visualizador.

Campos do formulário:
- Arquivo (drag & drop ou clique para selecionar)
- Nome do desenho
- Setor
- Número da revisão
- Descrição (opcional)
- Autor

O upload é enviado para `POST /api/desenhos/upload` como `multipart/form-data`.

---

### Exclusão de Desenho (apenas Supervisor)

- Botão **✕ excluir** disponível nos cards do dashboard
- Remove o desenho e **todas as suas versões** permanentemente (incluindo arquivos físicos no servidor)
- Solicita confirmação antes de executar

---

## Integração com a API

Todos os endpoints consumidos apontam para `http://localhost:8085`. Para alterar o host/porta da API, edite as chamadas `fetch` no arquivo `static/index.html`.

| Ação | Método | Endpoint |
|---|---|---|
| Login | POST | `/api/login` |
| Listar setores | GET | `/api/setores` |
| Listar desenhos | GET | `/api/desenhos?setor={setor}` |
| Abrir arquivo | GET | `/api/arquivo/{filename}` |
| Upload de versão | POST | `/api/desenhos/upload` |
| Excluir desenho | DELETE | `/api/desenhos/{id}` |

---

## Acessar de Outros Dispositivos (Tablets / Celulares)

Para acessar o sistema de outros dispositivos na mesma rede Wi-Fi:

1. No PC que roda o servidor, abra o CMD e execute `ipconfig`
2. Anote o endereço **IPv4** (ex: `192.168.1.105`)
3. Nos outros dispositivos, acesse `http://192.168.1.105:5500` no navegador

> O PC servidor precisa estar ligado e com o servidor local ativo.

---

## Solução de Problemas

**Login não funciona / "Matrícula ou senha incorretos"**
→ Verifique se o back-end está rodando em `http://localhost:8085`.

**Desenhos não aparecem após o login**
→ Confirme se o setor selecionado possui desenhos cadastrados no banco.
→ Clique no botão ↻ para recarregar a lista.

**Arquivo não abre no visualizador**
→ Verifique se o arquivo existe na pasta `uploads/` do back-end.
→ Formatos DWG e DXF não possuem visualização nativa — use o botão de download.

**CORS bloqueado no navegador**
→ Sirva o front-end via servidor local (ex: `python -m http.server 5500`) em vez de abrir o arquivo diretamente pelo sistema de arquivos.

---

## Personalização

Para alterar a interface, edite o arquivo `static/index.html` em qualquer editor de texto e recarregue o navegador (F5). Não é necessário nenhum processo de build.

As cores principais do tema estão definidas como variáveis CSS no início do arquivo:

```css
:root {
  --accent: #cc1111;    /* vermelho principal */
  --bg: #f5f5f5;        /* fundo geral */
  --surface: #ffffff;   /* cards e modais */
}
```
