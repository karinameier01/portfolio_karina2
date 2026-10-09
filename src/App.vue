<script setup>
import { computed, nextTick, onBeforeUnmount, onMounted, ref, watch } from 'vue'

const github = 'https://github.com/karinameier01'
const linkedIn = 'https://www.linkedin.com/in/karina-meier-69aba3368/'
const email = 'karinameier28@gmail.com'

const apps = ref([
  { id: 'about', name: 'Sobre Mim.exe', shortName: 'Sobre Mim', icon: 'person', open: true, minimized: false, maximized: false, zIndex: 2, x: 250, y: 100 },
  { id: 'projects', name: 'Projetos.app', shortName: 'Projetos', icon: 'folder', open: false, minimized: false, maximized: false, zIndex: 1, x: 310, y: 120 },
  { id: 'skills', name: 'Habilidades.sys', shortName: 'Habilidades', icon: 'code', open: false, minimized: false, maximized: false, zIndex: 1, x: 330, y: 95 },
  { id: 'resume', name: 'Currículo.pdf', shortName: 'Currículo', icon: 'document', open: false, minimized: false, maximized: false, zIndex: 1, x: 290, y: 115 },
  { id: 'contact', name: 'Contato.net', shortName: 'Contato', icon: 'mail', open: false, minimized: false, maximized: false, zIndex: 1, x: 350, y: 130 },
  { id: 'terminal', name: 'Terminal-KM.sh', shortName: 'Terminal', icon: 'terminal', open: false, minimized: false, maximized: false, zIndex: 1, x: 300, y: 90 },
])

const desktopApps = computed(() => apps.value)
const visibleApps = computed(() => apps.value.filter((app) => app.open && !app.minimized))
const currentTime = ref('')
const currentDate = ref('')
const activeAppId = ref('about')
const startMenuOpen = ref(false)
const dragState = ref(null)
const contactForm = ref({ name: '', email: '', message: '' })
const contactFeedback = ref('')
const terminalInput = ref('')
const terminalLines = ref([
  { type: 'system', text: 'KM Terminal 2.6 — ambiente interativo' },
  { type: 'system', text: 'Digite help para consultar os comandos disponíveis.' },
])
const terminalOutput = ref(null)
let clockTimer

const projects = [
  { name: 'SmartEvent', group: 'Web e aplicações', description: 'Projeto disponível no GitHub. Acesse o repositório para consultar a proposta, a documentação e as tecnologias utilizadas.', objective: 'Explorar o projeto e seus detalhes diretamente na fonte.', technologies: 'Consulte o repositório', repository: 'SmartEvent' },
  { name: 'Bookiss', group: 'Web e aplicações', description: 'Projeto disponível no GitHub. Acesse o repositório para conhecer sua proposta e implementação.', objective: 'Conhecer a solução e sua estrutura pelo repositório.', technologies: 'Consulte o repositório', repository: 'Bookiss' },
  { name: 'Jogo de Cartas Burro', group: 'Mobile', description: 'Projeto mobile desenvolvido no contexto de jogos digitais e aplicações interativas.', objective: 'Desenvolver uma experiência de jogo de cartas para dispositivos móveis.', technologies: 'Vue · Ionic · Capacitor · SQLite · Bluetooth', repository: 'joguinho_burro' },
  { name: 'LUMEN', group: 'Inovação e pesquisa', description: 'Projeto relacionado a inovação e pesquisa, apresentado entre os projetos do portfólio.', objective: 'Investigar e desenvolver uma proposta de inovação.', technologies: 'Consulte o repositório', repository: '' },
  { name: 'AlbumFig', group: 'Web e aplicações', description: 'Repositório de projeto listado no GitHub. Consulte a página para ver detalhes e tecnologias.', objective: 'Explorar o projeto e a implementação no repositório.', technologies: 'Consulte o repositório', repository: 'AlbumFig' },
  { name: 'FigAlbum', group: 'Web e aplicações', description: 'Repositório de projeto listado no GitHub. Consulte a página para ver detalhes e tecnologias.', objective: 'Explorar o projeto e a implementação no repositório.', technologies: 'Consulte o repositório', repository: 'FigAlbum' },
  { name: 'meuApp', group: 'Web e aplicações', description: 'Repositório de projeto listado no GitHub. Consulte a página para ver detalhes e tecnologias.', objective: 'Explorar o projeto e a implementação no repositório.', technologies: 'Consulte o repositório', repository: 'meuApp' },
]

