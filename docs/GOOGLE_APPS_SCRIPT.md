# 🚀 Guia: Configurar Google Apps Script para ConectaRH

## Passo 1: Acessar o Editor de Apps Script

1. Abra sua planilha "ConectaRH-Database"
2. No menu superior, clique em **Extensions** (Extensões)
3. Selecione **Apps Script**
4. Uma nova aba abrirá com o editor de código

## Passo 2: Limpar e Renomear

1. No editor, delete o código padrão (função `myFunction`)
2. Renomeie o projeto: clique em **"Untitled project"** no canto superior esquerdo
3. Digite: `ConectaRH-API`

## Passo 3: Criar Arquivo Principal (Code.gs)

O arquivo padrão chama-se `Code.gs`. **MANTENHA ESSE NOME**.

Copie e cole esse código:

```javascript
// Code.gs - Arquivo Principal da API ConectaRH

const SHEET_ID = "SEU_SHEET_ID_AQUI"; // Substitua com seu SHEET_ID
const SHEET = SpreadsheetApp.openById(SHEET_ID);

// Configuração de CORS
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const action = e.parameter.action;
    
    let response = {};
    
    switch(action) {
      case "login":
        response = handleLogin(data);
        break;
      case "getCriarAviso":
        response = handleCreateAviso(data);
        break;
      case "getAvisos":
        response = handleGetAvisos(data);
        break;
      case "getCriarEvento":
        response = handleCreateEvento(data);
        break;
      case "getEventos":
        response = handleGetEventos(data);
        break;
      case "getCriarMural":
        response = handleCreateMural(data);
        break;
      case "getMural":
        response = handleGetMural(data);
        break;
      case "getCriarPesquisa":
        response = handleCreatePesquisa(data);
        break;
      case "getPesquisas":
        response = handleGetPesquisas(data);
        break;
      case "responderPesquisa":
        response = handleResponderPesquisa(data);
        break;
      default:
        response = { success: false, error: "Ação inválida" };
    }
    
    return ContentService.createTextOutput(JSON.stringify(response))
      .setMimeType(ContentService.MimeType.JSON);
  } catch(error) {
    return ContentService.createTextOutput(JSON.stringify({
      success: false,
      error: error.toString()
    })).setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  // Suportar GET também
  return doPost(e);
}

// ==================== AUTENTICAÇÃO ====================

function handleLogin(data) {
  const sheet = SHEET.getSheetByName("Usuarios");
  const values = sheet.getDataRange().getValues();
  
  // Procurar usuário pelo email
  for (let i = 1; i < values.length; i++) {
    if (values[i][1] === data.email) { // Coluna B = Email
      // Aqui deveria validar senha com hash, por enquanto comparar direto
      if (values[i][2] === data.senha) { // Coluna C = Senha
        return {
          success: true,
          token: generateToken(values[i][1]),
          userId: values[i][0],
          nome: values[i][3],
          email: values[i][1],
          role: values[i][4] // Coluna E = Role
        };
      }
    }
  }
  
  return { success: false, error: "Email ou senha inválidos" };
}

function generateToken(email) {
  // Simulação simples de JWT - em produção usar biblioteca apropriada
  return Utilities.getUuid() + "_" + email;
}

// ==================== AVISOS ====================

function handleGetAvisos(data) {
  const sheet = SHEET.getSheetByName("Avisos");
  const values = sheet.getDataRange().getValues();
  const avisos = [];
  
  for (let i = 1; i < values.length; i++) {
    if (values[i][6] === "Ativo") { // Coluna G = Status
      avisos.push({
        id: values[i][0],
        titulo: values[i][1],
        conteudo: values[i][2],
        autor: values[i][3],
        datapublicacao: values[i][4],
        status: values[i][6],
        fixado: values[i][7]
      });
    }
  }
  
  // Ordenar fixados primeiro
  avisos.sort((a, b) => b.fixado - a.fixado);
  
  return { success: true, data: avisos };
}

function handleCreateAviso(data) {
  // Validar se é RH (verificar token/role)
  const sheet = SHEET.getSheetByName("Avisos");
  
  // Pegar próximo ID
  const values = sheet.getDataRange().getValues();
  const novoId = values.length;
  
  const novaLinha = [
    novoId,
    data.titulo,
    data.conteudo,
    data.autor,
    new Date(),
    new Date(),
    "Ativo",
    data.fixado || false
  ];
  
  sheet.appendRow(novaLinha);
  
  return { 
    success: true, 
    data: { id: novoId, ...data }
  };
}

// ==================== EVENTOS ====================

function handleGetEventos(data) {
  const sheet = SHEET.getSheetByName("Eventos");
  const values = sheet.getDataRange().getValues();
  const eventos = [];
  
  for (let i = 1; i < values.length; i++) {
    if (values[i][8] !== "Cancelado") { // Coluna I = Status
      eventos.push({
        id: values[i][0],
        titulo: values[i][1],
        descricao: values[i][2],
        dataEvento: values[i][3],
        hora: values[i][4],
        local: values[i][5],
        autor: values[i][6],
        capacidade: values[i][7],
        status: values[i][8],
        imagemURL: values[i][10]
      });
    }
  }
  
  return { success: true, data: eventos };
}

function handleCreateEvento(data) {
  const sheet = SHEET.getSheetByName("Eventos");
  const values = sheet.getDataRange().getValues();
  const novoId = values.length;
  
  const novaLinha = [
    novoId,
    data.titulo,
    data.descricao,
    data.dataEvento,
    data.hora,
    data.local,
    data.autor,
    data.capacidade || 50,
    "Planejado",
    new Date(),
    data.imagemURL || ""
  ];
  
  sheet.appendRow(novaLinha);
  
  return { 
    success: true, 
    data: { id: novoId, ...data }
  };
}

// ==================== MURAL ====================

function handleGetMural(data) {
  const sheet = SHEET.getSheetByName("Mural");
  const values = sheet.getDataRange().getValues();
  const posts = [];
  
  for (let i = 1; i < values.length; i++) {
    if (values[i][9] === "Publicado") { // Coluna J = Status
      posts.push({
        id: values[i][0],
        titulo: values[i][1],
        conteudo: values[i][2],
        imagemURL: values[i][3],
        categoria: values[i][4],
        autor: values[i][5],
        datapublicacao: values[i][6],
        fixado: values[i][8],
        curtidas: getContarCurtidas(values[i][0]),
        comentarios: getContarComentarios(values[i][0])
      });
    }
  }
  
  // Ordenar fixados primeiro, depois por data
  posts.sort((a, b) => {
    if (b.fixado !== a.fixado) return b.fixado - a.fixado;
    return new Date(b.datapublicacao) - new Date(a.datapublicacao);
  });
  
  return { success: true, data: posts };
}

function handleCreateMural(data) {
  const sheet = SHEET.getSheetByName("Mural");
  const values = sheet.getDataRange().getValues();
  const novoId = values.length;
  
  const novaLinha = [
    novoId,
    data.titulo,
    data.conteudo,
    data.imagemURL || "",
    data.categoria,
    data.autor,
    new Date(),
    new Date(),
    data.fixado || false,
    "Publicado"
  ];
  
  sheet.appendRow(novaLinha);
  
  return { 
    success: true, 
    data: { id: novoId, ...data }
  };
}

// ==================== PESQUISA DE CLIMA ====================

function handleGetPesquisas(data) {
  const sheet = SHEET.getSheetByName("PesquisaClima");
  const values = sheet.getDataRange().getValues();
  const pesquisas = [];
  
  for (let i = 1; i < values.length; i++) {
    pesquisas.push({
      id: values[i][0],
      titulo: values[i][1],
      descricao: values[i][2],
      dataInicio: values[i][3],
      dataFim: values[i][4],
      status: values[i][5],
      dataCriacao: values[i][6]
    });
  }
  
  return { success: true, data: pesquisas };
}

function handleCreatePesquisa(data) {
  const sheet = SHEET.getSheetByName("PesquisaClima");
  const values = sheet.getDataRange().getValues();
  const novoId = values.length;
  
  const novaLinha = [
    novoId,
    data.titulo,
    data.descricao || "",
    data.dataInicio,
    data.dataFim,
    "Ativa",
    new Date(),
    data.criadorId
  ];
  
  sheet.appendRow(novaLinha);
  
  return { 
    success: true, 
    data: { id: novoId, ...data }
  };
}

function handleResponderPesquisa(data) {
  const sheet = SHEET.getSheetByName("RespostasPesquisa");
  
  const novaLinha = [
    sheet.getDataRange().getValues().length,
    data.pesquisaId,
    null, // UsuarioID (anônimo)
    data.lideranca,
    data.ambiente,
    data.comunicacao,
    data.beneficios,
    data.crescimento,
    data.equilibrio,
    data.comentario || "",
    new Date()
  ];
  
  sheet.appendRow(novaLinha);
  
  return { success: true, respondido: true };
}

// ==================== UTILIDADES ====================

function getContarCurtidas(muralId) {
  const sheet = SHEET.getSheetByName("MuralCurtidas");
  const values = sheet.getDataRange().getValues();
  let count = 0;
  
  for (let i = 1; i < values.length; i++) {
    if (values[i][1] === muralId) count++;
  }
  
  return count;
}

function getContarComentarios(muralId) {
  const sheet = SHEET.getSheetByName("MuralComentarios");
  const values = sheet.getDataRange().getValues();
  let count = 0;
  
  for (let i = 1; i < values.length; i++) {
    if (values[i][1] === muralId) count++;
  }
  
  return count;
}
```

