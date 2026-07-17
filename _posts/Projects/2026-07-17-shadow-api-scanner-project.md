---
title: "Shadow API Discovery & Vulnerability Scanner"
author: ekansh
date: 2026-07-17 14:20:00 +0530
categories: [Projects, HCIC2026]
tags: [python, owasp, api-security, bola, hcic]
published : true
---

> Repo : [github.com/ekansh-jaiswal/ShadowAPI-Scanner](https://github.com/ekansh-jaiswal/ShadowAPI-Scanner)
{: .prompt-info }

## What This Is :

This is my submission for HCIC-SI2026, Problem 15 : Shadow API Discovery & Vulnerability
Scanner. It's a Python CLI tool that finds "Shadow APIs" — endpoints that are quietly getting
real traffic but were never documented anywhere — and then actually checks whether they're
exploitable instead of just guessing.

No machine learning anywhere in this. Every single finding traces back to one plain rule you
can read directly in the code, which matters a lot for a security tool specifically. If
something gets flagged, you should be able to ask "why," and get a real answer, not "the model
was pretty confident."

## Shadow APIs vs Zombie APIs :

Worth being precise here since these two terms get thrown around loosely, and this project is
specifically about the first one.

A **Shadow API** is an endpoint that was never officially documented or reviewed in the first
place. A developer builds something quickly to hit a deadline, it goes live, nobody adds it to
the spec, nobody on the security side ever looks at it. Nothing malicious happened, it just
slipped in under the radar, and stays invisible to normal monitoring because as far as any
official record is concerned, it doesn't exist.

A **Zombie API** is a different problem that looks similar from the outside : an endpoint that
*was* officially documented at some point, but never got properly shut down when it should
have — like an old API version nobody decommissioned after an upgrade. It keeps running,
forgotten, still reachable, still carrying whatever weaknesses it had when it was retired.

The difference matters for detection : Shadow APIs need to be *discovered* (traffic going
somewhere the docs don't mention), Zombie APIs need to be *tracked over time* (something that
used to matter quietly going stale while still technically alive). This project only tackles
Shadow APIs.

## The Problem, Concretely :

Digital health systems (think India's ABDM-style setups) expose a lot of web APIs, and over
time some of them get forgotten. An old debug route nobody removed. A "temporary" endpoint
that made it to production and just stayed there. These are dangerous exactly because nobody
remembers they exist — if nobody knows about it, nobody's watching it, even if it's still
handling patient data.

## How It Works :

The pipeline runs through four stages. Every single one of them is a plain yes/no or a fixed
formula, nothing fuzzy anywhere in it.

![Shadow API Scanner system flow](/assets/img/shadowapi-system-flow.svg)
_Fig. 1 — how data actually moves through the scanner, end to end_

**Stage 1 — is it in the spec?** For every endpoint the logs show real traffic for, the tool
checks one thing : does this exact path (after normalizing IDs, so `/patient/104` and
`/patient/117` count as the same path, `/patient/{id}`) appear in the OpenAPI spec? If yes,
Documented OK. If no, it's Shadow, and moves to Stage 2. Plain set comparison.

**Stage 2 — run the 7 checks.** Every Shadow endpoint gets checked against all 7 rules based
on the OWASP API Security Top 10 (2023). Each check either fires or it doesn't, no partial
match, no confidence percentage :

- **Sensitive Path or Parameters** — does the URL contain a keyword like `patient` or
  `aadhaar`? (API3:2023, Excessive Data Exposure)
- **Missing / Inconsistent Authentication** — does the spec say auth is required, and did at
  least one real request in the logs reach the endpoint without one? (API2:2023, Broken
  Authentication)
- **Excessive Data Exposure** — does the response leak fields it shouldn't (SSNs, diagnosis
  codes, more than the endpoint's stated purpose needs)? (API3:2023)
- **Rate Limit Absence** — does the endpoint get hammered with no throttling in place?
  (API4:2023, Unrestricted Resource Consumption)
- **Undocumented HTTP Method** — is it being called with a method the spec never listed?
- **Shadow API Endpoint** — the routine flag every undocumented path gets, just for showing
  up here at all (API9:2023, Improper Inventory Management)
- **BOLA (Cross-User Access)** — the one I actually care about the most, explained below

![Decision flow: raw request to severity score](/assets/img/shadowapi-decision-flow.svg)
_Fig. 2 — how a raw request becomes a labeled endpoint, then a severity score_

**Stage 3 — add up the weight.** Each check that fires adds a fixed number of points based on
its severity : CRITICAL adds 40, HIGH adds 25, MEDIUM adds 10, LOW adds 5, INFO adds 1. Summed
per endpoint, capped at 100 if it goes over (common, since a real BOLA finding usually
triggers several checks together).

**Stage 4 — turn the score into an action.** The 0-100 score gets bucketed into a severity
band, and that decides whether the tool blocks a deployment. `--fail-on critical,high` (the
default) means any endpoint landing in those bands makes the whole scan exit with a failure
code — exactly what a CI/CD pipeline would check before letting new code go live.

## What BOLA Actually Is :

**Broken Object Level Authorization** is currently API1:2023 on the OWASP API Security Top
10, and it's honestly one of the simplest vulnerabilities to understand and one of the most
common to find in the wild. It happens whenever an API checks *that* you're logged in, but
forgets to check *whether the specific object you're asking for actually belongs to you*.

So say an endpoint is `/api/v1/patients/{id}`. A properly built version checks : is this
token valid, AND does this token's owner actually have permission to see patient `{id}`? A
broken version only checks the first half — is this token valid — and then just returns
whatever `{id}` was asked for, regardless of who's asking. Swap the ID in the URL, and you're
looking at someone else's data using your own perfectly legitimate login.

The scanner doesn't try to guess this from traffic shape. It actually does it — it takes one
real user's token, and sends a real request trying to fetch a *different* real user's data
with it, and only marks something CRITICAL if that request genuinely succeeds and returns
real data back. No prediction, no pattern matching, just doing the thing and looking at what
comes back.

## Avoiding False Alarms — The Ownership Problem :

Here's the tricky part about BOLA specifically, and it's the thing that made this project
actually hard instead of a five-minute regex exercise : not every endpoint with a number in
the URL is protecting someone's private data.

`/api/v1/patients/{id}` is ownership-scoped — a specific patient's ID that only that patient
(or their doctor) should be able to look up. But `/api/v1/doctors/{id}` looks identical
structurally and isn't — looking up any doctor by ID is completely normal, there's no "owner"
of a doctor's public directory listing to protect. An early, naive version of this check fired
the cross-user probe on *any* endpoint with an ID in the path, which meant it wrongly flagged
the public doctor listing as broken. A scanner that keeps crying wolf isn't much better than
no scanner at all, so getting this right mattered as much as catching the real bugs.

## Testing It Against My Own Fake Health Gateway :

Built a mock API called "SwasthyaConnect" with five endpoints deliberately left undocumented,
just to have something safe to point the tool at first.

```bash
python scanner/cli.py \
    --log-file mock_env/access.log \
    --spec mock_env/openapi_spec.yaml \
    --mock-server-url http://localhost:8000 \
    --output report.html \
    --fail-on critical,high
```

```
Score  Level      Path
-----  ---------  --------------------------------------------
  100  CRITICAL   /api/v1/internal/debug/patient/{id}
  100  CRITICAL   /api/v1/patient-records/{id}
  100  CRITICAL   /api/v1/patients/{id}/insurance-claims
   66  HIGH       /api/v1/patients/{id}
   61  HIGH       /api/v1/otp/verify
   20  LOW        /api/v1/appointments/{id}

Overall Risk Exposure : CRITICAL (100/100)
Exit 1 -- CRITICAL findings present
```

Found all five shadow endpoints, no wrong guesses, and proved three real BOLA bugs live —
each one backed by a real captured request/response showing one fake patient's data (Aadhaar
number, diagnosis, prescription) returned using a *different* fake patient's login.

## Bugs I Actually Hit Building This :

This is the part I actually enjoyed writing down, since it's the difference between a demo
that only ever shows the happy path and something that got poked at until it broke.

**Silent probe failure.** While testing by hand, I killed the mock server mid-scan just to
see what would happen, and the scanner kept returning the exact same CRITICAL results as when
the server was up. Which is a genuinely bad failure mode for a tool whose entire pitch is "we
checked this for real, we didn't just guess" — an exception inside the probe code was being
silently swallowed. Fixed it by adding a real reachability check at startup : if the target
can't be reached, the scanner shows a clear warning, falls back to passive-only findings, and
the score actually drops to match (100/CRITICAL down to 61/HIGH in testing) instead of
quietly repeating stale results.

**The doctor-listing false positive**, already described above — fixed by only running the
cross-user check on endpoints that structurally look ownership-scoped.

**Severity rank-comparison bug.** `--fail-on medium` was failing to catch CRITICAL results,
because the exit-code logic was checking for an *exact* severity match instead of "is this at
least as serious as medium." Caught this while deliberately running the exit-code behaviour
across six different scenarios, not just the case where everything goes right.

## Validating Against Something I Didn't Build : OWASP crAPI

Testing only against my own mock server doesn't really prove much on its own — of course a
tool finds bugs in a fixture built around its own assumptions. So I ran it against OWASP's
**crAPI** (Completely Ridiculous API), a real, independently built, intentionally vulnerable
application.

Created two separate accounts through crAPI's actual signup flow, each verified through its
MailHog inbox, each added a vehicle. Captured real traffic with `mitmproxy` sitting in front
of crAPI — nothing hand-picked, no fake logs written by hand. Converted that capture into the
Nginx-format log the scanner expects, and hand-wrote a partial OpenAPI spec covering only what
a reasonable API consumer would expect to be officially documented.

Worth explaining why that spec has to come from an independent source rather than just being
generated from the same traffic being scanned : if it were, every endpoint the scanner ever
saw would automatically "match the spec," since the spec would just be a description of
whatever already happened. Nothing could ever get flagged as undocumented. The whole design
depends on the spec being a separate source of truth from the traffic being audited.

**The generalization bug.** The first automated run against crAPI correctly flagged
`/identity/api/v2/vehicle/{id}/location` as Shadow, but the BOLA probe didn't fire on it. The
ownership-scope check was using a small hardcoded keyword list built around my own healthcare
test data (`patient`, `appointment`, `prescription`) — it had no idea what to do with a
vehicle-management app's vocabulary. Rather than just adding "vehicle" to the list, which
would only fix this one case, I rebuilt the check to detect ownership scoping structurally :
any path containing an instance identifier — a numeric ID or a UUID — is treated as a
candidate for the BOLA probe by default, with a documented exclusion list for known public
reference data.

**Confirmed live, fully automated.** After the fix, the scanner's own CLI — using only the
vehicle UUIDs it pulled itself out of the captured log, nothing supplied by hand — ran the
probe and confirmed it :

```
URL:    http://localhost:8888/identity/api/v2/vehicle/fadcf145-85a5-4967-9691-44f75706a026/location
Status: 200 OK — unauthorized access confirmed
Response: {'carId': 'fadcf145-85a5-4967-9691-44f75706a026', 'vehicleLocation':
{'id': 3, 'latitude': '37.746880', 'longitude': '-84.301460'}, 'fullName': 'User A',
'email': 'usera@shadowscan.test'}
```

User B's own login token, User A's real vehicle location, real GPS coordinates and email
handed straight back. This matches OWASP's own documented crAPI Challenge 1 exactly, which
confirms two things at once : this is a known, real vulnerability class, and the scanner's
own detection logic — not a manually crafted request pretending to be automated — actually
finds it.

**One more bug, found while double-checking that fix.** Re-running the CLI against crAPI
gave a confusing result at first — the risk engine was correctly returning a CRITICAL BOLA
finding when I called the function directly, but the endpoint's label in the actual report
still showed HIGH. Traced it down to the scorer : the severity-band logic buckets purely by
summed weighted score, and an endpoint carrying one CRITICAL finding (40 points) alongside an
unrelated MEDIUM finding (10 points, the routine "Shadow API Endpoint" flag) summed to 50,
landing in the 50-74 HIGH band. The CRITICAL finding wasn't lost or wrong, it was just diluted
by an unrelated lower-severity finding sitting on the same endpoint.

The fix : an endpoint's final risk level is now the more severe of (a) its summed-score band,
and (b) the single highest individual finding severity actually present on it. Re-ran against
both crAPI and the mock environment afterward — crAPI now correctly shows CRITICAL, nothing
in the mock environment regressed, and one more endpoint that had been quietly sitting under
the same masking bug (`/api/v1/patients/{id}`) also got correctly promoted.

## What Else the Scanner Actually Catches :

Beyond BOLA, the other 6 checks cover a decent chunk of the practical, everyday API Top 10
issues that show up in real deployments — missing auth on endpoints that should have it,
sensitive keywords sitting in a URL path where they're easy to scrape, response bodies
returning more fields than the endpoint's job needs, no rate limiting on something that
clearly should have one, and HTTP methods being accepted that were never documented (a DELETE
route quietly existing on something the spec only lists as GET, for instance). None of these
need a live network probe the way BOLA does — they're derivable from the logs and the spec
diff alone, which is also why they're cheaper to run and safe to leave on even in a read-only,
`--no-active-probes` pass.

## DPDP Act Notes :

Since this is aimed at Indian digital health systems, findings involving personal or health
data carry a short note pointing at relevant sections of India's DPDP Act, 2023 — Section 6 on
limiting what data gets collected and why, Section 8(1) on keeping data reasonably secure, and
so on. Wrote these carefully as "relevant to" or "may be worth reviewing," never "this breaks
the law." The tool points out technical facts. It's not a lawyer and doesn't try to make legal
calls.

## Closing Thought :

Wrote this mostly to have something concrete to show for the HCIC submission, but the actual
useful part of the exercise for me was the debugging, not the feature list — chasing the
silent-probe bug, the false positive, the scoring dilution bug. All three were things that
looked fine on the surface and only broke under a slightly adversarial test (kill the server
mid-scan, point it at something it wasn't built around, run the exit code across every
`--fail-on` combination instead of just the default). That's probably the actual lesson to
take away from this one, more than the specific tool.

Full setup instructions and the complete crAPI validation writeup are in the
[repo](https://github.com/ekansh-jaiswal/ShadowAPI-Scanner) itself.
