---
description: "Audit and modernize .github/ Copilot agent customization in any repo. Migrates legacy persona files to custom agents, file instructions, skills, and creates portable agent context (AGENTS.md + docs/AGENT_CONTEXT/) that survives machine changes."
name: "Modernize Copilot Agent Customization"
argument-hint: "Optional: focus area (e.g. 'tylko agents' or 'pomiń skills')"
---

# Zadanie: Modernizacja konfiguracji GitHub Copilot Agent

Jesteś ekspertem od customizacji VS Code Copilot Agent (v0.43+). Twoim zadaniem jest **audit i reorganizacja katalogu `.github/`** oraz konfiguracji agenta w tym repozytorium, tak aby maksymalnie wykorzystać nowoczesne prymitywy Copilota: **Custom Agents, File Instructions, Skills, Portable Agent Context**.

## ⚠️ Najważniejsza zasada: PORTABILITY

Repo memory Copilota (`/memories/repo/`) jest **lokalna na maszynie** i **NIE jest commitowana do gita**. Po zmianie komputera lub `git clone` na innej maszynie agent traci cały kontekst.

**Rozwiązanie:** Źródłem prawdy MUSI być git:
- `AGENTS.md` w root (open standard — czyta Copilot, Cursor, Claude Code, Aider)
- `docs/AGENT_CONTEXT/` z 4 plikami (project-overview, conventions, build-and-test, gotchas)
- `/memories/repo/` używać tylko jako tymczasowy scratchpad bieżącej sesji

## Kontekst — nowoczesne prymitywy Copilota

