<h1 align="center">Portfolio — Desarrollo con IA</h1>
<p align="center">Pedro Pablo Cubells Talavera · <a href="https://github.com/Acedpol">@Acedpol</a> · Madrid</p>

<blockquote>
<b>🇬🇧 TL;DR for English-speaking readers:</b> this repo indexes a set of independent, real (not tutorial) full-stack projects built with AI-assisted development — each with its own GitHub repo, CI pipeline, and issue→branch→PR workflow. Project descriptions below are in Spanish (matching the actual repo history), but stack tables, links, and code are language-agnostic. Start with the <a href="#-tabla-resumen--summary-table">summary table</a>.
</blockquote>

Este repo es el punto de entrada al portfolio: reúne y explica, en un solo sitio, proyectos que en GitHub viven como repos independientes (cada uno con su propio historial de commits, CI y README técnico). El objetivo es que cualquiera — en particular alguien de RRHH sin tiempo para bucear en 6 repos — pueda ver de un vistazo qué se ha construido, con qué stack, y cómo se trabajó.

---

## 📋 Tabla resumen / Summary table

| # | Proyecto | Foco | Stack clave | Estado |
|---|---|---|---|---|
| 1 | [`expense-api`](https://github.com/Acedpol/expense-api) | Backend REST | FastAPI · SQLAlchemy 2.0 · JWT · PostgreSQL · Docker | ✅ v1.1.0 |
| 2 | [`expense-tracker-ui`](https://github.com/Acedpol/expense-tracker-ui) | Frontend SPA | React 19 · TypeScript · Vite · TanStack Query | ✅ v1.0.0 |
| 3a | [`personal-rag-assistant`](https://github.com/Acedpol/personal-rag-assistant) | IA aplicada — RAG (backend) | FastAPI · ChromaDB · sentence-transformers · Claude API | ✅ v1.0.0 |
| 3b | [`personal-rag-assistant-ui`](https://github.com/Acedpol/personal-rag-assistant-ui) | IA aplicada — RAG (frontend) | React 19 · TypeScript · Vite | ✅ v1.0.0 |
| 4 | Agente con herramientas | IA — agentes | *por definir* | ⏳ Pendiente |
| 5 | Capstone full-stack con IA | Integración de todo lo anterior | *por definir* | ⏳ Pendiente |

Los proyectos 1↔2 y 3a↔3b están relacionados solo por red (HTTP): el frontend consume la API del backend correspondiente, nunca comparten código ni carpeta.

<br/>

## 1️⃣2️⃣ `expense-api` + `expense-tracker-ui` — Gestión de gastos personales

Aplicación full-stack de control de gastos: API REST con autenticación, y una SPA que la consume.

**Backend — [`expense-api`](https://github.com/Acedpol/expense-api)** ([release v1.1.0](https://github.com/Acedpol/expense-api/releases/tag/v1.1.0))
- FastAPI · SQLAlchemy 2.0 · Alembic (migraciones) · Pydantic v2
- Autenticación JWT (python-jose + passlib/bcrypt), rate limiting con slowapi
- pytest + pytest-cov, ruff + pre-commit, Docker/docker-compose, CI en GitHub Actions
- CRUD completo de gastos con filtros, categorías y usuarios aislados por token

**Frontend — [`expense-tracker-ui`](https://github.com/Acedpol/expense-tracker-ui)** ([release v1.0.0](https://github.com/Acedpol/expense-tracker-ui/releases/tag/v1.0.0))
- React 19 + TypeScript + Vite, Tailwind CSS 4, React Router
- TanStack Query (estado de servidor) + React Hook Form + Zod (validación)
- Cliente API **tipado end-to-end**: `openapi-typescript` + `openapi-fetch` generan el cliente directamente desde el OpenAPI real del backend — si la API cambia, el frontend deja de compilar hasta adaptarse, en vez de romperse en runtime
- Vitest + React Testing Library + MSW, CI en GitHub Actions

<sub>Un bug real de backend (`ExpenseUpdate.date` que solo aceptaba `null`) apareció construyendo el frontend y se arregló en `expense-api`, con su propio test de regresión — no con un parche en el consumidor.</sub>

<br/>

## 3️⃣ `personal-rag-assistant` + `personal-rag-assistant-ui` — Asistente RAG sobre documentos propios

Sistema de *retrieval-augmented generation*: subes documentos propios (PDF), se trocean e indexan, y puedes preguntar sobre ellos con respuestas citando la fuente real.

**Backend — [`personal-rag-assistant`](https://github.com/Acedpol/personal-rag-assistant)** ([release v1.0.0](https://github.com/Acedpol/personal-rag-assistant/releases/tag/v1.0.0))
- FastAPI · SQLAlchemy (sin Alembic — decisión deliberada, una sola tabla de metadata)
- Pipeline de ingesta: `pypdf` (extracción) → `langchain-text-splitters` (chunking) → `sentence-transformers` (embeddings **locales**, sin API key) → `chromadb` (vector store, métrica coseno)
- Generación con `anthropic` (Claude API) — proveedor **Mock o real seleccionado automáticamente** según haya o no `ANTHROPIC_API_KEY`, para poder desarrollar y testear sin gastar tokens
- Sin autenticación (decisión deliberada y confirmada: herramienta de una sola colección compartida, el foco es la mecánica de RAG, no un sistema multiusuario)
- pytest, ruff, CI en GitHub Actions

**Frontend — [`personal-rag-assistant-ui`](https://github.com/Acedpol/personal-rag-assistant-ui)** ([release v1.0.0](https://github.com/Acedpol/personal-rag-assistant-ui/releases/tag/v1.0.0))
- React 19 + TypeScript + Vite, Tailwind CSS 4, TanStack Query
- Dos pantallas: **Documentos** (subir / listar / borrar / ver chunks) y **Preguntar** (pregunta → respuesta con fuentes citadas y badge de proveedor mock/real)
- Mismo patrón de cliente tipado (`openapi-typescript` + `openapi-fetch`) que en el proyecto 1-2

<br/>

## ⏳ Próximos pasos / What's next

| # | Proyecto | Foco |
|---|---|---|
| 4 | Agente con herramientas | Un LLM que decide y ejecuta acciones sobre herramientas externas (tool use), no solo responde texto |
| 5 | Capstone full-stack con IA | Integra los aprendizajes de los proyectos 1-4 en un producto único |

<br/>

## 🧭 Metodología / How I work

No son ejercicios sueltos: hay una forma de trabajar consistente entre proyectos, pensada para parecerse a cómo se construye software en un equipo real.

- **Flujo issue → rama → PR → CI verificado → squash-merge.** Cada feature se planifica como issue, se desarrolla en su rama, y solo se mergea a `main` cuando el pipeline de CI (tests + lint) pasa en verde en GitHub — verificado en el repo remoto, nunca solo en local.
- **Commits atómicos y trazables.** Formato [Conventional Commits](https://www.conventionalcommits.org/) (`feat`, `fix`, `refactor`...) en cada mensaje, un paso lógico por commit — nunca varios cambios agrupados en un commit gigante. El cuerpo del commit conserva el detalle técnico (causa del bug, cómo se verificó), no solo un titular.
- **CI desde el primer commit, no al final.** El pipeline de tests + lint se monta justo después del scaffold inicial de cada proyecto, antes de construir más funcionalidad — para que `main` esté protegida desde el principio, no como tarea de limpieza al cierre.
- **Verificación real, no "debería funcionar".** Cada feature se prueba contra el servidor real corriendo (navegador: capturas, red, consola) o con tests que golpean código real, no solo mocks que puedan enmascarar el bug.
- **Root cause, no parches.** Cuando algo falla — un bug de backend, un fallo de CI intermitente, una librería con comportamiento raro — se diagnostica la causa real antes de tocar nada, incluyendo leer código fuente de dependencias cuando hace falta.
- **Bugs cruzados se arreglan en su sitio.** Si construyendo el frontend aparece un bug real del backend, se arregla en el repo del backend con su propio test de regresión, nunca con un parche en el consumidor (ver ejemplo en `expense-api` arriba).

Todo el desarrollo está asistido por IA (Claude Code) bajo mi dirección — decisiones de arquitectura, revisión y control de calidad son míos. El método exacto de orquestación es parte del trabajo en curso, no del contenido público de este repo.

<br/>

## 🛠️ Habilidades técnicas / Technical skills

| Área | Tecnologías |
|---|---|
| **Backend** | Python, FastAPI, SQLAlchemy 2.0, Alembic, Pydantic v2, JWT (python-jose, passlib/bcrypt), slowapi |
| **Frontend** | React 19, TypeScript, Vite, Tailwind CSS 4, React Router, TanStack Query, React Hook Form, Zod |
| **IA aplicada** | Claude API (Anthropic), RAG (ChromaDB, sentence-transformers, langchain-text-splitters), diseño de prompts, selección automática mock/real provider |
| **Integración API** | OpenAPI, `openapi-typescript` + `openapi-fetch` (cliente tipado end-to-end generado desde el backend real) |
| **Testing** | pytest + pytest-cov, Vitest, React Testing Library, MSW (mocking de red) |
| **DevOps / CI** | Docker, docker-compose, GitHub Actions, ruff, pre-commit, commitlint + husky |
| **Control de versiones** | Git, flujo issue → rama → PR → squash-merge, Conventional Commits |

<br/>

<p align="center"><sub>Índice del portfolio — cada proyecto es un repo independiente con su propio README técnico y CI. Ver también el <a href="https://github.com/Acedpol">perfil de GitHub</a>.</sub></p>
