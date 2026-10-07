---
name: tailor-resume
description: Tailor your base resume to a specific job description, export an ATS-friendly PDF, and generate a matching cover letter and LinkedIn InMail, along with a fit score and apply/no-apply recommendation. Use when given a job description (pasted text, a JD file, or a URL) and asked to tailor the resume, apply, or evaluate a role.
---

# Tailor Resume

Turns a job description into: a tailored ATS-friendly resume PDF, a cover letter, a LinkedIn InMail draft, and a fit score with an apply/no-apply call - without changing the resume's visual design or inventing experience.

## Hard rules (never break these)

1. **Never change the resume's look, layout, theme, fonts, or section structure.** Only edit the text content within the existing template/source.
2. **Never fabricate resume content.** Every bullet, skill, or claim in the tailored resume must already exist in `base_resume/` (possibly reworded or reordered/reprioritized) - never invent metrics, tools, titles, or accomplishments not present in the base resume.
3. **Keep the resume ATS-friendly**: plain selectable text (no text-in-images, no tables that break linear text extraction, no columns that scramble reading order), standard section headers, no special characters that mangle parsing.
4. **Always produce a fit score (0-100) and an explicit Apply / Don't Apply / Borderline recommendation**, with the reasoning shown.
5. **New, real experience the user reveals mid-conversation must be verified before use.** If the user mentions a genuine accomplishment, skill, or project not yet in `resume_master_content.md`, ask clarifying questions to pin down the precise, honest scope of the claim before writing it into a tailored resume. Then add it to the permanent content bank (tagged with the date and "confirmed directly by [user], not from the original resume") so future runs can use it too - unless the user says not to add it there.
6. **If a JD names a specific required qualification with zero grounding in the content bank, stop and ask before building a full package** rather than silently scoring around it or guessing. This matters most for named technical requirements (a specific tool, a specific type of deep expertise) - don't assume the user has it, and don't assume they don't.

## One-time setup (do this before first use)

This skill needs two things that are specific to the person using it, not shipped with the skill itself:

