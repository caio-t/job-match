# job-match

A [Claude Code](https://claude.com/claude-code) skill that finds open job postings and ranks them by
how well they match your CV.

- Reads your CV from the folder where the session starts (PDF, DOCX, HTML, Markdown, or profile data
  in code). With no CV there, it asks for one or reads your LinkedIn profile.
- Searches a specific company's careers page (e.g. "the Aubay openings") or the job boards that fit
  your region (Portugal, Brazil, remote, and others).
- Reads each posting in full and scores it on required tech, seniority, role type, location and work
  model, skipping postings with hard blockers such as a required language you don't speak.
- Lists 10 postings by default, or the number you ask for, with match, reasons, gaps and direct links.

It only searches and lists. Applying or tailoring a CV happens only when you ask for it.

## Install

Copy the `job-match` folder into your Claude Code skills directory:

```bash
git clone https://github.com/<your-user>/job-match ~/.claude/skills/job-match
```

## Use

Open Claude Code in the folder that holds your CV and ask, for example:

- "Lista as 10 principais vagas da Aubay com match no meu perfil"
- "Top 5 remote Node.js jobs that match my CV"
- "/job-match 15 vagas em Lisboa"
