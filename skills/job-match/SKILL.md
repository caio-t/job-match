---
name: job-match
description: Procura vagas de emprego e ordena-as pelo match com o CV do utilizador — numa empresa específica (ex. "vagas da Aubay") ou no mercado em geral. Lê o CV da pasta onde a sessão começou; se não houver, pede o CV ou o PDF do perfil do LinkedIn. Lista 10 vagas por omissão (ou o número pedido) com local, regime, match, porquê, lacunas e link. A pedido, cria um CV em PDF à medida de uma vaga, só com factos do CV. Use quando o utilizador pede vagas, oportunidades, "jobs que combinam comigo", "top N vagas na empresa X", o match do perfil com vagas, ou "faz o CV para a vaga X".
---

# Job Match

Encontra as vagas abertas que mais combinam com o perfil do utilizador e lista-as por ordem de match.
Por omissão a skill **só pesquisa e lista** (passos 1 a 4). O CV à medida (passo 5) só acontece quando
o utilizador o pede. Candidatar e preencher formulários ficam fora da skill e exigem pedido explícito.

## Onde corre

A skill funciona no Claude Code e na app do Claude (claude.ai, desktop, telemóvel). Na app:
- não há pasta de trabalho: usa o CV **anexado à conversa**; se não houver, pede-o;
- onde o texto fala em AskUserQuestion, faz a pergunta no chat;
- se uma página não se lê com as ferramentas web, dá o link e pede ao utilizador que cole o texto;
- os ficheiros gerados (CV, carta) entregam-se para **descarregar**, não se gravam numa pasta.

## Entradas

Tira do pedido do utilizador (pergunta só o que for indispensável):

| Entrada | Por omissão |
|---|---|
| Empresa (ex. Aubay) | nenhuma: pesquisa o mercado em geral |
| Número de vagas a listar | **10** |
| Local / país / remoto | o local do CV; se não houver, o país do utilizador |
| Área ou cargo | o que o CV mostra como foco principal |

## Passo 1 — Perfil do utilizador

1. **Procura o CV na pasta onde a sessão começou** (na app do Claude: nos ficheiros anexados) (o diretório de trabalho principal), sem descer a
   `node_modules`, `.git`, `.next` ou pastas de build:
   - ficheiros `*.pdf`, `*.docx`, `*.html`, `*.md` com `cv`, `resume`, `curriculo`, `currículo` ou o nome
     da pessoa no nome do ficheiro;
   - dados de perfil em código (ex. `profile.ts`, `profile.json`, `resume.json`).
2. **Se houver vários**, escolhe o mais recente e mais completo; usa os outros só para completar
   (ex. um CV focado em Node.js e outro em .NET juntos dão o perfil todo). Diz em uma linha que
   ficheiros usaste.
3. **Se não houver nenhum**, pergunta ao utilizador (AskUserQuestion) e não avances sem resposta:
   - "Indicar o caminho do CV", ou
   - "Usar o meu perfil do LinkedIn": pede ao utilizador que exporte o perfil em PDF (no perfil,
     **Mais → Guardar em PDF**) e o anexe, ou que cole o texto. Não abras nem leias o LinkedIn por
     conta própria: os termos do LinkedIn proíbem o acesso automatizado.
4. **Monta um resumo do perfil apenas com factos do CV**: cargo e senioridade, anos de experiência,
   stack com anos quando indicados, liderança e arquitetura, domínios, línguas e nível, local e regime
   pretendido. **Nunca inventes competências.** O que não está no CV é lacuna, não match.

## Passo 2 — Fontes de vagas

Usa as melhores fontes para o pedido, por esta ordem, e verifica sempre a vaga na fonte original.

**Com empresa indicada:**
1. **Página de carreiras da empresa**: é a fonte de verdade. Encontra o URL real a partir da navegação
   do site oficial (o link "Carreiras/Careers"); não adivinhes caminhos.
2. O feed público do portal de recrutamento da empresa, se existir (ver abaixo).
3. Agregadores e LinkedIn Jobs só através dos resultados de pesquisa, para descobrir vagas que
   faltem; confirma cada uma na página da empresa, porque os agregadores mostram vagas fechadas.

**Sem empresa (mercado em geral), por região:**
- **Portugal:** ITJobs.pt, LinkedIn Jobs, Landing.jobs, Net-Empregos, Indeed PT, Glassdoor PT, Teamlyzer
  (para cultura e salários).
- **Brasil:** LinkedIn Jobs, Gupy, Programathor, Indeed BR, Glassdoor BR, Catho.
- **Remoto internacional:** LinkedIn Jobs (remoto), We Work Remotely, Remote OK, Wellfound, Otta.
- **Outros países:** LinkedIn Jobs e o Indeed do país.

Faz as pesquisas independentes em paralelo (WebSearch, várias queries no mesmo turno). Para ler
páginas usa as ferramentas de leitura web (WebFetch ou equivalente).