const skillGroups = [
  { title: 'Desenvolvimento Web', items: ['HTML', 'CSS', 'JavaScript', 'Vue.js'] },
  { title: 'Aplicações Mobile', items: ['Ionic', 'Vue', 'Capacitor'] },
  { title: 'Back-end e Banco de Dados', items: ['PHP', 'SQL', 'SQLite'] },
  { title: 'Hardware e Programação', items: ['C++', 'Arduino', 'Tinkercad'] },
  { title: 'Ferramentas e Outros', items: ['Scratch', 'Excel', 'Word'] },
]

const courses = [
  'Excel Nível Básico — SENAI (20 horas)',
  'Introdução à Programação — SENAI (10 horas)',
  'Introdução aos Jogos Digitais — SENAI',
]

const terminalCommands = ['help', 'about', 'skills', 'projects', 'contact', 'clear']

function updateClock() {
  const now = new Date()
  currentTime.value = now.toLocaleTimeString('pt-BR', { hour: '2-digit', minute: '2-digit', hour12: false })
  currentDate.value = now.toLocaleDateString('pt-BR', { day: '2-digit', month: '2-digit', year: 'numeric' })
}

function raiseWindow(id) {
  const app = apps.value.find((item) => item.id === id)
  if (!app) return
  app.zIndex = Math.max(...apps.value.map((item) => item.zIndex)) + 1
  activeAppId.value = id
}

function openApp(id) {
  const app = apps.value.find((item) => item.id === id)
  if (!app) return
  if (!app.open && window.innerWidth > 760) {
    app.x = Math.max(24, Math.round((window.innerWidth - 690) / 2))
    app.y = Math.max(28, Math.round((window.innerHeight - 590) / 2) - 24)
  }
  app.open = true
  app.minimized = false
  raiseWindow(id)
  startMenuOpen.value = false
}

function closeApp(id) {
  const app = apps.value.find((item) => item.id === id)
  if (!app) return
  app.open = false
  app.minimized = false
  const next = apps.value.filter((item) => item.open && !item.minimized).sort((a, b) => b.zIndex - a.zIndex)[0]
  activeAppId.value = next?.id || ''
}

function minimizeApp(id) {
  const app = apps.value.find((item) => item.id === id)
  if (!app) return
  app.minimized = true
  const next = apps.value.filter((item) => item.open && !item.minimized).sort((a, b) => b.zIndex - a.zIndex)[0]
  activeAppId.value = next?.id || ''
}

function toggleMaximize(app) {
  app.maximized = !app.maximized
  raiseWindow(app.id)
}

function clickTaskbarApp(app) {
  if (app.open && !app.minimized && activeAppId.value === app.id) {
    minimizeApp(app.id)
    return
  }
  openApp(app.id)
}

function startDrag(event, app) {
  if (event.button !== 0 || app.maximized || window.innerWidth <= 760 || event.target.closest('button')) return
  event.preventDefault()
  dragState.value = { id: app.id, pointerX: event.clientX, pointerY: event.clientY, x: app.x, y: app.y }
  raiseWindow(app.id)
}

function moveWindow(event) {
  if (!dragState.value) return
  const app = apps.value.find((item) => item.id === dragState.value.id)
  if (!app) return
  const maxX = Math.max(12, window.innerWidth - 240)
  const maxY = Math.max(12, window.innerHeight - 112)
  app.x = Math.min(maxX, Math.max(12, dragState.value.x + event.clientX - dragState.value.pointerX))
  app.y = Math.min(maxY, Math.max(12, dragState.value.y + event.clientY - dragState.value.pointerY))
}

