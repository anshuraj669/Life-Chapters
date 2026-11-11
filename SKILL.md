---
name: life-chapters
description: Life Chapters, a cross-media project and event dossier that gathers related documents, photos, recordings, and videos on the NAS into one summary, a shared file tag, and a photo album. Use when the user asks to show, collect, or recall everything related to one project, trip, event, course, or period, for example "show me everything related to my 2025 college project".
---

# Life Chapters

Every project, trip, or event in a person's life becomes one chapter: its files, photos, and voices brought together in one place.

# Goal

Life Chapters answers one question: "What do I have about this topic?" It collects every related file that already exists on the user's NAS (documents, photos, audio recordings, and videos), explains what the topic was about from the content of those files, and links the results so they can be recalled in one step later.

The user-visible results are:

1. A **topic dossier**: a one-page Markdown summary with what the topic was, a dated timeline, the people involved, key facts and outcomes, and every related file grouped by type. The format is shown in the "How the output looks" section of [references/disclosure.md](references/disclosure.md#how-the-output-looks).
2. A **topic tag** added to every confirmed file, so the whole collection can be listed again later across folders without a new search.
3. A **photo album** for the topic, created from the same people, date range, and folder conditions.

Supported scope: files the user is authorized to access on this NAS, located through the user's own folders, dates, people, and keywords. Life Chapters does not search the internet, does not read files outside the scope confirmed by the user, and does not edit, move, or delete any user file. Its only write effects are the tag, the album, and the saved dossier, and each needs user confirmation first.

# Required input

Collect only the following before delegating. Ask in one short message and accept whatever the user already said.

1. **Topic**: a short name for what to collect, for example "2025 college project" or "Goa trip December 2025".
2. **At least one anchor**, ideally two or more:
   - a date range (start and end date);
   - one or more folders the user names;
   - names of people as they appear in the user's Photos people list;
   - keywords likely to appear in file names or in text inside photos and documents;
   - a place name.
3. **Recordings**: whether audio recordings found for the topic should be transcribed. Default: no. If yes, transcribe at most 5 recordings.
4. **Tag name**: propose a lowercase hyphenated tag derived from the topic, such as `college-project-2025`, and let the user change it.
5. **Dossier folder**: where the dossier should be saved. Default: the first folder the user named; if no folder was named, the user's own home directory as resolved by the file expert.

Recall mode: if the user asks to see a topic again and gives an existing tag name, only the tag name is required.

If the user cannot give any anchor, explain that a topic name alone is too broad to search reliably, ask for one anchor, and wait. Do not guess dates, folders, or people.

# Generation-time capability evidence

- `ug-filesystem` → `file-agent`
- `ug-filemgr` → `file-agent`
- `ug-photo` → `photo-agent`
- `ug-document` → `document-agent`
- `ug-knowledge` → `knowledge-agent`

Video files are found as files through `file-agent`. The media library search is not used, because it covers movies and TV rather than the user's own recordings.

# Workflow

Run the steps in order. Every downstream step uses only the real result returned by the upstream delegation, with the actual paths, names, and dates copied into its instruction. Never pass a vague reference such as "the files from before". Read-only steps run without confirmation; write steps run only after the user confirms in Step 6.

**Step 0. Confirm scope.** Repeat the topic, anchors, recording choice, tag name, and dossier folder back to the user in one short summary and wait for a yes. While waiting, report the state as `waiting_user`.

**Step 1. Recall mode (only when the user gave an existing tag).** Delegate:

`delegate --target file-agent --instruction 'List all files that carry the tag the user named for this topic, searching recursively in every folder the user can access. Return each file path, type, size, and modified date. Read only. Do not change tags or files. If the tag does not exist or no file carries it, say so clearly.'`

Include the tag name given by the user in the instruction. If files are returned, go to Step 5 to rebuild the dossier from them, then finish without writing anything unless the user asks to update the dossier. If the tag does not exist or returns no results, tell the user and offer a new collection with anchors.

**Step 2. Find related files.** Delegate:

`delegate --target file-agent --instruction 'Find files related to the topic the user confirmed. Search only inside the folders the user named; if none were named, resolve the user home directory and search there. Limit results to files modified within the confirmed date range when one was given, and to file names containing the confirmed keywords when keywords were given. Include documents, images, audio, and video. Return at most 200 results as exact paths grouped by type, with size and modified date. Read only. Do not open, move, rename, tag, or delete anything. If nothing matches, return an empty result and say so.'`

Write the real topic, folders, dates, and keywords into the instruction. If the result is empty, report that no matching files were found with these anchors and ask the user whether to change the anchors. Do not widen the search on your own.

**Step 3. Find related photos.** Delegate:

`delegate --target photo-agent --instruction 'Search the photo library for photos and videos matching the confirmed conditions: the named people, the confirmed date range, the confirmed place, the confirmed folders, and photos whose file name or recognized text contains the confirmed keywords. Use only the conditions the user actually gave. Return at most 100 items with path, capture date, people, and place. Read only. Do not create or change albums. If nothing matches, return an empty result and say so.'`

Write the real conditions into the instruction. An empty photo result is not a failure; continue and note it in the dossier.

**Step 4. Understand the documents.** If Step 2 returned documents, delegate once with up to 30 of the returned document paths:

`delegate --target document-agent --instruction 'Read the listed documents and, for each one, return a one-sentence summary, any dates, people, decisions, results, or grades it mentions, and the page or section where each fact appears. Then return a combined summary of what the topic was about and its main outcome, using only facts found in these documents. Mark any document that could not be read. Read only. Do not write or save any file.'`

List the real document paths from Step 2 inside the instruction. If Step 2 returned more than 30 documents, use the 30 most recently modified and state in the dossier how many were not read.

**Step 4b. Transcribe recordings (only if the user said yes).** If Step 2 returned audio files, delegate once with up to 5 of the returned audio paths:

`delegate --target knowledge-agent --instruction 'Transcribe the listed audio recordings. For each one return the transcript text, the recording length, and a three-sentence summary of what was discussed. Mark any recording that could not be transcribed and why. Read only. Do not save any file.'`

List the real audio paths in the instruction. If the user said no, list recordings in the dossier by name and date only.

**Step 5. Write the dossier draft.** Delegate:

`delegate --target document-agent --instruction 'Write a Markdown dossier in a concise, friendly style using only the facts supplied in this instruction. Follow this structure: title, one-paragraph overview, dated timeline, people involved, key facts and outcomes, file index grouped as documents, photos, recordings, and videos with each exact path, and a coverage note stating how many files were found, read, transcribed, or skipped. Do not add facts that are not supplied. Return the draft text only and do not save it.'`

Paste the verified results from Steps 1 to 4b into the instruction: file lists, document facts, photo list, and transcript summaries. Follow the format in [references/disclosure.md](references/disclosure.md#how-the-output-looks).

**Step 6. Confirm the write actions.** Show the user the dossier draft and a short action list:

- create the tag (only if it does not exist yet) and add it to the listed number of files;
- create an album with the topic name from the photo conditions;
- save the dossier as a Markdown file in the confirmed folder.

Ask the user to approve all, approve some, or edit first. Report `waiting_user` while waiting. Run only the approved actions below.

**Step 7. Tag the files (if approved).** Delegate:

`delegate --target file-agent --instruction 'List existing tags. If the confirmed tag does not exist, create it. Then add the confirmed tag to each file in the confirmed list. Do not change any other tag and do not move, rename, or delete any file. Return how many files were tagged, which failed, and why.'`

Write the real tag name and the exact confirmed file paths into the instruction. Tag changes may need platform approval; report `waiting_approval` while the approval is pending.

**Step 8. Create the album (if approved).** Delegate:

`delegate --target photo-agent --instruction 'Create a condition album named after the confirmed topic using only the confirmed people, date range, place, folders, and keywords. Return the album name, its identifier, and how many items it contains. Do not change or delete any existing album.'`

Write the real album name and conditions into the instruction. Report `waiting_approval` while the approval is pending.

**Step 9. Save the dossier (if approved).** Delegate:

`delegate --target knowledge-agent --instruction 'Save the supplied Markdown text as a new file in the confirmed folder with a file name based on the topic and the word dossier. Do not overwrite an existing file; if the name is taken, add a number. Return the saved file path and confirm the write was verified.'`

Paste the final approved dossier text, the real folder, and the topic into the instruction. Report `waiting_approval` while the approval is pending.

# Completion and failure

Handle every delegation state explicitly:

- `completed`: use only the returned business result. Report results to the user only after the delegation for that step is `completed`.
- `failed`: tell the user which step failed and the reason given. If Step 2 and Step 3 both fail, stop, because there is nothing to build a dossier from. If only one of them fails, continue with the other and say clearly in the dossier which source is missing. If Step 4 or 4b fails, continue and list those files as not read. If Step 7, 8, or 9 fails, report it and still show the results of the other write steps.
- `cancelled`: stop the current step and every step that depends on it. Report what was already completed, for example "12 files tagged, album not created, dossier not saved".
- `waiting_user`: show the question or choice to the user and pause. Do not continue until the user answers.
- `waiting_approval`: tell the user that an approval is pending for the tag, album, or save action and pause. Never report that action as done before the delegation returns `completed`.

Empty result rules: if no files and no photos are found, report "no results for these anchors", list the anchors that were used, suggest one anchor to change, and stop. Do not substitute similar or unrelated files, and do not widen the date range, folders, or people without the user's agreement.

Partial results: always state counts, for example "found 46 files and 81 photos, read 30 of 38 documents, transcribed 3 of 3 recordings". Never present a partial dossier as complete.

Final report when everything is `completed`: the dossier path, the tag name with the number of tagged files, the album name with its item count, and one line telling the user they can recall this collection later by asking for the tag.

Third-party components, AI tools, the functionality developed for this Hackathon, and the output format are described in [references/disclosure.md](references/disclosure.md).
