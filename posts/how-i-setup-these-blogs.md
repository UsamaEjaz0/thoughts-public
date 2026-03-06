My first blog is about how I set up these blogs. The whole thing took about 10 minutes.

## The problem
I wanted a place to write. Not a full-blown CMS. Not a WordPress site. Not a Medium account where someone else owns my content. Just a simple setup where I write markdown files, push them to GitHub, and they show up on my site. No databases. No admin panels. No nonsense.

## The stack
I'm using Lovable for the frontend and Supabase Edge Functions as a lightweight server-side proxy. The blog content itself lives in a public GitHub repo as plain markdown files.

The entire thing, including building the site, took about 10 minutes using Lovable. I described what I wanted, it generated the frontend, I wired up two edge functions, and it was done. No boilerplate. No scaffolding. No hours spent configuring build tools. Ten minutes from nothing to a live blog.

Here's how the pieces fit together.

## Edge functions
There are two edge functions. Both are dead simple.

**blog-list** fetches `posts/index.json` from the GitHub repo and returns it as JSON with caching headers. This file is the source of truth for all blog metadata: slug, title, date, and summary.

**blog-post** accepts a slug, fetches `posts/<slug>.md` from the repo, and returns the raw markdown along with the slug. It validates that the slug exists and caches the response.

Both functions support an optional `GITHUB_TOKEN` secret. Without it, GitHub's API allows 60 requests per hour for unauthenticated calls. That sounds like a lot until a few people visit your blog at the same time and each page load triggers a fetch. With the token, you get 5,000 requests per hour, which is more than enough for a personal blog.

The edge functions also set caching headers so repeated requests for the same content don't hit GitHub at all. The browser caches the response, and subsequent visits are served instantly. Between the token and the caching, GitHub's rate limits become a non-issue.

That's it. No database. No ORM. No migrations. Two functions, each doing one thing.

## The index file
The entire blog list is driven by a single JSON file in the repo:

```json
[
  { "slug": "how-i-setup-these-blogs", "title": "How I setup these blogs", "date": "2026-03-05", "summary": "Short summary" }
]
```

When I want to publish a new post, I add an entry here and push the markdown file. That's the whole publishing workflow.

## The frontend
The frontend calls the edge functions via the Supabase client with retry logic built in.

`/blog` lists all posts sorted newest-first, showing title, date, and summary. `/blog/:slug` fetches the markdown and renders it with the title and date pulled from the blog list. Both pages have retry on error.

The homepage has a BlogSection component that shows the 5 most recent posts with a "View all" link. Nothing fancy.

## Why this works
There's no build step for content. I write a `.md` file, update `index.json`, push to GitHub, and it's live. The edge functions handle caching so the site is fast. GitHub handles storage and version control so I get history for free. If I ever want to move, my content is just files in a folder.

The whole thing is held together by a JSON file and two functions. Sometimes the best architecture is barely any architecture at all.

The repo is public if you want to see how it's structured: [github.com/UsamaEjaz0/thoughts-public](https://github.com/UsamaEjaz0/thoughts-public).
