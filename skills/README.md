# Claude Skills

Two Claude skills I built and use for real work — not demos.

A skill is a set of instructions that teaches Claude how to handle a specific recurring task
correctly, every time, without me re-explaining the rules from scratch in each conversation.
These two solve problems I actually had.

---

## barometer-report

A workflow for producing research-quality analytical reports from raw survey data — built from
several years of experience running a longitudinal MSE (micro and small enterprise) barometer
study. It encodes the rules that separate a competent first-pass analysis from a genuinely
rigorous one: correct terminology, segmentation discipline, denominator accuracy, longitudinal
framing, and a calibrated, non-overclaiming tone. This is domain expertise — the kind that
normally takes years to develop — captured so it can be applied consistently and reused.

📄 [`barometer-report-SKILL.md`](./barometer-report-SKILL.md)

---

## rename-invoices

An automation that scans a folder for unfiled invoice PDFs, reads each one to identify the
vendor and date, renames it to a consistent format, and files it into the correct monthly
folder. Built to remove a small but recurring piece of admin from my own workflow. It also
handles the practical edge cases that come up in real use — like files that look present but
are actually still cloud-only and haven't downloaded yet.

📄 [`rename-invoices-SKILL-anonymised.md`](./rename-invoices-SKILL-anonymised.md)

See also: [`Jak-jsem-Claude-naucila-skill.mp4`](../projects/Jak-jsem-Claude-naucila-skill.mp4) —
a recording of the actual session where I taught Claude this skill, step by step.

---

## Why these two together

One is deep domain expertise turned into a reusable process. The other is an everyday
operational task automated end to end. Different problems, same underlying approach: figure out
the rules an expert would follow, write them down clearly enough that an AI can apply them
consistently, and let it run the task instead of doing it by hand each time.
