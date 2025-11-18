ArtMint – Human-Readable System Blueprint (v1)

A complete product specification for the current version of the ArtMint app.

1. Product Overview

ArtMint is a social AI-art platform where users can:

• Create AI-generated artwork
• Publish artwork publicly
• Browse global artwork in a Gallery
• Follow other creators
• Collect artwork (except their own)
• View artwork created and collected by any user
• Maintain full public and private profiles

ArtMint combines creativity, collecting, and social discovery into one ecosystem.

2. Current Feature Modules (MVP Progress)
Module 1 — User System

Status: Complete

Includes:
• Authentication (email/password)
• Profiles with username, bio, avatar
• Avatar upload via Supabase Storage
• Public profile pages at /u/:username
• Follow/unfollow system
• Dynamic follower/following counts
• Discover Users page
• Followers / Following pages
• Clean navigation layout

Module 2.1 — Artwork System

Status: Complete

Includes:
• art_pieces table
• Full CRUD for artwork
• Artwork bucket for uploads
• Create Artwork page with:
– Title
– Prompt
– Story
– Image upload
• AI Image Generation:
– User enters prompt
– Click “Generate with AI”
– Image is generated through Lovable AI
– Image auto-uploaded
• Public Artwork Detail page
• Global Gallery page showing latest artwork
• Artwork cards unified across the app

Module 2.2 — Collect System

Status: Complete

Database:
artwork_collections table with RLS allowing self-managed collections.

Rules:
• Users can Collect any artwork except their own
• If the logged-in user is the creator, show a Creator label instead of a Collect button
• Collection count is computed dynamically from the artwork_collections table (not stored)
• Collection count is hidden from public
• Collection count is visible only to the creator on their own artwork detail page

UI:
• Collect toggle on Gallery cards, Public Profile cards, and Artwork Detail
• When user has collected it → “Collected” badge
• When not → “Collect” button
• When creator → “Creator” badge

Module 2.3 — Unified “My Artwork” Page

Status: Complete

New route:
/my-artwork

Tabs inside:

Created Artwork

Collected Artwork

Replaces:
• Old "My Artwork" button inside Gallery
• Old “Collected Artwork” tab inside Profile

This is now consistent with how other users’ profiles display both sections.

3. Navigation Structure
Top Navigation

• Discover
• Gallery
• My Artwork
• Profile
• Logout (when logged in)

Public Pages

• Landing Page
• Gallery
• Public Profile /u/:username
• Artwork Detail /art/:id

Private Pages

• My Artwork /my-artwork
• Create Artwork /art/create
• Edit Artwork /art/edit/:id
• Profile Settings /me

4. Database Schema Overview (Human-Readable)
profiles

• id
• username
• bio
• avatar_url
• followers_count (dynamic)
• following_count (dynamic)

follows

• follower_id
• following_id
• created_at

Counts computed dynamically.

art_pieces

• id
• creator_id
• title
• image_url
• original_prompt
• story
• created_at
• updated_at

RLS ensures creators manage their own artwork.

artwork_collections

• id
• user_id
• artwork_id
• created_at

RLS ensures users can only collect/uncollect for themselves.

5. Content Rules
Artwork

• Minimum title length: 3 characters
• Story is optional
• Only the creator may edit or delete a piece
• AI-generated images follow prompt entered by user
• Only active pieces show in Gallery

Collect System Rules

• No self-collection
• Hidden collector counts (except for creator)
• Badge logic:
– Creator = “Creator”
– Collected = “Collected”
– Not collected = “Collect”

6. User Experience (UX) Overview
Gallery

• Infinite scroll or pagination
• Artwork cards show image, title, creator
• Collect button logic applied

Artwork Detail

• Large image
• Full story
• Prompt
• Creator badge if owner
• Collect button if viewer is not creator
• Collector count visible only to creator

Public Profile

• Avatar, bio, follow system
• Two sections: Created Artwork / Collected Artwork
• Horizontal spacing clean and consistent with Gallery layout

My Artwork

• Identical layout to Public Profile
• Tabs: Created / Collected

7. Branding and Naming

App name: ArtMint

Terminology:
Use artwork, NOT “piece” or “pieceMint.”
Use Collect, NOT “save,” “favorite,” or “like.”

Badge names:
• Creator
• Collected

8. What Comes Next (Future Modules)

These are not built yet but will be added later:

Module 3 — Competition System

• Weekly competitions
• Submission windows
• Fair rotation exposure
• Scoring system
• Creator rewards
• Collector rewards

Module 4 — Social Layer

• Activity feed
• Notifications
• Comments
• Sharing

Module 5 — Premium Features

• Premium minting
• Higher resolution AI generation
• Mintable “winning artwork”

9. Maintenance Guidance

Because ArtMint is now synced with GitHub + Lovable:

To update the spec

You can say:

“Update the human blueprint with the new feature: ______.”

I will open the file in GitHub and modify only the relevant sections.

To add new modules

Say:

“Add Module X.Y to the blueprint and create the Lovable prompt for it.”

10. Purpose of this Document

This specification serves as:

• A master reference for what the app currently supports
• A shared map for all future AI development
• A “truth source” for Lovable, GitHub Copilot, Bubble, or any other builder
• A protection against feature drift
• A reset button if the project ever breaks and needs a rebuild

This document ensures ArtMint stays consistent, scalable, and rebuildable from scratch at any moment.