function stopDrag() {
  dragState.value = null
}

function sendMessage() {
  contactFeedback.value = 'Mensagem validada. Este formulário é uma demonstração e não envia dados; escreva diretamente para o e-mail abaixo.'
}

function printResume() {
  document.body.classList.add('print-resume')
  window.print()
  window.setTimeout(() => document.body.classList.remove('print-resume'), 1000)
}

function runCommand() {
  const command = terminalInput.value.trim().toLowerCase()
  if (!command) return
  if (command === 'clear') {
    terminalLines.value = []
    terminalInput.value = ''
    return
  }

  const responses = {
    help: `Comandos disponíveis:\n${terminalCommands.map((item) => `  ${item}`).join('\n')}`,
    about: 'Karina Meier — estudante de Técnico em Informática para Internet no SENAC (2024–2026), em Joinville — SC.',
    skills: 'Web: HTML, CSS, JavaScript, Vue.js\nMobile: Ionic, Vue, Capacitor\nBack-end e dados: PHP, SQL, SQLite\nHardware: C++, Arduino, Tinkercad\nFerramentas: Scratch, Excel, Word',
    projects: 'Projetos apresentados: SmartEvent, Bookiss, Jogo de Cartas Burro, LUMEN, AlbumFig, FigAlbum e meuApp.\nAbra Projetos.app para consultar os repositórios.',
    contact: `E-mail: ${email}\nGitHub: github.com/karinameier01\nLinkedIn: linkedin.com/in/karina-meier-69aba3368`,
  }
  terminalLines.value.push({ type: 'command', text: `karina@km-os:~$ ${command}` })
  terminalLines.value.push({ type: responses[command] ? 'output' : 'error', text: responses[command] || `Comando não reconhecido: ${command}. Digite help para ver as opções.` })
  terminalInput.value = ''
}

watch(terminalLines, async () => {
  await nextTick()
  if (terminalOutput.value) terminalOutput.value.scrollTop = terminalOutput.value.scrollHeight
}, { deep: true })

function onDocumentPointerDown(event) {
  if (startMenuOpen.value && !event.target.closest('.start-menu') && !event.target.closest('.start-button')) {
    startMenuOpen.value = false
  }
}

onMounted(() => {
  updateClock()
  clockTimer = window.setInterval(updateClock, 1000)
  window.addEventListener('pointermove', moveWindow)
  window.addEventListener('pointerup', stopDrag)
  document.addEventListener('pointerdown', onDocumentPointerDown)
})

onBeforeUnmount(() => {
  window.clearInterval(clockTimer)
  window.removeEventListener('pointermove', moveWindow)
  window.removeEventListener('pointerup', stopDrag)
  document.removeEventListener('pointerdown', onDocumentPointerDown)
})
</script>

