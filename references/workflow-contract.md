# Cross-Repository Workflow Contract

Version: 1.0
Purpose: a small interface contract shared by `video-editing-skills` and `local-generative-colab-skill`. This file is the bridge; detailed references are loaded only when needed.

## Roles
- Video-editing repository owns: brief interpretation, editorial decisions, timeline, graphics, captions, sound design intent, render specification, creative review, delivery QA.
- Local-generative-colab repository owns: capability detection, remote backend setup, model discovery/selection, remote job execution, persistence, artifact transfer and backend logs.
- Neither repository may claim the other's work succeeded without checking its output.

## Stage contract
Each stage records a manifest entry with:
- `stage`, `status` (`planned|running|passed|failed|blocked`), UTC start/end,
- inputs and output paths with sizes and SHA-256 where practical,
- command or job identifier, tool/model versions, backend, seed when supported,
- log path, validation performed, failure reason, retry count.

Outputs are written into the project workspace, never only to ephemeral remote storage. A stage is not `passed` merely because a command was submitted; its outputs must be retrieved and validated.

## Capability preflight
Before installing packages or submitting jobs, check only capabilities required for the requested task:
1. local shell and file access;
2. Python and required packages;
3. ffmpeg/ffprobe for media work;
4. Node/Remotion or Blender only if selected;
5. selected remote CLI, authentication, quota, and a harmless connectivity/status check;
6. enough available disk and remote runtime for the planned job.

Report each as `available|unavailable|not-needed|not-verified`. Never print secrets. Never infer successful authentication from a config file's existence. Never claim GPU execution without a job result or backend evidence.

## Remote-job lifecycle
`planned -> submitted -> running -> completed -> retrieved -> validated`.
Failure or timeout goes to `failed` or `blocked`; it must not be silently called complete.
- Use a unique run ID and isolated output directory.
- Persist source inputs and the exact script/config before launch.
- Limit concurrent jobs to confirmed quota and available resources.
- Retry transient failures at most twice by default, with backoff; do not retry invalid inputs or deterministic code errors blindly.
- On completion, retrieve outputs immediately, verify expected files, nonzero size, hashes when available, and logs.
- Treat missing completion markers, incomplete downloads, or stale files as failure.

## Model selection contract
Choose a model only for a specific shot/task. Record model ID, revision, license/source, required VRAM, precision, expected output, and reason. Prefer reproducibility over a vague claim that a model is currently "best". If live discovery is unavailable, say so and use a known compatible fallback only if permitted.

## Media handoff contract
The editing pipeline must receive a manifest containing file path, type, duration/frame count, dimensions, frame rate, audio streams, provenance/license, and generation/extraction stage. Preserve original assets. Generated or reconstructed footage must not be represented as authentic source footage.

## Quality contract
Separate four checks:
1. **Process:** command/job completed and logs contain no unresolved fatal error.
2. **Artifact:** files exist, are non-empty, decodable, and match expected metadata.
3. **Content:** intended shots, captions, overlays, pacing, and audio are present in the actual rendered media.
4. **Editorial:** result serves the brief, has coherent continuity, clear hierarchy, purposeful transitions, and acceptable sound.

Text mentions, generated plans, and a successful exit code alone do not prove content or editorial quality. Critical failures block delivery; all other known defects are disclosed.

## Security and privacy
Never put API keys, cookies, tokens, personal credentials, or private source media in committed files, prompts, public logs, or issue reports. Redact command output before storing it. Use least privilege and user-approved data destinations.

## Compatibility
Every run records the commit SHA of both repositories and the version of this contract. Pin the revisions used for a project; do not mix unpinned `main` content mid-run.
