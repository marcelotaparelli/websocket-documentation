# websocket-documentation

- websocket permite a interação em tempo real
- websocket é um protocolo, assim com o HTTP. A comunicação aqui é baseada em eventos.
- é mais dinâmico
- cliente se conecta ao servidor criando um socket
- o cliente ou o servidor podem emitir eventos (comunicação bidirecional)


## O Fluxo do Algoritmo

1. **Conexão:** O cliente abre uma conexão via WebSocket com o servidor.
2. **Evento de Digitação:** Sempre que o usuário pressiona uma tecla, o frontend envia o novo conteúdo para o servidor.
3. **Broadcast:** O servidor recebe o conteúdo e o retransmite para **todos os outros** usuários conectados, exceto para quem enviou.
4. **Sincronização:** O frontend dos outros usuários atualiza o campo de texto automaticamente.

<br><br>


## Implementação com Node.js e Socket.io

### 1. Servidor (`server.js`)

O servidor age como o "maestro", garantindo que as mensagens cheguem a todos.

```javascript
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = new Server(server);

// Serve o arquivo HTML básico
app.get('/', (req, res) => {
  res.sendFile(__dirname + '/index.html');
});

io.on('connection', (socket) => {
  console.log('Usuário conectado:', socket.id);

  // Escuta mudanças no documento
  socket.on('edit-file', (content) => {
    // Envia para todos os outros usuários, menos para o remetente
    socket.broadcast.emit('update-file', content);
  });

  socket.on('disconnect', () => {
    console.log('Usuário desconectado');
  });
});

server.listen(3000, () => {
  console.log('Servidor rodando em http://localhost:3000');
});

```

<br><br>


### 2. Cliente (`index.html`)

O cliente escuta o evento de digitação e avisa o servidor.

```html
<textarea id="editor" style="width:100%; height:300px;"></textarea>

<script src="/socket.io/socket.io.js"></script>
<script>
  const socket = io();
  const editor = document.getElementById('editor');

  // Ao digitar: envia o conteúdo para o servidor
  editor.addEventListener('input', () => {
    socket.emit('edit-file', editor.value);
  });

  // Ao receber do servidor: atualiza o texto dos outros usuários
  socket.on('update-file', (newContent) => {
    editor.value = newContent;
  });
</script>

```

<br><br>


## Desafios de Edição Simultânea Real

O código acima funciona para casos simples, mas em produção, enfrentaríamos problemas como o **conflito de cursores** (o cursor do usuário pula para o fim do texto quando recebe uma atualização).

Para sistemas complexos como o Google Docs, existem duas estratégias principais:

* **OT (Operational Transformation):** O servidor transforma as operações (ex: "inserir 'A' na posição 5") para que façam sentido em todas as telas, mesmo que cheguem atrasadas.
* **CRDT (Conflict-free Replicated Data Types):** Uma estrutura de dados inteligente onde a ordem das edições não importa; o resultado final será sempre o mesmo para todos.

<br><br>

### Como testar:

1. Salve os arquivos e rode `npm install express socket.io`.
2. Execute `node server.js`.
3. Abra **duas ou mais abas** no navegador em `localhost:3000`.
4. Digite em uma delas 



```
npm install socket.io

// servidor.js

import http from "http";
import { Server } from "socket.io";

...
const servidorHttp = http.createServer(app);

servidorHttp.listen(porta, ...)

const io = new Server(servidorHttp);

io.on("connection", () => {
  console.log("Um cliente se conectou");
});
...
```

```
// no html

<script src="/socket.io/socket.io.js"></script>
<script src="documento.js" type="module"></script>
```

```
// documento.js

const socket = io();
```

**obtendo id do socket**
```
// servidor.js
...
const io = new Server(servidorHttp);

export default io;
```

```
// src/socket-back.js
import io from "./servidor.js"

io.on("connection", (socket) => {
  console.log("Um cliente se conectou! ID:", socket,id);
});

// alterar a fonte do app para socket-back.js
```

**Emitindo eventos**
```
// document.js

const socket = io();

const textoEditor = document.getElementById("editor-texto");

textoEditor.addEventListener("keyup", () => {
  socket.emit("texto_editor", textoEditor.value);
});
```
```
// src/socket-back.js

import io from "./servidor.js"

io.on("connection", (socket) => {
  console.log("Um cliente se conectou! ID:", socket,id);

  socket.on ("texto_editor", (texto) => {
    console.log("texto");
  });
});
```

<br><br>

**Do servidor para os clientes**

```
// src/socket-back.js

import io from "./servidor.js"

io.on("connection", (socket) => {
  console.log("Um cliente se conectou! ID:", socket,id);

  socket.on ("texto_editor", (texto) => {
    socket.broadcast.emit("texto_editor_clientes", texto);
  });
});
```

```
// documento.js

const socket = io();

const textoEditor = document.getElementById("editor-texto");

textoEditor.addEventListener("keyup", () => {
  socket.emit("texto_editor", textoEditor.value);
});

socket.on("text_editor_clientes", (texto) => {
  textoEditor.value = texto;
})
```