<template>
  <div class="os-shell">
    <div class="wallpaper-art" aria-hidden="true">
      <div class="wallpaper-orbit orbit-one"></div>
      <div class="wallpaper-orbit orbit-two"></div>
    </div>

    <main class="desktop" aria-label="Área de trabalho de Karina Meier">
      <header class="desktop-top">
        <div class="identity">
          <span class="system-label">PORTFÓLIO DIGITAL <i></i> JOINVILLE — SC</span>
          <h1>Olá, eu sou a <span>Karina.</span></h1>
          <p>Estudante de tecnologia <b>/</b> Desenvolvedora em formação</p>
        </div>
        <a class="profile-link" :href="github" target="_blank" rel="noreferrer" aria-label="Abrir GitHub de Karina">
          <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M9 19c-4.3 1.4-4.3-2.5-6-3m12 6v-3.9a3.4 3.4 0 0 0-.9-2.7c3-.3 6.1-1.5 6.1-6.7a5.2 5.2 0 0 0-1.4-3.6 4.8 4.8 0 0 0-.1-3.6s-1.2-.4-3.9 1.4a13.5 13.5 0 0 0-7.1 0C5 1.1 3.8 1.5 3.8 1.5a4.8 4.8 0 0 0-.1 3.6 5.2 5.2 0 0 0-1.4 3.6c0 5.2 3.1 6.4 6.1 6.7a3.4 3.4 0 0 0-.9 2.7V22"/></svg>
          <span>GitHub</span><span class="external-mark">↗</span>
        </a>
      </header>

      <section class="desktop-shortcuts" aria-label="Aplicativos">
        <button v-for="app in desktopApps" :key="app.id" class="desktop-shortcut" type="button" @dblclick="openApp(app.id)" @click="openApp(app.id)">
          <span class="app-icon" :class="`icon-${app.icon}`">
            <svg v-if="app.icon === 'person'" viewBox="0 0 48 48" aria-hidden="true"><circle cx="24" cy="16" r="7"/><path d="M10 40c1.5-8 6-12 14-12s12.5 4 14 12"/></svg>
            <svg v-else-if="app.icon === 'folder'" viewBox="0 0 48 48" aria-hidden="true"><path d="M5 13a3 3 0 0 1 3-3h12l4 5h16a3 3 0 0 1 3 3v17a3 3 0 0 1-3 3H8a3 3 0 0 1-3-3z"/><path d="M5 19h38"/></svg>
            <svg v-else-if="app.icon === 'code'" viewBox="0 0 48 48" aria-hidden="true"><path d="m17 15-9 9 9 9M31 15l9 9-9 9M27 11l-6 26"/></svg>
            <svg v-else-if="app.icon === 'document'" viewBox="0 0 48 48" aria-hidden="true"><path d="M12 5h16l9 9v29H12z"/><path d="M28 5v10h9M18 24h13M18 30h13M18 36h9"/></svg>
            <svg v-else-if="app.icon === 'mail'" viewBox="0 0 48 48" aria-hidden="true"><rect x="6" y="11" width="36" height="27" rx="4"/><path d="m8 15 16 12 16-12"/></svg>
            <svg v-else viewBox="0 0 48 48" aria-hidden="true"><path d="m17 15-9 9 9 9M31 15l9 9-9 9M27 11l-6 26"/></svg>
          </span>
          <span class="shortcut-name">{{ app.shortName }}<small>{{ app.name.split('.').pop() }}</small></span>
        </button>
      </section>

      <div class="window-layer" aria-label="Janelas de aplicativos">
        <section
          v-for="app in apps"
          v-show="app.open && !app.minimized"
          :key="app.id"
          class="app-window"
          :class="{ active: activeAppId === app.id, maximized: app.maximized }"
          :style="{ left: `${app.x}px`, top: `${app.y}px`, zIndex: app.zIndex }"
          :aria-label="app.name"
          @pointerdown="raiseWindow(app.id)"
        >
          <header class="window-titlebar" @pointerdown="startDrag($event, app)">
            <div class="title-identity">
              <span class="title-app-icon">{{ app.icon === 'terminal' ? '>_' : app.icon === 'folder' ? 'F' : app.icon === 'mail' ? '@' : app.icon === 'document' ? 'PDF' : app.icon === 'code' ? '</>' : 'KM' }}</span>
              <span>{{ app.name }}</span>
              <span class="title-divider">/</span>
              <span class="title-caption">KARINA MEIER</span>
            </div>
            <div class="window-controls">
              <button type="button" aria-label="Minimizar janela" title="Minimizar" @click.stop="minimizeApp(app.id)"><span class="minimize-symbol"></span></button>
              <button type="button" aria-label="Maximizar ou restaurar janela" title="Maximizar ou restaurar" @click.stop="toggleMaximize(app)"><span class="maximize-symbol"></span></button>
              <button class="close-control" type="button" aria-label="Fechar janela" title="Fechar" @click.stop="closeApp(app.id)"><span></span><span></span></button>
            </div>
          </header>

          <div v-if="app.id === 'about'" class="window-content about-content">
            <div class="app-intro">
              <div class="eyebrow">PERFIL DE ESTUDANTE <span>01 / 06</span></div>
              <h2>Tecnologia com curiosidade,<br><em>aprendizado com propósito.</em></h2>
              <p>Sou a Karina Meier, estudante de Técnico em Informática para Internet no SENAC, em Joinville — SC. Estou construindo minha trajetória por meio da formação técnica, projetos e experiências de aprendizagem.</p>
              <p>Busco uma oportunidade de estágio em tecnologia para aprofundar meus conhecimentos, colaborar com uma equipe e crescer na área de desenvolvimento.</p>
              <button class="text-action" type="button" @click="openApp('contact')">Vamos conversar <span>→</span></button>
            </div>
            <aside class="about-aside">
              <div class="profile-monogram">KARINA MEIER<span>.</span></div>
              <span class="aside-label">FORMAÇÃO ATUAL</span>
              <strong>Técnico em Informática<br>para Internet</strong>
              <span class="aside-school">SENAC · 2024 — 2026</span>
              <div class="aside-location"><span></span> JOINVILLE, SC</div>
            </aside>
            <div class="recognition-strip">
              <div><span class="recognition-index">01</span><span><b>Meninas na Tecnologia</b><small>UFSC / SENAC · participação em 2024 e 2026</small></span></div>
              <div><span class="recognition-index">02</span><span><b>3º lugar · 2024</b><small>Etapa final do projeto Meninas na Tecnologia</small></span></div>
              <div><span class="recognition-index">03</span><span><b>16ª JEDI</b><small>Join.Valle / Sebrae · 2026</small></span></div>
            </div>
          </div>

          <div v-else-if="app.id === 'projects'" class="window-content projects-content">
            <div class="content-heading">
              <div><div class="eyebrow">PORTFÓLIO ACADÊMICO <span>02 / 06</span></div><h2>Projetos em construção.</h2></div>
              <a class="small-outline-link" :href="`${github}?tab=repositories`" target="_blank" rel="noreferrer">Todos no GitHub <span>↗</span></a>
            </div>
            <p class="content-lede">Uma seleção de projetos listados no meu perfil. Acesse os repositórios para conferir as descrições e tecnologias publicadas.</p>
            <div class="project-grid">
              <article v-for="(project, index) in projects" :key="project.name" class="project-card">
                <div class="project-card-top"><span class="project-number">0{{ index + 1 }}</span><span class="project-group">{{ project.group }}</span></div>
                <h3>{{ project.name }}</h3>
                <p>{{ project.description }}</p>
                <div class="project-meta"><span>OBJETIVO</span><p>{{ project.objective }}</p></div>
                <div class="project-tech"><span>TECNOLOGIAS</span><b>{{ project.technologies }}</b></div>
                <a :href="project.repository ? `${github}/${project.repository}` : `${github}?tab=repositories`" target="_blank" rel="noreferrer" class="project-link">Ver no GitHub <span>↗</span></a>
              </article>
            </div>
            <p class="source-note">Descrições e detalhes técnicos devem ser conferidos nos respectivos repositórios do GitHub.</p>
          </div>

          <div v-else-if="app.id === 'skills'" class="window-content skills-content">
            <div class="content-heading"><div><div class="eyebrow">FERRAMENTAS DE APRENDIZADO <span>03 / 06</span></div><h2>Habilidades técnicas.</h2></div></div>
            <p class="content-lede">Conhecimentos desenvolvidos na formação e em projetos. Sem pontuações artificiais: cada habilidade faz parte de uma trajetória em andamento.</p>
            <div class="skills-layout">
              <article v-for="group in skillGroups" :key="group.title" class="skill-card">
                <span class="skill-marker"></span><h3>{{ group.title }}</h3>
                <div class="skill-tags"><span v-for="item in group.items" :key="item">{{ item }}</span></div>
              </article>
            </div>
            <div class="beyond-code"><div><span class="eyebrow">ALÉM DO CÓDIGO</span><h3>Aprender também é colaborar.</h3></div><p>Trabalho em equipe <i></i> Comunicação <i></i> Resolução de problemas</p></div>
          </div>

          <div v-else-if="app.id === 'resume'" class="window-content resume-content">
            <div class="resume-paper">
              <div class="resume-topline"><span>KARINA MEIER / CURRÍCULO</span><span>JOINVILLE — SC</span></div>
              <div class="resume-name"><div><span class="eyebrow">ESTUDANTE DE TECNOLOGIA</span><h2>Karina Meier</h2></div><span class="resume-initials">KM</span></div>
              <div class="resume-section"><h3>FORMAÇÃO</h3><div class="resume-entry"><b>Técnico em Informática para Internet</b><span>SENAC · 2024 — 2026</span><p>Previsão de conclusão: dezembro de 2026.</p></div></div>
              <div class="resume-section"><h3>CURSOS · SENAI</h3><ul><li v-for="course in courses" :key="course">{{ course }}</li></ul></div>
              <div class="resume-section"><h3>PROJETOS E PARTICIPAÇÕES</h3><ul><li>Meninas na Tecnologia — UFSC / SENAC; 3º lugar em 2024 e participação em 2026.</li><li>16ª JEDI — Join.Valle / Sebrae.</li><li>Projetos acadêmicos de tecnologia publicados no GitHub.</li></ul></div>
              <div class="resume-section objective-section"><h3>OBJETIVO</h3><p>Buscar estágio em Tecnologia da Informação para aplicar e desenvolver conhecimentos em programação, desenvolvimento de sistemas e resolução de problemas.</p></div>
              <div class="resume-contact">{{ email }} <span>·</span> Joinville — SC</div>
            </div>
            <button class="primary-action print-action" type="button" @click="printResume"><svg viewBox="0 0 24 24" aria-hidden="true"><path d="M7 8V3h10v5M7 17H5a2 2 0 0 1-2-2v-4a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2v4a2 2 0 0 1-2 2h-2M7 14h10v7H7z"/></svg> Imprimir / salvar currículo em PDF</button>
              <p class="print-note">Na janela de impressão, escolha “Salvar como PDF” para baixar o documento.</p>
            </div>

          <div v-else-if="app.id === 'contact'" class="window-content contact-content">
            <div class="contact-intro"><div class="eyebrow">CANAL ABERTO <span>05 / 06</span></div><h2>Uma boa conversa<br><em>pode começar aqui.</em></h2><p>Estou em busca de oportunidades de estágio em tecnologia e aberta a conexões acadêmicas.</p></div>
            <div class="contact-details">
              <a :href="`mailto:${email}`"><span class="contact-label">E-MAIL</span><b>{{ email }}</b><span class="detail-arrow">↗</span></a>
              <div><span class="contact-label">LOCALIZAÇÃO</span><b>Joinville — SC</b></div>
              <a :href="github" target="_blank" rel="noreferrer"><span class="contact-label">GITHUB</span><b>karinameier01</b><span class="detail-arrow">↗</span></a>
              <a :href="linkedIn" target="_blank" rel="noreferrer"><span class="contact-label">LINKEDIN</span><b>Karina Meier</b><span class="detail-arrow">↗</span></a>
            </div>
            <form class="message-form" @submit.prevent="sendMessage">
              <div class="form-heading"><h3>Envie uma mensagem</h3><span>DEMONSTRAÇÃO</span></div>
              <label>Seu nome<input v-model.trim="contactForm.name" name="name" autocomplete="name" required minlength="2" placeholder="Como podemos chamar você?"></label>
              <label>Seu e-mail<input v-model.trim="contactForm.email" name="email" type="email" autocomplete="email" required placeholder="voce@exemplo.com"></label>
              <label>Mensagem<textarea v-model.trim="contactForm.message" name="message" required minlength="10" rows="3" placeholder="Escreva sua mensagem..."></textarea></label>
              <button class="primary-action" type="submit">Validar mensagem <span>→</span></button>
              <p v-if="contactFeedback" class="form-feedback" role="status">{{ contactFeedback }}</p>
              <p class="form-disclaimer">O formulário valida os campos localmente, mas não envia nem armazena dados.</p>
            </form>
          </div>

          <div v-else class="window-content terminal-content">
            <div class="terminal-heading"><div class="eyebrow">AMBIENTE DE LINHA DE COMANDO <span>06 / 06</span></div><h2>Terminal KM</h2><p>Um pequeno atalho para explorar o portfólio.</p></div>
            <div ref="terminalOutput" class="terminal-screen" role="log" aria-live="polite">
              <div v-for="(line, index) in terminalLines" :key="index" :class="`terminal-line ${line.type}`"><span v-if="line.type === 'output'" class="output-prompt">›</span><span>{{ line.text }}</span></div>
              <form class="terminal-form" @submit.prevent="runCommand"><span>karina@km-os:~$</span><input v-model="terminalInput" aria-label="Comando do terminal" autocomplete="off" spellcheck="false"><button type="submit" aria-label="Executar comando">↵</button></form>
            </div>
            <div class="terminal-hints"><span>EXPERIMENTE</span><button v-for="command in terminalCommands" :key="command" type="button" @click="terminalInput = command; runCommand()">{{ command }}</button></div>
          </div>
        </section>
      </div>

      <footer class="taskbar">
        <button class="start-button" :class="{ selected: startMenuOpen }" type="button" @click.stop="startMenuOpen = !startMenuOpen">
          <span class="km-mark">KM</span><span class="start-text">Início</span>
        </button>
        <div class="taskbar-divider"></div>
        <nav class="taskbar-apps" aria-label="Aplicativos abertos na barra de tarefas">
          <button v-for="app in apps.filter((item) => item.open)" :key="app.id" class="taskbar-item" :class="{ active: activeAppId === app.id && !app.minimized }" type="button" @click="clickTaskbarApp(app)">
            <span class="taskbar-app-mark">{{ app.icon === 'terminal' ? '>_' : app.icon === 'folder' ? 'F' : app.icon === 'mail' ? '@' : app.icon === 'document' ? 'PDF' : app.icon === 'code' ? '</>' : 'KM' }}</span><span>{{ app.shortName }}</span>
          </button>
        </nav>
        <div class="system-tray"><span class="network-status" aria-label="Rede conectada"><i></i><i></i><i></i></span><span class="battery-status" aria-label="Bateria simulada"><i></i></span><div class="tray-clock"><b>{{ currentTime }}</b><span>{{ currentDate }}</span></div></div>
      </footer>

      <aside v-if="startMenuOpen" class="start-menu" aria-label="Menu Início" @pointerdown.stop>
        <div class="menu-brand"><span class="km-mark">KM</span><div><b>KM</b><small>Karina Meier · 2.6</small></div></div>
        <div class="menu-profile"><span>PERFIL DO SISTEMA</span><b>Estudante de tecnologia</b><small>Joinville — SC · 2024–2026</small></div>
        <div class="menu-app-list"><span class="menu-section-label">APLICATIVOS</span><button v-for="app in apps" :key="app.id" type="button" @click="openApp(app.id)"><span class="menu-icon">{{ app.icon === 'terminal' ? '>_' : app.icon === 'folder' ? 'F' : app.icon === 'mail' ? '@' : app.icon === 'document' ? 'PDF' : app.icon === 'code' ? '</>' : 'KM' }}</span>{{ app.name }}<span class="menu-open">Abrir</span></button></div>
        <div class="menu-social"><a :href="github" target="_blank" rel="noreferrer">GitHub ↗</a><a :href="linkedIn" target="_blank" rel="noreferrer">LinkedIn ↗</a><a :href="`mailto:${email}`">E-mail ↗</a></div>
      </aside>
      <div class="desktop-status"><i></i><span>ESTUDANTE DE TECNOLOGIA</span></div>
    </main>
  </div>
</template>
