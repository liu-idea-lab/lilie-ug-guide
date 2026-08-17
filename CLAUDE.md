# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Current state: retired, redirects only

As of August 2026 this site no longer publishes content. `index.html` is a redirect to
https://entrepreneurship.rice.edu/, which is now the single source for undergraduate
courses, programs, and events.

Live URL: https://liu-idea-lab.github.io/lilie-ug-guide/ (GitHub Pages, `main` branch, root).

The redirect uses three layers so it works with or without JavaScript: a `meta refresh`,
a `location.replace()` call, and a visible link as the fallback. `rel="canonical"` and
`noindex` point search engines at the Lilie site instead of this one.

Why it was retired: the content had gone stale (BUSI course codes after the Fall 2026
ENTR migration, spring 2026 event dates, a missing course) and keeping a second listing
in sync with entrepreneurship.rice.edu was not worth the upkeep.

## Legacy: the old data pipeline

`data.json` is still in the repo but nothing reads it. It was generated from a Google
Sheet through an Apps Script (`Code.gs`, never tracked here) that called OpenAI to turn
each sheet tab into structured JSON, then pushed the result to `main` via the GitHub API.

If someone runs "Publish > Publish to Website" from that Sheet, it will overwrite
`data.json` and nothing visible will change. Removing the Apps Script trigger is the
clean way to stop that. Before deleting `data.json`, check the script is gone, since it
pushes by blob SHA and may error against a missing file.

The pre-redirect version of the site is in git history at commit `5c40a93`.
