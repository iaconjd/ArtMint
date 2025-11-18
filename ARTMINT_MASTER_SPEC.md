🚀 PROMPT — FULL BUILD INSTRUCTIONS

You are rebuilding the ArtMint project from scratch OR updating an existing codebase.
Follow everything exactly.
Do not add features beyond this specification.

PART 1 — BRANDING & TERMINOLOGY

App name: ArtMint
Primary terminology:
– “Artwork” (never “piece”, “artpiece”, or “post”)
– “Collect” (never like/favorite/save)
– “Creator”
– “Collector”

PART 2 — DATABASE SCHEMA (Supabase)
2.1 Profiles
CREATE TYPE public.user_role AS ENUM ('creator', 'collector', 'both');

CREATE TABLE public.profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  username TEXT UNIQUE NOT NULL,
  role public.user_role NOT NULL DEFAULT 'both',
  premium BOOLEAN NOT NULL DEFAULT false,
  premium_expires_at TIMESTAMPTZ,
  profile_picture_url TEXT,
  bio TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Profiles readable by all" ON public.profiles FOR SELECT USING (true);
CREATE POLICY "Users insert own profile" ON public.profiles FOR INSERT WITH CHECK (auth.uid() = id);
CREATE POLICY "Users update own profile" ON public.profiles FOR UPDATE USING (auth.uid() = id);

2.2 Follows Table
CREATE TABLE public.follows (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  follower_id UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  following_id UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  CONSTRAINT unique_follow UNIQUE (follower_id, following_id),
  CONSTRAINT no_self_follow CHECK (follower_id != following_id)
);

ALTER TABLE public.follows ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Follows readable by all" ON public.follows FOR SELECT USING (true);
CREATE POLICY "Users can follow others" ON public.follows FOR INSERT WITH CHECK (auth.uid() = follower_id);
CREATE POLICY "Users can unfollow" ON public.follows FOR DELETE USING (auth.uid() = follower_id);

Follow Counts Are Computed Dynamically

No stored counters.
Use SQL aggregate queries.

2.3 Artwork Table — art_pieces
CREATE TABLE public.art_pieces (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  creator_id UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  title TEXT NOT NULL CHECK (char_length(title) BETWEEN 3 AND 80),
  image_url TEXT NOT NULL,
  original_prompt TEXT NOT NULL,
  story TEXT CHECK (story IS NULL OR char_length(story) <= 400),
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  is_active BOOLEAN NOT NULL DEFAULT true
);

ALTER TABLE public.art_pieces ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Artwork visible to all" ON public.art_pieces FOR SELECT USING (is_active = true OR creator_id = auth.uid());
CREATE POLICY "Users create own artwork" ON public.art_pieces FOR INSERT WITH CHECK (auth.uid() = creator_id);
CREATE POLICY "Users update own artwork" ON public.art_pieces FOR UPDATE USING (auth.uid() = creator_id);
CREATE POLICY "Users delete own artwork" ON public.art_pieces FOR DELETE USING (auth.uid() = creator_id);


Storage buckets:
– avatars
– artworks
Both must allow public viewing but restrict uploading/editing/deleting to the authenticated user only.

2.4 Collect System — artwork_collections
CREATE TABLE public.artwork_collections (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES public.profiles(id) ON DELETE CASCADE,
  artwork_id UUID NOT NULL REFERENCES public.art_pieces(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE(user_id, artwork_id)
);

ALTER TABLE public.artwork_collections ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Anyone can view collections" ON public.artwork_collections FOR SELECT USING (true);
CREATE POLICY "Users can collect" ON public.artwork_collections FOR INSERT WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Users can un-collect" ON public.artwork_collections FOR DELETE USING (auth.uid() = user_id);


Collector count is computed on demand, not stored.

PART 3 — ROUTES
Required pages:
Route	Purpose
/	Landing page
/login	Auth
/register	Auth
/me	Profile Settings
/u/:username	Public Profile
/discover	User discovery
/gallery	Gallery of all artwork
/art/create	Create Artwork
/art/:id	Artwork Detail
/art/:id/edit	Edit Artwork
/my-artwork	Unified page for Created + Collected artwork
PART 4 — UI REQUIREMENTS
4.1 Navbar

Items:
– Discover
– Gallery
– My Artwork
– Profile
– Log out
Left side: ArtMint brand name

4.2 Create Artwork Page

Fields:
– Original Prompt (textarea)
– “Generate with AI” button
– Image preview
– Title
– Story (optional)

AI generation uses AI Nano Banana model.

Upload created image to Supabase storage.

4.3 Gallery

Displays all active artwork:

Card UI includes:
– Image
– Title
– Story preview
– Creator username
– Creation date
– Collect button OR Creator badge

4.4 Artwork Detail Page

Must show:

– Image
– Title
– Story
– Creator
– Creation date
– Collect toggle (if not creator)
– Creator badge (if creator)

Hidden collector count:
– If currentUser.id === artwork.creator_id → show count
– Otherwise → hide

4.5 Public Profile Page (/u/:username)

Show:

– Avatar
– Username
– Bio
– Followers / Following
– Follow / Unfollow button
– Artwork section (creator’s artwork)
– Collected Artwork section (what they’ve collected)

4.6 My Artwork Page (/my-artwork)

Tabs:

Tab 1: Created Artwork
Cards identical to Gallery but no Collect button.

Tab 2: Collected Artwork
Cards identical to Gallery but:
– If collected item was created by YOU, show “Creator”
– If collected item was created by others, show the Collect toggle

This page replaces:
– Old "My Artwork" inside Gallery
– Old "Collected Artwork" inside Profile

PART 5 — COLLECT SYSTEM LOGIC
Collect Rules

You cannot collect your own artwork.

If artwork.creator_id === user.id
→ show “Creator” badge
→ hide collect button

Collector count is NOT visible publicly.

Collector count is ONLY visible on:
– The creator’s artwork detail page
– The creator’s own My Artwork tab (Created)

PART 6 — DESIGN SYSTEM
Theme

– Artistic premium
– Dark indigo + vibrant cyan
– Rounded cards
– Smooth hover animations
–consistent card grid across Gallery, Public Profile, and My Artwork

END OF PROMPT
