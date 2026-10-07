# tailor-resume: a Claude Code skill

Turns a job description into a tailored, ATS-friendly resume PDF, a cover letter, a LinkedIn InMail draft, and an honest fit score - without ever inventing experience you don't have.

This folder is meant to be copied into someone else's own `.claude/skills/` directory. It does not contain any of my personal resume data - you fill that part in yourself, locally, and it never leaves your machine unless you choose to share it.

## What's in here

```
tailor-resume-skill/
  SKILL.md                              <- the skill definition Claude Code reads
  README.md                             <- this file
  templates/
    resume_master_content.template.md   <- copy to base_resume/resume_master_content.md
    resume_source.template.html         <- copy to base_resume/resume_source.html
    preferences.template.md             <- copy to preferences.md (next to SKILL.md)
```

## Setup (5-10 minutes)

1. **Install the skill.** Copy the whole `tailor-resume-skill/` folder into your own Claude Code project at `.claude/skills/tailor-resume/` (the folder name under `skills/` should be `tailor-resume` to match `SKILL.md`'s `name:` field).

2. **Build your content bank.** Copy `templates/resume_master_content.template.md` to `base_resume/resume_master_content.md` in your project, and fill it in from your actual resume(s). Every bullet should be something real and verifiable - this file is the only thing the skill is allowed to draw resume content from, so whatever isn't in here can't end up on a tailored resume. The more detail you put in (including an older, more detailed resume version alongside a newer concise one, if you have both), the better it can tailor.

3. **Set up your visual template.** Copy `templates/resume_source.template.html` to `base_resume/resume_source.html` and fill in your real name, contact info, roles, and education. This file's CSS/layout is a clean, ATS-friendly single-column design (no photo, no tables/columns that scramble text extraction order) - feel free to restyle it to your taste, or replace it entirely with your own HTML resume, as long as it stays plain-text-extractable. Once you're happy with the look, the skill will never change the structure again, only the text content.

4. **Set your preferences.** Copy `templates/preferences.template.md` to `preferences.md` (next to `SKILL.md`) and fill in your target titles, domain preference, compensation floor, and tone.

5. **Install a PDF renderer.** [WeasyPrint](https://doc.courtbouillon.org/weasyprint/) is a solid free option:
   ```
   pip install weasyprint
   ```
   On macOS you'll also need Homebrew's `pango`:
   ```
   brew install pango
   export DYLD_FALLBACK_LIBRARY_PATH="/opt/homebrew/lib"
   ```
   On Linux, your distro's `libpango`/`libpangocairo` packages are usually enough. Any other HTML-to-PDF tool works too - just tell Claude which one you're using if it's not WeasyPrint.

## Using it

Once set up, just give Claude a job description - pasted text, a file, or a URL - and ask it to tailor your resume, evaluate a role, or apply. It will:
1. Parse the JD and score your fit (0-100, with an Apply / Borderline / Don't Apply call).
2. Tailor your resume (reordering/rewording only, never inventing).
3. Export a PDF and sanity-check it reads cleanly for ATS parsing.
4. Draft a cover letter and a short LinkedIn InMail.
5. Save everything to `applications/<company>_<role>_<date>/`.

If a JD requires something specific you haven't told it about yet (a tool, a certification, a type of experience), it should ask rather than guess - and if you confirm you do have it, it'll add it to your content bank (with your permission) so future tailoring runs remember it too.

## Why this structure

The core discipline this skill enforces is: **separate the content (what's true about you) from the presentation (how it's tailored and worded) from the judgment (should you even apply)**. The content bank is the only source of truth; nothing is invented; every run produces an honest score, not just a prettier resume. That's the part worth keeping intact if you adapt this for yourself.
