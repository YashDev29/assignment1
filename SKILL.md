---
name: "ai-job-digest"
description: Build a daily digest of up to 100 recent AI job postings (AI engineer, ML engineer, GenAI/LLM engineer, agentic AI, data scientist, research scientist, MLOps, AI product roles) for India or any region, by searching and fetching public job boards, with LinkedIn postings included only where they are reachable. Make sure to use this skill whenever the user asks for today's AI jobs, top 100 AI job postings, "AI jobs on LinkedIn", AI jobs in India or a city like Bengaluru or Hyderabad, a daily or weekly AI job list, new ML/GenAI openings, or wants to find, track or compare AI/ML job openings, even if they don't say "digest".
---

# AI Job Digest

Produce a deduplicated, recency-ordered list of AI job postings, shown in chat as tables. Every row must be backed by a page you actually saw.

## Say this once, briefly, at the top of the answer

- LinkedIn job listings need a login and scraping them breaks LinkedIn's terms, so this skill cannot pull a live LinkedIn feed or a LinkedIn "top 100". LinkedIn rows appear only when a public LinkedIn or mirror page shows up in results.
- The list comes from public boards. The count is best effort: if fewer than 100 verified postings are found, return what was found, state the exact number, and never pad or invent rows, links, companies or salaries.
- Dates are as shown by each source at fetch time and some pages may be cached.

## Inputs (ask only if missing, otherwise use defaults)

- Role focus: default all AI/ML roles
- Location: default worldwide/remote; if the user names a country or city (for example India, Bengaluru) use it. Do not assume the user's location.
- Seniority: any. Count: 100 max.

## Workflow

1. **Fetch listing pages first, they yield the most rows.** In testing, Wellfound role pages could be opened with web_fetch and returned about 25 postings per page with title, company, location, pay, experience and posted age. Pattern: `https://wellfound.com/role/l/<role-slug>/<country-or-city>` with `?page=2`, `?page=3`. Useful slugs: `ai-engineer`, `machine-learning-engineer`, `data-scientist`, `product-manager`. Fetch pages 1 to 3 for each relevant role. Only fetch URLs that appeared in earlier search results or in the user's message.
2. **Then search for breadth.** Run short, distinct queries per role family and location, for example `Generative AI engineer <city> jobs`, `LLM engineer <city> hiring`, `AI research scientist <country>`, `site:linkedin.com/jobs AI engineer <country>`. Results from foundit, Bayt, aijobs.net, Greenhouse, Lever, Ashby and company career pages are usually usable. Use the actual current date when a date is needed.
3. **Know what fails.** Indeed blocks fetches (bot detection): use only what its search snippets show and link to the results page, labeled "Indeed results". LinkedIn search results are mostly profile or news pages, so expect very few LinkedIn rows. Do not retry blocked sites repeatedly.
4. **Extract per posting:** title, company, location and work mode, posted age or date, pay only if stated, source name, direct URL.
5. **Filter.** Keep genuine AI/ML roles. Drop duplicates (same company, title, location), non-AI roles (plain backend, QA, DevOps, support), roles whose location is "everywhere" when a country was requested, and anything older than about 30 days unless the date is unknown (then keep it in a separate "date not shown" batch). Drop postings with a clearly old date (for example a 2025 posting date).
6. **Order** by recency (hours/days first, then weeks, then about a month, then date-unknown), then by pay or location transparency. Say so in one line.
7. **Cap** at the requested count.

## Output format (chat)

Open with the caveats above and the exact number found, plus one line of market context only if a retrieved source supports it (cite it).

Then numbered markdown tables in batches by recency, for example "1-19 last ~2 weeks", "20-38 about 3 weeks to 1 month", "39-N posted date not shown". Columns: `# | Title | Company | Location | Posted | Source`. Put pay in the title cell in brackets when stated, e.g. `AI Engineer (₹20L-45L)`. Source is a markdown link to the real posting URL. For date-unknown rows drop the Posted column.

End with 1 to 3 short observations only if clearly supported by the list (common cities, frequent role types, pay range seen), and offer to filter by city, seniority or salary, merge a pasted LinkedIn alert list, or fetch more pages.

## Quality rules

- Every row traces to a search result or fetched page. Unknown fields are "not shown", not guessed.
- Skip a row rather than guess a company or location from a vague snippet.
- Titles and short facts only; never reproduce job description text.
- If the user pastes LinkedIn alert emails or links, merge them, mark them "user-provided", and dedupe.
- On a repeat run in the same conversation, mention which postings are new versus already listed.
