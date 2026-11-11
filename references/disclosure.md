# Disclosure and output format

## Principal functionality developed for this Hackathon

Life Chapters is a new orchestration Skill written for the UGREEN AI NAS Skills Hackathon. The following were designed during the Hackathon:

- the cross-media collection workflow, which combines file search, photo search, document reading, and optional audio transcription for one user-defined topic;
- the anchor-based scoping rules (dates, folders, people, keywords, place) and the empty-result and partial-result behavior;
- the use of a shared file tag as a persistent link between related files, which allows one-step recall later;
- the chapter format described in the section "How the output looks" below.

## Third-party components and open source libraries

None. The Skill contains only Markdown instructions. It relies on the NAS experts and capabilities published in the official Uliya Skill Kit capability catalog (version 0.7.3): `file-agent`, `photo-agent`, `document-agent`, and `knowledge-agent`.

## AI tools used and their principal uses

- **Claude (Anthropic), through Claude Code, with the official uliya-skill-kit**: used to check feasibility against the capability catalog, draft the Skill text, and run the kit's static checker.
- **UGREEN NAS experts at runtime**: perform all real NAS work (search, reading, transcription, tagging, album creation, saving) after delegation. Life Chapters itself does not run code.

## Data handling

All processing happens through the NAS experts on the user's own device. Life Chapters does not send user data to external services and contains no credentials, scripts, or downloads.

## How the output looks

Every chapter (the dossier written in Step 5) follows this structure. Use only facts returned by completed delegations. Leave a section out when there is no verified content for it, and say so in the coverage note.

1. **Title**: the topic name and the date range, for example "2025 College Project (January to May 2025)".
2. **Overview**: one paragraph, at most five sentences, stating what the topic was and its main outcome, based on the document and transcript facts.
3. **Timeline**: a dated list built from document dates, photo capture dates, and recording dates. Earliest first. Each entry names its source file.
4. **People**: the names that appear in photos, documents, or recordings, with their role when a source states it.
5. **Key facts and outcomes**: short bullet points such as decisions, results, grades, or deliverables. Each one names its source file and page or section.
6. **File index**: four groups (documents, photos, recordings, videos). Each item shows the exact path and the date. Photos may be summarized as a count plus the album name when there are more than 20.
7. **Coverage note**: counts of files found, documents read, recordings transcribed, and anything skipped or failed, with the reason.

### Example

The values below only show the layout. A real chapter contains only the user's verified data.

```markdown
# 2025 College Project (January to May 2025)

## Overview
A four-person final-year project building a crop disease detection app. The final report was submitted on 12 May 2025 and scored 86/100.

## Timeline
- 14 Jan 2025: project proposal approved (Proposal_v2.pdf, page 1)
- 03 Mar 2025: field visit photos, 24 photos (album: 2025 College Project)
- 21 Apr 2025: viva practice recording (viva_practice.m4a)
- 12 May 2025: final report submitted (Final_Report.pdf, page 2)

## People
- Rahul: team lead (Proposal_v2.pdf)
- Priya: data collection (Final_Report.pdf, section 3)

## Key facts and outcomes
- Model accuracy 91% on the test set (Final_Report.pdf, section 5)
- Final score 86/100 (Grade_Sheet.pdf, page 1)

## File index
- Documents (6): Proposal_v2.pdf, Final_Report.pdf, Grade_Sheet.pdf, ...
- Photos (81): album "2025 College Project"
- Recordings (3): viva_practice.m4a, ...
- Videos (2): demo_final.mp4, ...

## Coverage
Found 46 files and 81 photos. Read 6 of 6 documents. Transcribed 3 of 3 recordings. No failures.
```
