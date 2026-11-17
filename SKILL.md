---
name: job-match
description: Procura vagas de emprego e ordena-as pelo match com o CV do utilizador — numa empresa específica (ex. "vagas da Aubay") ou no mercado em geral. Lê o CV da pasta onde a sessão começou; se não houver, pede o CV ou lê o perfil do LinkedIn. Lista 10 vagas por omissão (ou o número pedido) com local, regime, match, porquê, lacunas e link. Use quando o utilizador pede vagas, oportunidades, "jobs que combinam comigo", "top N vagas na empresa X", ou o match do perfil com vagas.
---

# Job Match

Encontra as vagas abertas que mais combinam com o perfil do utilizador e lista-as por ordem de match.
A skill **só pesquisa e lista**. Candidatar, criar CVs à medida ou preencher formulários são passos
seguintes, que só acontecem quando o utilizador os pede.

## Entradas

Tira do pedido do utilizador (pergunta só o que for indispensável):

| Entrada | Por omissão |
|---|---|
| Empresa (ex. Aubay) | nenhuma: pesquisa o mercado em geral |
| Número de vagas a listar | **10** |
| Local / país / remoto | o local do CV; se não houver, o país do utilizador |
| Área ou cargo | o que o CV mostra como foco principal |

## Passo 1 — Perfil do utilizador

1. **Procura o CV na pasta onde a sessão começou** (o diretório de trabalho principal), sem descer a
   `node_modules`, `.git`, `.next` ou pastas de build:
   - ficheiros `*.pdf`, `*.docx`, `*.html`, `*.md` com `cv`, `resume`, `curriculo`, `currículo` ou o nome
     da pessoa no nome do ficheiro;
   - dados de perfil em código (ex. `profile.ts`, `profile.json`, `resume.json`).
2. **Se houver vários**, escolhe o mais recente e mais completo; usa os outros só para completar
   (ex. um CV focado em Node.js e outro em .NET juntos dão o perfil todo). Diz em uma linha que
   ficheiros usaste.
3. **Se não houver nenhum**, pergunta ao utilizador (AskUserQuestion) e não avances sem resposta:
   - "Indicar o caminho do CV", ou
   - "Ler o meu perfil do LinkedIn": abre o perfil no browser do utilizador (Claude in Chrome) em
     `linkedin.com/in/<perfil>/` e lê as secções de experiência, formação, competências e línguas.
     Se o URL do perfil não for conhecido, pede-o.
4. **Monta um resumo do perfil apenas com factos do CV**: cargo e senioridade, anos de experiência,
   stack com anos quando indicados, liderança e arquitetura, domínios, línguas e nível, local e regime
   pretendido. **Nunca inventes competências.** O que não está no CV é lacuna, não match.

## Passo 2 — Fontes de vagas

Usa as melhores fontes para o pedido, por esta ordem, e verifica sempre a vaga na fonte original.

**Com empresa indicada:**
1. **Página de carreiras da empresa**: é a fonte de verdade. Encontra o URL real a partir da navegação
   do site oficial (o link "Carreiras/Careers"); não adivinhes caminhos.
2. LinkedIn Jobs filtrado pela empresa.
3. Agregadores (Glassdoor, Indeed, ITJobs, Talent.com, beBee) só para descobrir vagas que faltem;
   confirma cada uma na página da empresa, porque os agregadores mostram vagas fechadas.

**Sem empresa (mercado em geral), por região:**
- **Portugal:** ITJobs.pt, LinkedIn Jobs, Landing.jobs, Net-Empregos, Indeed PT, Glassdoor PT, Teamlyzer
  (para cultura e salários).
- **Brasil:** LinkedIn Jobs, Gupy, Programathor, Indeed BR, Glassdoor BR, Catho.
- **Remoto internacional:** LinkedIn Jobs (remoto), We Work Remotely, Remote OK, Wellfound, Otta.
- **Outros países:** LinkedIn Jobs e o Indeed do país.

