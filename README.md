# Contribution 1: Podcast artwork is displayed instead of episode artwork in episode details (Pocket Casts iOS #3897)

**Contribution Number:** 1
**Student:** Jonathan Toussaint 
**Issue:** [[GitHub issue link]](https://github.com/Automattic/pocket-casts-ios/issues/3897)  
**Status:** Phase 2 Complete

---

## Why I Chose This Issue

I have previous experience in iOS engineering and want to deepen my iOS skills in a production codebase. I previously contributed one PR to the Pocket Casts iOS repo ([#3526](https://github.com/Automattic/pocket-casts-ios/pull/3526)) several months ago, and want to explore the codebase more deeply to understand the architecture and how it all works. Personally, I use Pocket Casts every day (most days for at least an hour) and this is a bug I've reproduced in my day-to-day life as a user of the app.

---

## Understanding the Issue

### Problem Description

When the "use episode artwork" setting is enabled, the episode detail screen displays the podcast artwork instead of the episode-specific artwork. This is inconsistent with the player, which correctly shows episode artwork for the same episode.

### Expected Behavior

When "use episode artwork" is enabled in Settings, the episode detail screen should display episode-specific artwork when available, falling back to podcast artwork when no episode artwork exists (the same behavior as the player).

### Current Behavior

The episode detail screen always shows the podcast artwork, regardless of the "use episode artwork" setting or whether episode artwork is available.

### Affected Components

- `podcasts/Episode/EpisodeDetailViewController.swift` — the episode detail screen (the bug is here)
- `podcasts/PodcastImageView.swift` — the image view used by the detail screen
- `podcasts/ImageManager.swift` — handles image loading logic, including episode artwork resolution
- `podcasts/EpisodeArtwork.swift` — responsible for loading and caching episode artwork from show notes and embedded audio metadata

---

## Reproduction Process

### Environment Setup

No challenges - I had gone through the setup steps previously when I first contributed to this repo, so I was familiar with the necessary commands to get up and running.

### Steps to Reproduce

1. Enable "use episode artwork" in Settings → Appearance
2. Navigate to the detail screen for an episode with episode-specific artwork (e.g. podcast "La Bonne Auberge", episode "Cyberpunk - Choomba Night")
3. The artwork displayed is the podcast artwork, not the episode artwork

### Reproduction Evidence

- **Commit showing reproduction:** [Preliminary change (see screenshot before vs. after)](https://github.com/Automattic/pocket-casts-ios/commit/d8392c0500ac3966260d9f610ab38e016093f7e3)
- **Screenshots/logs:** 
Before change:
<img src="https://github.com/jon-tous/ai301-contribution-template/blob/main/screenshots/repro-before.png" width=300>
After change:
<img src="https://github.com/jon-tous/ai301-contribution-template/blob/main/screenshots/repro-after.png" width=300>
- **My findings:** The episode detail view's `performUpdateDisplayedData()` method calls `podcastImage.setPodcast(uuid:size:)`, which always loads artwork by podcast UUID. It never checks whether episode-specific artwork is available. Meanwhile, the player and other parts of the app use `setBaseEpisode(episode:size:)` or `imageForEpisode(_:size:)`, which route through a code path that respects the "use episode artwork" setting.

---

## Solution Approach

### Analysis

In `EpisodeDetailViewController.swift`, the `performUpdateDisplayedData()` method sets the image like this:
```swift
  if let uuid = episode.parentPodcast()?.uuid {
      podcastImage.setPodcast(uuid: uuid, size: .page)
  }
```

`setPodcast(uuid:size:)` on `PodcastImageView` calls `ImageManager.loadImage(podcastUuid:imageView:size:showPlaceHolder:)`, which only ever fetches podcast artwork. It has no awareness of episode artwork.

`PodcastImageView` already has a `setBaseEpisode(episode:size:)` method that calls `ImageManager.loadImage(episode:imageView:size:)` — this method first calls `loadEmbeddedImageIfRequired`, which checks `Settings.loadEmbeddedImages` and the episode artwork cache, and falls back to podcast artwork if nothing is found. This is the same path used by the player and the `EpisodeImage` SwiftUI component.

### Proposed Solution

Replace the `setPodcast` call in `performUpdateDisplayedData()` with `setBaseEpisode(episode:size:)`. No new logic needed — the correct behavior already exists in PodcastImageView and ImageManager, it just wasn't being used here.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The episode detail screen bypasses episode artwork entirely by calling the wrong method on PodcastImageView. The fix is to call the method that respects the episode artwork setting instead.

**Match:** The player (`NowPlayingPlayerItemViewController`) and the `EpisodeImage` SwiftUI view both use `imageForEpisode(_:size:)` / `loadImage(episode:imageView:size:)` to load images. `PodcastImageView.setBaseEpisode(episode:size:)` wraps this same logic and is the appropriate method for UIKit callers.

**Plan:** 
1. In `EpisodeDetailViewController.swift`, replace the `setPodcast(uuid:size:)` call inside `performUpdateDisplayedData()` with `podcastImage.setBaseEpisode(episode: episode, size: .page)`
2. Remove the now-unnecessary `if let uuid = episode.parentPodcast()?.uuid` guard (the new call doesn't need it)
3. No new tests needed — the existing image loading logic is already exercised elsewhere; this is a one-line call-site fix

**Implement:** [Branch](https://github.com/jon-tous/pocket-casts-ios/tree/jt/fix-episode-artwork)

**Review:** 
- [ ] Follows existing patterns used by the player and other episode image views
- [ ] Does not change behavior when "use episode artwork" is disabled (falls back to podcast artwork as before)
- [ ] No new dependencies or abstractions introduced
- [ ] `make format` run on changed files

**Evaluate:** Manually test with an episode that has episode artwork (e.g. "La Bonne Auberge" / "Cyberpunk - Choomba Night") with the setting on and off. Confirm the detail screen now matches the player when the setting is enabled, and still shows podcast artwork when disabled.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
