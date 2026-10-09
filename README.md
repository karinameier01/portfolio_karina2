# KM-OS — Portfólio de Karina Meier

Portfólio interativo em Vue 3 e Vite, apresentado como uma área de trabalho de sistema operacional. O conteúdo destaca a formação técnica de Karina Meier, estudante de Técnico em Informática para Internet no SENAC (2024–2026), em Joinville — SC.

## Executar localmente

```sh
npm install
npm run dev
```

### Usar o botão Go Live do VS Code

O espaço de trabalho recomenda a extensão **Live Server** e configura o botão **Go Live** para abrir a versão compilada do site em `http://127.0.0.1:5500`. Ao abrir a pasta no VS Code, autorize a tarefa automática **Prepare site for Go Live**: ela compila o projeto e atualiza `dist` quando os arquivos-fonte mudarem. Depois, clique em **Go Live** na barra de status.

O Live Server serve os arquivos de produção compilados do Vue; para desenvolvimento com atualização HMR, use `npm run dev` em `http://localhost:5173`.

Para gerar a versão de produção:

```sh
npm run build
npm run preview
```

## Aplicativos do KM-OS

- **Sobre Mim.exe** — perfil, formação e participações.
- **Projetos.app** — seleção de projetos e acesso aos repositórios GitHub.
- **Habilidades.sys** — tecnologias e competências em desenvolvimento.
- **Currículo.pdf** — currículo com opção de imprimir ou salvar como PDF.
- **Contato.net** — links de contato e formulário de demonstração; o formulário não envia nem armazena dados.
- **Terminal-KM.sh** — comandos `help`, `about`, `skills`, `projects`, `contact` e `clear`.

No desktop, abra os atalhos e arraste, redimensione, minimize, maximize ou feche as janelas. Em telas menores, os aplicativos se adaptam a uma visualização em tela cheia, com navegação pela barra inferior.

Os detalhes técnicos de cada projeto podem ser conferidos diretamente nos repositórios do perfil [karinameier01](https://github.com/karinameier01).
