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
