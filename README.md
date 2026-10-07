<h1 align="center">Mohan Singh</h1>

```http
HTTP/1.1 200 OK
Content-Type: application/developer+json
X-Role: Junior Software Developer
X-Based-In: Ontario, Canada
X-Backend: Java (Spring Boot), Python (FastAPI)
X-Frontend: React, TypeScript
X-Education: Sheridan College, Computer Programming (Honours)
X-Soundtrack: YouTube Music (queue: saved)
X-Auth: Required. Anonymous requests get a 401.
```

I'm a backend-leaning full-stack developer. I like finding out what a system does when things go wrong, ideally before a user does. The projects below are the evidence, and the bugs are the best part.

---

## 🎵 The one that's live

**[YTM Queue Saver](https://chromewebstore.google.com/detail/youtube-music-queue-saver/gnflndpgbhdpcgcmaegdecabbeacciic)** is a Chrome extension (Manifest V3) with a FastAPI backend. YouTube Music can't save your current queue, so one accidental click on a "recommended" track and your 40-song session is gone. This puts it back as a real playlist.

It's on the Chrome Web Store, and I use it myself.

| | |
|---|---|
| **Extension** | [`YtmQueueSaver-extension`](https://github.com/Mohan-2791/YtmQueueSaver-extension) (React, TypeScript, Vite, Tailwind) |
| **API** | [`YtmQueueSaver-backend`](https://github.com/Mohan-2791/YtmQueueSaver-backend) (FastAPI, PostgreSQL, Docker) |
| **Tests** | 79, all offline, zero network calls, zero API quota spent, run by GitHub Actions on every push |

### The number I care about most

YouTube gives a project **10,000 quota units a day**. Writing one track costs **50**.

```text
Restore a 100-track queue:   5,050 units
Restore it again:                0 units
```

The first version would have burned through the daily budget in two big restores. Playlist caching plus skipping tracks that are already there made the repeat restore free. A per-user and a global circuit breaker cover the rest.

### How it knows your queue got replaced

My first attempt listened for clicks on YouTube Music's play buttons. It broke within days, because their markup changes quietly and often. So it stopped watching *how* the queue changed and started comparing *what's in it*:

```ts
// simplified
const replaced = overlap(previousQueue, currentQueue) < 0.3
```

A queue that's just advancing keeps nearly all its track IDs. A new station shares almost none. Data doesn't get redesigned on a Tuesday.

---

## 🐛 Bugs I'm weirdly proud of

Every one of these broke something real. They're written up in more detail in the [engineering challenges](https://github.com/Mohan-2791/YtmQueueSaver-backend/blob/master/docs/engineering-challenges.md) doc.

**The `return` that doubled the bill.** Every restore came back `200 OK`. Nothing looked wrong. The only witness was the quota ledger. A `return` sat inside an `except` block, so a *successful* database commit fell through and created a second playlist. Fixed it, then wrote a test that asserts a repeat restore makes **no** YouTube call. A passing response doesn't prove correct behaviour. Testing that something *doesn't* happen does.

**Faster made it worse.** Large restores took 15 to 25 seconds, so I ran the inserts in parallel. YouTube answered with `409`s, and each retry cost another 50 units. The slow path was a symptom. The real fix was making fewer calls, not making them faster.

**The playlist that appeared twice.** A restore creates a brand-new playlist, so it isn't safe to retry. My generic fetch helper retried on timeout anyway, which turned slow successes into duplicate playlists. Retries are now off for that call, and the UI timeout (50s) is deliberately longer than the server's (45s), so the UI never gives up before the thing it's waiting on.

**The debounce that never fired.** YouTube Music mutates its DOM almost constantly during playback. A trailing debounce keeps getting postponed and may never run. A leading and trailing throttle guarantees it runs at least every `waitMs`.

**Audio that pauses when you look away.** Web players pause when they think the tab is hidden. A script in the page's MAIN world overrides `document.hidden` before the player's own listeners see it. It has to load via `src`, because YouTube Music's CSP blocks inline scripts.

**Auth that waved everyone through.** My own audit found a fallback that let requests with no `Authorization` header in as a default user. It's gone. Missing credentials now mean `401`. The dev-only `test-login` route returns `404` in production, not `403`, so it doesn't even confirm it exists.

**`primary_class=True`.** A SQLAlchemy typo that surfaced at import time inside a traceback pointing at library internals. It's `primary_key`. I now read my own column definitions like someone else wrote them.

---

## 🚗 In progress: RVS

**Recreational Vehicle Storage Management System**: Spring Boot, Spring Security, JPA/Hibernate, MySQL, React, TypeScript. Repos: [`rvs-backend`](https://github.com/Mohan-2791/rvs-backend) and [`rvs-frontend`](https://github.com/Mohan-2791/rvs-frontend).

It's a parking-lot app for RVs, and it's a good excuse to build real access control. Three roles (Admin, Employee, Client) see different slices of the same data, and Spring Security decides who gets what. The server makes that call, not the UI.

- JWT authentication and role-based access
- A 7-entity model: users, clients, vehicles, spaces, contracts, gate logs, inquiries
- Contract approval, space management, gate logging, admin reporting
- A public contact form rate-limited to 5 requests per hour per client, because I've read the internet
- React frontend: protected routes, dashboards, still being built

---

## 🧰 What I reach for

<p>
  <img src="https://skillicons.dev/icons?i=java,spring,py,fastapi,cs,dotnet,ts,react,tailwind,vite,mysql,postgres,docker,githubactions,maven,linux,git" alt="Java, Spring, Python, FastAPI, C#, .NET, TypeScript, React, Tailwind, Vite, MySQL, PostgreSQL, Docker, GitHub Actions, Maven, Linux, Git" />
</p>

Also: JWT, OAuth 2.0, AES-256-GCM, pytest, SQLAlchemy, and C and SQL when the course calls for it.

---

## 📝 Rules I now believe

1. **Test the thing that shouldn't happen.** A `200` isn't proof of anything.
2. **"Don't crash" and "don't fake it" are different goals.** Missing config should fail loudly, not quietly substitute something insecure.
3. **Fewer requests beat faster requests.**
4. **Hiding a button isn't security.** The backend has to say no too.
5. **Write down the limitation.** My backend README has a "Known Limitations" section. Pretending there are none is worse.

---

## 🍔 Day job

Crew member at Burger King while finishing a full-time diploma. Peak-hour inventory is capacity planning with fries: high traffic, no autoscaling, zero tolerance for running out of the thing everyone wants.

---

## 📬 Reach me

[LinkedIn](https://www.linkedin.com/in/mohan-singh-197379218/) · [Chrome Web Store](https://chromewebstore.google.com/detail/youtube-music-queue-saver/gnflndpgbhdpcgcmaegdecabbeacciic)

<sub>Last health check: 200. If you get a 502, the YouTube quota is probably gone for the day.</sub>