- `base_resume/resume_master_content.md` - a content bank of every real, verifiable bullet/role/skill from the user's actual resume(s). See `templates/resume_master_content.template.md` in this skill's folder for the structure to copy and fill in. **This is the only source of truth for resume content** - tailoring means selecting/reordering/rewording from this file, never adding anything not in it.
- `base_resume/resume_source.html` - the locked visual template. See `templates/resume_source.template.html` for a clean, ATS-friendly single-column starting point (no photo, no tables/columns that break reading order) - replace the placeholder header, experience, education, etc. with the user's real information, or swap in their own existing resume design as long as it stays ATS-friendly. **This defines the look/theme that must never change** once set.
- `preferences.md` (in this skill's folder) - target titles/seniority, domain preferences, compensation floor, tone preference. See `templates/preferences.template.md`.
- A PDF rendering pipeline. [WeasyPrint](https://doc.courtbouillon.org/weasyprint/) is a good default (`pip install weasyprint`); on macOS it needs Homebrew's `pango`/`glib` on the dynamic library path:
  ```
  brew install pango
  export DYLD_FALLBACK_LIBRARY_PATH="/opt/homebrew/lib"
  weasyprint <source>.html <output>.pdf
  ```
  On Linux, `apt install libpango-1.0-0 libpangocairo-1.0-0` (or your distro's equivalent) is usually enough; no extra library path needed. Adjust the render command for whatever PDF tool is actually installed.

If `base_resume/resume_source.html` or `resume_master_content.md` is missing, stop and ask the user to set them up (or do it together) rather than inventing resume content from scratch.

## Workflow

Given a job description (pasted text, a file path, or a URL):

1. **Load inputs**: the JD, `base_resume/resume_master_content.md` (content bank), `base_resume/resume_source.html` (template), and `preferences.md` (must-have criteria: target titles, domain preference, comp floor, etc.) in this skill's directory.
2. **Parse the JD**: extract company, role title, seniority level, required/preferred skills, comp range if disclosed, and any deal-breakers.
3. **Score fit** against `preferences.md` and the base resume:
   - Keyword/skill overlap between JD requirements and base resume content.
   - Title/seniority match (JD level vs. the user's target levels). Company-specific title ladders vary a lot (e.g. "Senior PM" at one company can outrank "Director" at another) - don't assume a title is a downlevel just because the word sounds junior; ask the user if unsure and record the answer in `preferences.md` for next time.
   - Must-have criteria from `preferences.md` (comp floor, domain preference, title level).
   - Produce a 0-100 score and a short breakdown of what helped/hurt it.
   - Recommendation: **Apply** (score generally >=70 and no must-have violated), **Borderline** (50-69, or one soft must-have missed), **Don't Apply** (<50, or a hard must-have like comp floor clearly violated).
4. **Tailor the resume**: copy `resume_source.html` into the job's output folder, and within that copy, re-prioritize, select, and reword bullets/skills FROM `resume_master_content.md` to mirror the JD's language and emphasis. Trim to fit ~1-2 pages by cutting less-relevant roles/bullets. Do not add anything not already in the content bank. Never edit the CSS or HTML structure - only the text content.
   - **Role order is fixed: strict reverse-chronological by end date, always.** Never reorder roles by relevance to the JD, only reorder or cut bullets *within* a role. Double-check each role's end date against the one below it - it's easy to misorder two roles that overlapped in seniority but not in time.
   - **The "Core Expertise" / skills line is dynamic**: rebuild it every time from a keyword pool in `resume_master_content.md` (see the template for how to structure this). Pick ~7-9 terms that best mirror the JD's own vocabulary (prefer the JD's exact phrasing when a truthful equivalent exists in the pool), ordered most-relevant-to-this-JD first. This is the main ATS keyword-matching lever - treat it as the section that should look meaningfully different across applications. Never introduce a keyword not grounded in the content bank.
   - **Headline and sub-headline (under the name) are also dynamic, not fixed text**: reword them per JD to foreground the most relevant positioning. This is framing/positioning language, not a factual claim, so it can be reworded freely - but never claim a title the user doesn't actually hold. Keep it editing text only, never the CSS/layout - same hard rule as everywhere else.
   - **Check literal phrase match, not just topical similarity**: after drafting the Core Expertise line, re-read the JD's qualifications/responsibilities line by line and confirm its key phrases appear near-verbatim wherever a truthful grounded equivalent exists - don't settle for a topically-similar-but-differently-worded term. If the pool's existing entries don't capture a JD phrase precisely enough, add a more precisely-worded entry to the pool (grounded in an exact quote from the content bank) rather than forcing a loose match.
5. **Export PDF**: render with the configured pipeline. Sanity-check it's still a single coherent, ATS-parseable document (extract its text back out and confirm it reads cleanly, in the right order).
6. **Generate cover letter** (`cover_letter.md`): tone per `preferences.md`, grounded only in real resume content, explicitly connecting the user's background to this JD's priorities. Keep it under ~300 words unless asked otherwise. Make the positive case only - honest gaps/caveats belong in `score.md`, not the cover letter.
7. **Generate LinkedIn InMail** (`inmail.md`): concise, short enough for a cold outreach message (~100-150 words), referencing the specific role and one or two concrete, real qualifications.
8. **Write the score report** (`score.md`): fit score, breakdown, apply/don't-apply/borderline call, and any flags (e.g. "comp not disclosed", "title below target level", named required qualifications with no grounded match).
9. **Self-review before delivering** (every run, no exceptions): re-read every generated file (resume, cover letter, InMail, score) against `resume_master_content.md` and check for:
   - **Hallucinations**: any claim, metric, tool, or accomplishment not traceable to the content bank.
   - **Wrong data**: numbers, dates, titles, or names that don't match the content bank or the JD (e.g. a figure transcribed wrong, a stale title, a company name typo, wrong recipient/company references left over from another application).
   - **Profanity or unprofessional language.**
   - **Privacy/security issues**: no real credentials, personal data beyond what's already on the base resume (phone/email/LinkedIn), no accidental inclusion of another person's private info, no sensitive data leaking into the wrong file.
   - **Internal consistency**: dates/titles match across resume, cover letter, and InMail; the Core Expertise line's claims are all backed by bullets elsewhere in the resume.
   - **No em-dashes or other formatting quirks** the user hasn't asked for - check what tone/formatting rules are in `preferences.md`.
   Fix anything found before sending. If something is ambiguous, flag it to the user rather than guessing.
10. **Save everything** to `applications/<company>_<role-slug>_<YYYY-MM-DD>/`: `resume.pdf`, `resume_source.html` (the tailored copy, not the base), `cover_letter.md`, `inmail.md`, `score.md`, and the original JD as `jd.md`.
11. **Report back**: fit score + recommendation up front, then a one-line summary of what was tailored and where the files landed.

## Notes

- If the JD is a URL, fetch and extract the JD text first; don't guess content from the URL alone.
- If comp isn't disclosed in the JD, don't penalize the score for it - just flag it as unknown in `score.md` rather than assuming it fails the floor.
- If the user gives feedback on a generated resume/letter (tone, emphasis, a section to cut), treat that as a standing preference - note it in `preferences.md` under "Learned preferences" so future runs apply it automatically.
- For an informal/quick request (e.g. "just a cover letter paragraph for this one"), match that narrower scope exactly - don't build a full package (resume, score, InMail) when only one piece was asked for.