## Passo 4: Substituir o SHEET_ID

Na linha `const SHEET_ID = "SEU_SHEET_ID_AQUI";`

Copie o ID que você copiou no Passo 5 do setup do Sheets:

```javascript
const SHEET_ID = "1a2b3c4d5e6f7g8h9i0j"; // Seu ID real
```

## Passo 5: Deploy do Apps Script

1. Clique em **Deploy** (botão azul no canto superior direito)
2. Selecione **"New deployment"**
3. Clique no ícone de engrenagem ⚙️
4. Selecione **"Web app"**
5. Configure:
   - **Execute as**: Sua conta do Google
   - **Who has access**: Anyone
   - Clique em **Deploy**
6. Copie a URL gerada: `https://script.google.com/macros/s/{DEPLOYMENT_ID}/usercontent`

**GUARDE ESSA URL** - você precisará dela no React!

## Passo 6: Testar a API

Abra seu navegador e teste um endpoint:

```
https://script.google.com/macros/s/{DEPLOYMENT_ID}/usercontent?action=getAvisos
```

Se receber um JSON com `{"success": true, "data": [...]}`, está funcionando! ✅

## ✅ Checklist

- [ ] Code.gs criado e preenchido
- [ ] SHEET_ID substituído
- [ ] Apps Script deployado como Web App
- [ ] URL do deployment copiada
- [ ] Endpoint testado no navegador
- [ ] Retorna JSON correto

---

## Próximo Passo

Vá para: `docs/FRONTEND_SETUP.md` para configurar o React