**organizando arquivos**
```
// public socket-front-documento.js
import { atualizaTextoEditor } from "./documento.js";

const socket = io();

function emitirTextoEditor(texto) {
  socket.emit("texto_editor", texto);
}

socket.on("text_editor_clientes", (texto) => {
  atualizaTextoEditor(texto);
})

export { emitirTextoEditor };
```
```
// documento.js
import { emitirTextoEditor } from "./socket-front-documento.js";

const textoEditor = document.getElementById("editor-texto");

textoEditor.addEventListener("keyup", () => {
  emitirTextoEditor()(textoEditor.value);
});

function atualizaTextoEditor(text) {
  textoEditor.value = texto;
}

export { atualizaTextoEditor };
```

<br><br>

**Salas do socket.io**
- Sala é um conceito que envolve o agrupamento de conexões e agrupar clientes;

```
// documento.js
import { emitirTextoEditor, selecionarDocumento } from "./socket-front-documento.js";

const parametros = new URLSearchParams(window.location.search);
const nomeDocumento = parametros.get("nome");

const textoEditor = document.getElementById("editor-texto");
const tituloDocumento = document.getElementById("titulo-documento");

tituloDocumento.textContent = nomeDocumento || "Documento sem titulo";

selecionarDocumento(nomeDocumento);

textoEditor.addEventListener("keyup", () => {
  emitirTextoEditor(textoEditor.value);
});

function atualizaTextoEditor(text) {
  textoEditor.value = texto;
}

export { atualizaTextoEditor };
```

```
// public socket-front-documento.js
import { atualizaTextoEditor } from "./documento.js";

const socket = io();

function selecionarDocumento(nome) {
  socket.emit("selecionar_documento", nome);
}

function emitirTextoEditor(texto) {
  socket.emit("texto_editor", texto);
}

socket.on("text_editor_clientes", (texto) => {
  atualizaTextoEditor(texto);
})

export { emitirTextoEditor, selecionarDocumento };
```

```
// src/socket-back.js

import io from "./servidor.js"

io.on("connection", (socket) => {
  console.log("Um cliente se conectou! ID:", socket,id);

  socket.on("selecionar_documento", (nomeDocumento) => {
    socket.join(nomeDocumento);
  })

  socket.on ("texto_editor", (texto) => {
    socket.broadcast.emit("texto_editor_clientes", texto);

    socket.to("Javascript").emit("texto_editor_clientes", texto);
  });

});
```

<br><br>

**Enviando para as salas corretas**

```
// src/socket-back.js

import io from "./servidor.js"

io.on("connection", (socket) => {
  console.log("Um cliente se conectou! ID:", socket,id);

  socket.on("selecionar_documento", (nomeDocumento) => {
    socket.join(nomeDocumento);
  })

  socket.on ("texto_editor", ({ texto, nomeDocumento }) => {
    socket.to("nomeDocumento").emit("texto_editor_clientes", texto);
  });

});
```

```
// public socket-front-documento.js
import { atualizaTextoEditor } from "./documento.js";

const socket = io();

function selecionarDocumento(nome) {
  socket.emit("selecionar_documento", nome);
}

function emitirTextoEditor(dados) {
  socket.emit("texto_editor", dados);
}

socket.on("text_editor_clientes", (texto) => {
  atualizaTextoEditor(texto);
})

export { emitirTextoEditor, selecionarDocumento };
```


```
// documento.js
import { emitirTextoEditor, selecionarDocumento } from "./socket-front-documento.js";

const parametros = new URLSearchParams(window.location.search);
const nomeDocumento = parametros.get("nome");

const textoEditor = document.getElementById("editor-texto");
const tituloDocumento = document.getElementById("titulo-documento");

tituloDocumento.textContent = nomeDocumento || "Documento sem titulo";

selecionarDocumento(nomeDocumento);

textoEditor.addEventListener("keyup", () => {
  emitirTextoEditor({
    texto: textoEditor.value,
    nomeDocumento: nomeDocumento
  });
});

function atualizaTextoEditor(text) {
  textoEditor.value = texto;
}

export { atualizaTextoEditor };
```

<br><br>

**Guardando os dados localmente**


```
// src/socket-back.js

import io from "./servidor.js"

const documentos = [
  {
    nome: "Javascript",
    text: "texto de js..."
  },
  {
    nome: "Socket io",
    texto: "texto de io..."
  },
];

io.on("connection", (socket) => {
  console.log("Um cliente se conectou! ID:", socket,id);

  socket.on("selecionar_documento", (nomeDocumento) => {
    const documento = encontrarDocumento(nomeDocumento);
    socket.join(nomeDocumento);
  })

  socket.on ("texto_editor", ({ texto, nomeDocumento }) => {
    socket.to("nomeDocumento").emit("texto_editor_clientes", texto);
  });
});

function encontrarDocunento(nome) {
  const documento = documentos.find((documento) => {
    return documento.nome === nome;
  });

  return documento;
}
```

<br><br>

https://github.com/alura-cursos/alura-docs-2