| Prymityw | Lokalizacja | W git? | Cel |
|---|---|---|---|
| **AGENTS.md** | root repo | ✅ | Open standard, quick start dla wszystkich agentów AI |
| **docs/AGENT_CONTEXT/** | root repo | ✅ | **ŹRÓDŁO PRAWDY** kontekstu projektu |
| Workspace Instructions | `.github/copilot-instructions.md` | ✅ | Krótkie reguły always-on (≤50 linii), linkuje do `docs/` |
| File Instructions | `.github/instructions/*.instructions.md` | ✅ | Auto-load przez `applyTo` glob |
| Custom Agents | `.github/agents/*.agent.md` | ✅ | Persony z ograniczonym `tools: [...]` |
| Skills | `.github/skills/<name>/SKILL.md` | ✅ | Slash commands `/`, workflow procedury |
| Prompts | `.github/prompts/*.prompt.md` | ✅ | Pojedyncze parametryzowane zadania |
| Hooks | `.github/hooks/*.json` | ✅ | Deterministyczne lifecycle events |
| Repo Memory | `/memories/repo/*.md` | ❌ | **Tylko lokalny cache/scratchpad** |

Każdy plik MUSI mieć YAML frontmatter z `description` (kluczowe dla discovery!). `applyTo: "**"` przepala kontekst — używaj selektywnie.

## Faza 0 — Audit (TYLKO READ)
Zanim cokolwiek zmienisz:
1. Wylistuj wszystkie pliki w `.github/` rekursywnie
2. Sprawdź czy istnieje `AGENTS.md` w root i `docs/AGENT_CONTEXT/`
3. Przeczytaj `.github/copilot-instructions.md`, wszystkie luźne `*.md` w `.github/`
4. Sprawdź `package.json` / `pyproject.toml` / inny manifest — stack, scripts, testy
5. Sprawdź `README.md` i `docs/` — domena, architektura
6. Sprawdź `/memories/repo/` — co jest w lokalnym cache
7. **Wylistuj problemy:** monolityczne pliki, luźne `ROLE_*.md`, brak `AGENTS.md`, brak `docs/AGENT_CONTEXT/`, repo memory traktowane jako źródło prawdy

## Faza 1 — Portable Agent Context (NAJWAŻNIEJSZE — działa na każdej maszynie)
Utworzyć w **git**:

### `docs/AGENT_CONTEXT/` — źródło prawdy
- **`project-overview.md`** — stack, framework versions, struktura katalogów, domena, kluczowe pliki, roadmap
- **`build-and-test.md`** — komendy build/test/lint, pre-commit checklist, wymagania CI/CD, debug tips
- **`conventions.md`** — code style, naming, krytyczne reguły frameworka, git workflow, język (locale)
- **`gotchas.md`** — znane pułapki, common mistakes, edge cases środowiska/CI

Każdy plik 30-80 linii, konkretne fakty, linki do `docs/`/`README`.

### `AGENTS.md` w root — open standard
Krótki (40-60 linii) wstęp:
- Quick Start dla agenta (4 linki do `docs/AGENT_CONTEXT/`)
- TL;DR krytycznych reguł
- Project at a glance (stack jednolinijka)
- Build & test (najważniejsze komendy)
- Linki do konfiguracji `.github/agents/`, `.github/instructions/`, `.github/skills/`
- **Ważna sekcja:** "Memory vs Repo — `docs/AGENT_CONTEXT/` ma pierwszeństwo nad lokalną repo memory"

## Faza 2 — Custom Agents
Jeśli repo ma role/persony rozproszone po luźnych `*.md` (np. `ROLE_*.md`), zmigrować do `.github/agents/<name>.agent.md`:
```yaml
---
description: "Use when: <trigger keywords for auto-routing>"
name: "<Persona Name>"
tools: [read, edit, search, execute]   # minimalny zestaw, nie wszystkie
model: ['Claude Sonnet 4.5 (copilot)', 'GPT-5 (copilot)']
---
```
Tools per rola:
- Frontend/Backend dev: `[read, edit, search, execute]`
- Data steward: `[read, edit, search]` (no shell)
- Project manager / reviewer: `[read, search]` (read-only)

W body: Constraints (DO NOT...), Approach (kroki), Style, Output. **Body powinno linkować do `docs/AGENT_CONTEXT/conventions.md` itp. — nie duplikować treści.** **Po migracji USUŃ luźne `ROLE_*.md`.**

## Faza 3 — Odchudzenie copilot-instructions.md
Cel: ≤50 linii. Zostaw tylko:
- 1-liner stack
- 3-5 KRYTYCZNYCH reguł (forbidden patterns)
- Język/locale
- **Linki do `AGENTS.md` i `docs/AGENT_CONTEXT/*`** (NIE inline'uj treści)
- Linki do `.github/agents/`, `.github/skills/`, `.github/prompts/`
- Krótka notka: "repo memory = lokalny cache, źródło prawdy = `docs/AGENT_CONTEXT/`"

Wszystko inne → przenieś do `docs/AGENT_CONTEXT/` (portable!) lub `.github/instructions/` (applyTo).

## Faza 4 — File Instructions z `applyTo`
Utwórz w `.github/instructions/` osobne pliki dla różnych obszarów:
- `<język>.instructions.md` — `applyTo: ["**/*.ts", "**/*.tsx"]` (lub `**/*.py`, etc.)
- `<framework>.instructions.md` — np. dla Prisma, Django ORM, etc.
- `api-routes.instructions.md` — `applyTo: "app/api/**"` lub odpowiednik
- `ci-cd.instructions.md` — `applyTo: ".github/workflows/**"`

Każdy plik ma JEDEN concern + `description` z trigger keywords. Może linkować do `docs/AGENT_CONTEXT/conventions.md` dla głębszego kontekstu.

## Faza 5 — Skills (slash commands)
Zidentyfikuj **powtarzalne workflow'y** w repo (pre-commit, deploy, data refresh, etc.) i utwórz `.github/skills/<name>/SKILL.md`:
```yaml
---
name: pre-commit
description: "Use when: ... (trigger keywords). Step-by-step procedure for ..."
---
```
Body: When to Use, Step 1...N (z konkretnymi komendami shell), Decision Rules. Skille pojawiają się jako `/<name>` w chacie.

## Faza 6 — Repo memory jako pointer
W `/memories/repo/` utwórz `README.md` który:
- Wskazuje że źródło prawdy to `docs/AGENT_CONTEXT/`
- Definiuje workflow: tymczasowe notatki → po weryfikacji przenieś do `docs/`
- Zachowuje istniejące wpisy które są legitymnie sesyjne (debug notes, weryfikacje)

## Faza 7 — Sprzątanie i commit
1. Usuń wszystkie obsolete pliki (zmigrowane do agents/instructions/docs)
2. Zachowaj backup `.bak` do weryfikacji (lokalnie, nie commituj)
3. **Commity rozdziel:**
   - `chore(copilot): modernize agent customization` — `.github/` (agents, instructions, skills)
   - `docs(agents): add portable agent context (AGENTS.md + docs/AGENT_CONTEXT)` — pliki w git jako źródło prawdy
4. **NIE pushuj** bez potwierdzenia użytkownika (jeśli ArgoCD/auto-deploy)

## Wymagania jakościowe
- **Portability first:** każdy istotny fakt o projekcie MUSI być w git (`docs/AGENT_CONTEXT/` lub `AGENTS.md`)
- **Frontmatter:** każdy plik z `description` (keyword-rich, "Use when: ..." pattern)
- **Brak duplikatów** treści — `docs/AGENT_CONTEXT/` to single source, reszta linkuje
- **Linkowanie zamiast inlinowania**
- **Język:** zachowaj locale projektu (sprawdź existing files)
- **Minimal tools** w agentach — nie dawaj wszystkiego "na zapas"
- **Testuj `applyTo`** — `"**"` tylko gdy reguła naprawdę dotyczy wszystkiego

## Plan akcji
Zacznij od **Fazy 0 (audit)** i pokaż mi raport z propozycjami ZANIM zaczniesz tworzyć/usuwać pliki. Po akceptacji — wykonuj fazy 1→7 sekwencyjnie, z todo listą i krótkimi raportami statusu.

## Test scenariusza "nowy komputer"
Po zakończeniu modernizacji zweryfikuj mentalnie:
1. `git clone repo` na świeżej maszynie
2. Otwórz w VS Code / Cursor / Claude Code
3. Czy agent ma pełen kontekst z samego git? **Jeśli wymaga lokalnej memory — pivot, przenieś treść do `docs/AGENT_CONTEXT/`**

## Anti-patterns których unikaj
- ❌ **Repo memory jako źródło prawdy** — ginie przy zmianie kompa
- ❌ Monolityczny `copilot-instructions.md` >100 linii
- ❌ Luźne `ROLE_*.md` które agent musi ręcznie czytać
- ❌ `applyTo: "**"` z treścią relevantną tylko dla części plików
- ❌ Vague descriptions: "A helpful agent" — agent nie znajdzie
- ❌ Swiss-army agents z `tools: [<wszystko>]`
- ❌ Duplikowanie treści — jeden fakt powinien być w jednym miejscu (`docs/AGENT_CONTEXT/`), reszta linkuje
- ❌ Tworzenie hooks gdy wystarczą instrukcje (hooks tylko dla DETERMINISTYCZNEGO wymuszenia)
- ❌ Brak `AGENTS.md` w root — bez tego agenci spoza VS Code (Cursor, Aider) nie znajdą kontekstu