Faz as pesquisas independentes em paralelo (WebSearch, várias queries no mesmo turno). Para ler
páginas usa WebFetch ou o agente `web-fetch`; para sites que só carregam com JavaScript ou exigem
sessão (LinkedIn), usa o browser do utilizador.

### Técnicas que funcionam em páginas de carreiras

- **Muitas páginas carregam as vagas por JavaScript.** Se a extração de texto só mostrar algumas vagas
  ou um botão "Ver mais", lê os dados da página em vez de clicar:
  - Next.js: `document.getElementById('__NEXT_DATA__')`, procurando um array `jobs` (ou parecido);
  - procura em `performance.getEntriesByType('resource')` por endpoints `/api/...` (ex. `/api/job?id=`)
    e chama-os com `fetch` a partir da própria página para obter a descrição completa;
  - portais comuns: Teamtailor, SmartRecruiters, Recruitee, Workable, Greenhouse, Lever (muitos têm
    feed JSON público).
- **Recusa os cookies não essenciais** quando aparece o banner.
- **Guarda o link direto de cada vaga** (ex. `.../carreira?o=<id>`), não só o da listagem.
- **Campos vazios não são dados.** Se o regime (híbrido/remoto/presencial) ou o local estiverem em
  branco, escreve "não indicado"; não deduzas de outras vagas.

## Passo 3 — Avaliar o match

1. Faz uma primeira triagem por título e tecnologias, para ficar com o dobro do número a listar.
2. **Lê a descrição completa** de cada vaga da triagem (requisitos obrigatórios, anos, línguas,
   local, regime).
3. Pontua cada vaga:

| Critério | Peso |
|---|---|
| Tecnologias obrigatórias que o CV tem | alto |
| Senioridade e anos pedidos vs. anos reais | alto |
| Tipo de cargo (dev, tech lead, arquiteto) vs. o percurso | médio |
| Local e regime vs. onde o utilizador vive ou quer trabalhar | médio |
| Tecnologias "nice to have" | baixo |

4. **Exclui**, e diz que excluíste, as vagas com bloqueio: língua obrigatória que o utilizador não
   fala, tecnologia principal ausente do CV, áreas sem relação (ex. SAP funcional para um dev),
   vagas claramente abaixo do nível.
5. Converte em estrelas: ⭐⭐⭐⭐⭐ (quase o CV inteiro) até ⭐ (encaixe fraco). Usa meias estrelas.

## Passo 4 — Lista final

Responde na língua do utilizador. Lista **N vagas** (10 por omissão), da melhor para a pior:

| # | Vaga | Local · Regime | Match | Porquê |
|---|---|---|---|---|

- **Porquê**: o que encaixa (concreto, com tecnologias) e as **lacunas** do CV face à vaga.
- **Links**: o link direto de cada vaga (na tabela ou numa lista logo a seguir).
- Se houver menos vagas boas do que N, lista só as que fazem sentido e diz porquê.

Depois da tabela:
- **Recomendação**: as 1 a 3 vagas a que vale a pena candidatar já, e que CV usar ou adaptar.
- **Excluídas**: uma linha com os tipos de vaga que ficaram de fora e porquê.
- **Avisos**: datas antigas, regime por confirmar, vagas vistas só em agregadores.
- **Fontes**: lista em markdown dos URLs usados.

## Regras

- Só factos do CV. Uma competência que a vaga pede e o CV não mostra é uma lacuna, mesmo que pareça
  provável que o utilizador a tenha. Se quiseres saber, pergunta.
- Não te candidates a nada, não preenchas formulários e não envies mensagens sem pedido explícito.
- Se for pedido um CV à medida de uma vaga, cria-o só com factos dos CVs existentes e mostra as
  lacunas que deixaste de fora, para o utilizador confirmar se tem essa experiência.
- Não guardes dados pessoais do utilizador (telefone, e-mail, morada) em ficheiros desta skill.
