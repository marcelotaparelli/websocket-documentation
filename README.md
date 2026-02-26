# websocket-documentation

- websocket permite a interação em tempo real
- websocket é um protocolo, assim com o HTTP. A comunicação aqui é baseada em eventos.
- é mais dinâmico
- cliente se conecta ao servidor criando um socket
- o cliente ou o servidor podem emitir eventos (comunicação bidirecional)

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