**LinkedIn, Indeed e Glassdoor** proíbem nos termos de uso a recolha automatizada. Destes sites usa
só o que vem nos resultados de pesquisa (título, empresa, link); não abras, não percorras listagens
e não entres com a sessão do utilizador. Para a descrição completa, procura a mesma vaga na página
da empresa ou dá o link ao utilizador.

### Ler páginas de carreiras

- **Feeds públicos dos portais de recrutamento.** Muitas empresas publicam as vagas num portal
  (Greenhouse, Lever, Workable, Recruitee, SmartRecruiters, Teamtailor) que tem um feed ou API
  pública documentada para listar vagas. Prefere esse feed à página quando existir.
- **Se a página carregar as vagas por JavaScript** e o texto não aparecer, não leias dados internos
  da página nem chames endpoints não documentados: usa o feed público do portal ou dá o link ao
  utilizador.
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

## Passo 5 — CV à medida (só a pedido)

Corre quando o utilizador pede um CV para uma vaga (ex. "faz o CV para a #1", "CV para a vaga de
Tech Lead"). Se a vaga não estiver na lista desta sessão, pede o link e lê a descrição completa primeiro.

1. **Cruza a vaga com o perfil do passo 1.** Separa:
   - *encaixes*: requisitos da vaga que o CV prova (com anos, quando o CV os dá);
   - *lacunas*: requisitos que o CV não mostra.
2. **Pergunta pelas lacunas importantes antes de escrever** (AskUserQuestion, até 4 de cada vez):
   "A vaga pede X. Tens experiência? Quantos anos?". Só o que o utilizador confirmar entra no CV,
   com as palavras e os anos que ele der. Sem resposta, fica de fora.
3. **Escreve o CV no idioma da vaga** (vaga em inglês → CV em inglês), a partir de
   [templates/cv-template.md](templates/cv-template.md) (o HTML do modelo está no bloco de código):
   - **Título**: o cargo da vaga + as 3 ou 4 tecnologias principais que o utilizador tem.
   - **Resumo** (3 a 4 linhas): anos de experiência, foco atual e as tecnologias pedidas que tem.
   - **Pontos fortes para a vaga**: 4 a 5 bullets, cada um ligado a um requisito da vaga.
   - **Experiência**: todos os empregos do CV, sem cortar períodos; nos relevantes, põe primeiro os
     bullets e projetos com a stack da vaga. Não mudes cargos, empresas nem datas.
   - **Competências**: agrupadas, com as pedidas pela vaga primeiro; mantém os anos do CV.
   - **Formação e línguas**: exatamente como no CV.
   - Cabe em 2 páginas A4.
4. **Gera o PDF** com o Chrome/Edge em modo headless, a partir do HTML:
   - Windows: `"C:/Program Files/Google/Chrome/Application/chrome.exe"`
   - macOS: `"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"`
   - Linux: `google-chrome` ou `chromium`

   ```bash
   "<chrome>" --headless=new --disable-gpu --no-pdf-header-footer \
     --print-to-pdf="<pasta>/<ficheiro>.pdf" "file:///<pasta>/<ficheiro>.html"
   ```
   Se não houver Chrome (ex. na app do Claude), gera o PDF a partir do mesmo HTML com uma biblioteca
   Python (ex. WeasyPrint); se nenhuma estiver disponível, entrega o HTML e diz para imprimir em PDF.
   Abre o PDF (Read) e confirma que o layout está bem e que tem no máximo 2 páginas.
5. **Guarda na pasta onde a sessão começou** (na app do Claude: entrega para descarregar), com o nome `<nome>_cv_<empresa>_<vaga>.html` e `.pdf`
   (minúsculas, `_` entre palavras). Nunca na pasta desta skill. Se a pasta for um repositório git,
   não faças commit dos CVs (têm telefone e e-mail) e avisa o utilizador.
6. **Responde com**: o caminho do PDF, o que puseste em destaque e porquê, e as **lacunas que ficaram
   de fora** para o utilizador confirmar.
7. **Carta de apresentação**, só se for pedida: 3 parágrafos curtos (porquê esta empresa e vaga,
   2 ou 3 provas do CV ligadas aos requisitos, disponibilidade), no mesmo estilo e com o nome
   `<nome>_cover_letter_<empresa>.pdf`.

## Regras

- Só factos do CV. Uma competência que a vaga pede e o CV não mostra é uma lacuna, mesmo que pareça
  provável que o utilizador a tenha. Se quiseres saber, pergunta.
- Não te candidates a nada, não preenchas formulários e não envies mensagens sem pedido explícito.
- No CV à medida, não inventes nem aumentes nada: nem anos, nem cargos, nem certificações, nem graus
  académicos. Um curso não concluído aparece como tal (ex. "Master's coursework in ...").
- Não guardes dados pessoais do utilizador (telefone, e-mail, morada) em ficheiros desta skill.
- Respeita os termos de uso e o `robots.txt` de cada site. Não entres em páginas que exigem login,
  não contornes bloqueios nem CAPTCHAs e lê só as páginas necessárias para o pedido.
