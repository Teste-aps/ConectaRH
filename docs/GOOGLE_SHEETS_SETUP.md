# 📊 Guia: Configurar Google Sheets para ConectaRH

## Passo 1: Criar a Planilha Principal

1. Acesse [Google Sheets](https://sheets.google.com)
2. Clique em **"+ Criar"** → **Planilha em branco**
3. Renomeie para: `ConectaRH-Database`
4. Compartilhe com sua conta de serviço (ou deixe privada para você)

## Passo 2: Criar as Abas (Tabelas)

Delete a aba padrão "Plan1" e crie as seguintes abas:

### 1️⃣ **Usuarios**
```
Colunas:
A: ID (número sequencial, começar em 1)
B: Email
C: Senha (SERÁ PREENCHIDA PELO CÓDIGO - hash bcrypt)
D: Nome
E: Role (valores: "RH" ou "Colaborador")
F: Status (valores: "Ativo" ou "Inativo")
G: DataCriacao (data)
H: Departamento (opcional)

Exemplo de dados iniciais:
ID | Email                | Senha    | Nome           | Role        | Status | DataCriacao | Departamento
1  | rh@empresa.com      | xxx...   | Gerente RH     | RH          | Ativo  | 2026-09-30  | RH
2  | joao@empresa.com    | xxx...   | João Silva     | Colaborador | Ativo  | 2026-09-30  | TI
3  | maria@empresa.com   | xxx...   | Maria Santos   | Colaborador | Ativo  | 2026-09-30  | Vendas
```

### 2️⃣ **Avisos**
```
Colunas:
A: ID
B: Titulo
C: Conteudo (texto longo)
D: Autor (email ou nome de quem criou)
E: DataPublicacao
F: DataAtualizacao (última edição)
G: Status (Ativo/Inativo)
H: Fixado (TRUE/FALSE - aparece no topo)

Exemplo:
ID | Titulo                  | Conteudo          | Autor          | DataPublicacao | Status | Fixado
1  | Empresa em folia!       | Lorem ipsum...    | rh@empresa.com | 2026-09-30     | Ativo  | TRUE
```

### 3️⃣ **Eventos**
```
Colunas:
A: ID
B: Titulo
C: Descricao
D: DataEvento (data do evento)
E: Hora (horário)
F: Local (endereço ou link de reunião)
G: Autor
H: CapacidadeMaxima (número de pessoas)
I: Status (Planejado/Acontecendo/Encerrado/Cancelado)
J: DataCriacao
K: ImagemURL (link para imagem de banner)

Exemplo:
ID | Titulo           | Descricao     | DataEvento | Hora  | Local          | Capacidade | Status
1  | Confraternização | Festa de fim  | 2026-10-15 | 18:00 | Sala Auditório | 100        | Planejado
```

### 4️⃣ **EventosInscritos**
```
Colunas:
A: ID
B: EventoID (referência para Eventos)
C: UsuarioID (referência para Usuarios)
D: DataInscricao
E: Status (Inscrito/Cancelado)

Exemplo:
ID | EventoID | UsuarioID | DataInscricao | Status
1  | 1        | 2         | 2026-09-30    | Inscrito
2  | 1        | 3         | 2026-09-30    | Inscrito
```

### 5️⃣ **Mural**
```
Colunas:
A: ID
B: Titulo
C: Conteudo
D: ImagemURL (URL da imagem)
E: Categoria (ex: Comunicados, Cultura, Social)
F: Autor
G: DataPublicacao
H: DataAtualizacao
I: Fixado (TRUE/FALSE)
J: Status (Publicado/Rascunho/Arquivado)

Exemplo:
ID | Titulo              | Conteudo  | ImagemURL | Categoria    | Status
1  | Novo projeto        | Lorem...  | [URL]     | Comunicados  | Publicado
2  | Time de TI          | Foto do   | [URL]     | Cultura      | Publicado
```

### 6️⃣ **MuralCurtidas**
```
Colunas:
A: ID
B: MuralID (referência para Mural)
C: UsuarioID (referência para Usuarios)
D: DataCurtida

Exemplo:
ID | MuralID | UsuarioID | DataCurtida
1  | 1       | 2         | 2026-09-30
```

### 7️⃣ **MuralComentarios**
```
Colunas:
A: ID
B: MuralID
C: UsuarioID
D: Texto (comentário)
E: DataComentario

Exemplo:
ID | MuralID | UsuarioID | Texto              | DataComentario
1  | 1       | 2         | Adorei a notícia! | 2026-09-30
```

### 8️⃣ **PesquisaClima**
```
Colunas:
A: ID
B: Titulo
C: Descricao
D: DataInicio
E: DataFim
F: Status (Ativa/Encerrada)
G: DataCriacao
H: CriadorID (quem criou)

Exemplo:
ID | Titulo           | DataInicio | DataFim    | Status | DataCriacao
1  | Clima 2026 Q4    | 2026-09-30 | 2026-10-31 | Ativa  | 2026-09-30
```

### 9️⃣ **RespostasPesquisa**
```
Colunas:
A: ID
B: PesquisaID
C: UsuarioID (SERÁ NULL - resposta anônima)
D: Lideranca (0-5)
E: Ambiente (0-5)
F: Comunicacao (0-5)
G: Beneficios (0-5)
H: Crescimento (0-5)
I: Equilibrio (0-5)
J: Comentario (texto, pode estar vazio)
K: DataResposta

Exemplo:
ID | PesquisaID | UsuarioID | Lideranca | Ambiente | Comunicacao | DataResposta
1  | 1          | NULL      | 4         | 3        | 5           | 2026-09-30
2  | 1          | NULL      | 3         | 2        | 4           | 2026-09-30
```

## Passo 3: Formatar Cabeçalhos (Header)

Para cada aba:

1. Selecione a primeira linha
2. Clique em **Format** → **Bold** (deixar em negrito)
3. Adicione uma cor de fundo (ex: cinza claro)

## Passo 4: Proteger Dados Importantes

Para proteger a aba "Usuarios" (dados sensíveis):

1. Clique na aba "Usuarios"
2. Clique no menu **⋮** (três pontos)
3. Selecione **Protect sheets and ranges**
4. Marque "Sua planilha"
5. Configure para que apenas você (RH) possa editar

## Passo 5: Copiar o ID da Planilha

Na URL: `https://docs.google.com/spreadsheets/d/{SHEET_ID}/edit`

Copie o `{SHEET_ID}` - você precisará dele no Google Apps Script!

Exemplo:
```
Sua URL: https://docs.google.com/spreadsheets/d/1a2b3c4d5e6f7g8h9i0j/edit
Seu SHEET_ID: 1a2b3c4d5e6f7g8h9i0j
```

## Passo 6: Gerar Primeiro Usuário de Teste

Na aba **Usuarios**, adicione manualmente:

```
ID | Email              | Senha    | Nome       | Role        | Status
1  | teste@empresa.com  | 123456   | Teste RH   | RH          | Ativo
```

**Nota:** A senha será hashida pelo código do Apps Script depois.

---

## ✅ Checklist de Setup

- [ ] Planilha "ConectaRH-Database" criada
- [ ] 9 abas criadas (Usuarios, Avisos, Eventos, etc)
- [ ] Cabeçalhos adicionados em cada aba
- [ ] Primeira linha formatada (bold + cor)
- [ ] Aba "Usuarios" protegida
- [ ] SHEET_ID copiado e anotado
- [ ] Primeiro usuário adicionado (teste)

---

## Próximo Passo

Depois de completar este setup, vá para: `docs/GOOGLE_APPS_SCRIPT.md`
