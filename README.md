# kubectl Training Ground

```
$ kubectl get pods -n bank-prod
NAME                          READY   STATUS             RESTARTS          AGE
ledger-api-7d4f9b2c1-kx8vq    0/1     CrashLoopBackOff   26 (14s ago)      3d2h
```

Three little terminals that teach you kubectl by breaking something and making you fix it.

No install. No cluster. No network. One HTML file per case — open it in a browser and
you are the on-call admin for the next twenty minutes. Every command you type is answered
by a simulated cluster that behaves like the real thing: pods get random names, restart
counts climb while you watch, new pods sit in `ContainerCreating` for a second before they
go `Running`, and events fade after an hour like they really do.

Start at **[index.html](index.html)**, or hand someone a single case file. Set it to your
editor's theme first if you like — there are five, and the choice follows you between cases.

---

## The cases

| | | |
|---|---|---|
| **[The Midnight Ledger](kubectl-midnight-ledger.html)** | 17 missions · 15–20 min · beginner | You are the night-shift admin at a bank. The ledger service is crash-looping, balances are frozen, and someone is skimming fractions of a cent off every transaction. Three suspects. One of them planted a false lead. |
| **[Moon Base: Dark Side](kubectl-moon-base.html)** | 18 missions · 15–20 min · intermediate | You are the only one awake on the far side of the Moon. Life support is crash-looping, the Earth uplink is dead, and the supply window is 40 hours out. Nobody sabotaged anything — an automation did exactly what it was told. |
| **[RobCo: Personality Crisis](kubectl-robco-personality.html)** | 17 missions · 20–25 min · proficient | Boston, October 2077. You're on call for RobCo Industries' Protectron fleet. 28 units are running the wrong personality: fire units are offering house fires a Nuka-Cola. Every pod says Running. The cause is one line in a config file, and you'll need grep to find it. |

They teach the same commands from opposite directions. The Ledger is a whodunnit: follow
the trail to a person. Moon Base has no culprit at all — it is a triage problem, where the
cause is one label copied from the wrong manifest, and the ending depends on what you decide
the station can afford to switch off. Do the Ledger first, then Moon Base.

RobCo is the step up. Nothing crashes, so `get pods` has nothing to tell you. The evidence is
buried in hundreds of log lines spread across several pods, one suspect looks guilty but never
ran, and the fix is a `rollout undo` that you have to justify first. There are fewer signposts
and more digging, and the digging is done with the same `grep`, `awk` and `sort | uniq -c` you'd
use on a real incident.

## What you actually practise

Everything below is standard kubectl, straight out of the public Kubernetes docs.

```
kubectl get namespaces                                  what compartments exist
kubectl get pods -n <ns>             (-A for all)       what is running
kubectl get pods <pod> -o wide       (-l app=…)         which node, or just one app
kubectl describe pod <pod> -n <ns>                      images, env, volumes, exit codes, events
kubectl describe pods -n <ns> | grep -i image: | sort -u        which version everything runs
kubectl logs <pod> -n <ns>           (--previous)       what it printed (before it died)
kubectl logs <pod> --since=1h --timestamps  (--tail=20) only the recent lines, with times
kubectl get events -n <ns>           (-o wide)          what happened, when it FIRST happened, and who did it
kubectl get deploy / sts -n <ns>                        what manages the pods
kubectl top pods -n <ns>             (-A)               what is actually using resources
kubectl scale deploy <name> --replicas=0 … then 1       stop it, bring it back clean
watch -n 1 kubectl get pods -n <ns>                     live view (Ctrl+C or Enter to stop)
kubectl config set-context --current --namespace=<ns>   stop typing -n
```

RobCo adds the proficient set:

```
kubectl get pods -A | grep -v Running                   everything that isn't healthy
kubectl logs deploy/<name> -n <ns> | grep -i warn       one pod's warnings, out of hundreds of lines
kubectl logs -l app=<name> -n <ns> --prefix --tail=-1   every pod of an app, each line labelled
… | grep -c · -v · -A5 · -B2 · -n · -o · -w             count, invert, context, line numbers
… | awk '{print $7}' | sort | uniq -c | sort -rn         tally a column
kubectl get rs -n <ns>  (-o wide)                       one per revision; AGE = when it rolled out
kubectl rollout history deploy/<name> (--revision=N)    what changed, why, and what it ran
kubectl get cm / describe cm / get cm -o yaml           read the config itself
kubectl rollout undo deploy/<name>                      back one revision (it becomes a new one)
kubectl rollout status deploy/<name>                    waits for it (Ready isn't the same as correct)
```

Plus the thing people actually get asked on a call: **why did it stop?**

| What you see | What it means |
|---|---|
| `Exit 0 · Completed` | Finished its job normally. Not a problem. |
| `Exit 1 · Error` | The app quit on an error. Read `logs --previous`. |
| `Exit 137 · OOMKilled` | Used more memory than its limit. The kernel killed it. |
| `Killing` / `ScalingReplicaSet` events | Stopped on purpose. Check the SOURCE column: a controller leaves its name there, a person running `kubectl` doesn't. |
| `ImagePullBackOff`, 0 restarts | The image can't be pulled. The container has never started, so it can't have done anything. |

## What they deliberately don't cover — yet

These are aimed at people who do basic admin and escalate anything serious. So the games are
read-only apart from `scale` (and, in RobCo, `rollout undo`), and the risky commands are
answered with an explanation instead of an outcome:

- `edit`, `apply`, `exec` — changes go live the moment you save
- `delete pods -l …` — one typo takes out the wrong pods
- scaling, restarting or deleting a `StatefulSet` — it holds data
- anything in `kube-system` or `monitoring`

Type one anyway. The game tells you what it would have done and why it's somebody else's
call, which is the actual lesson.

## Themes

Top right of every page: **Dark**, **Light**, **Monokai**, **Dracula**, **Nord**. Pick one and
it sticks, across the index and both games.

What doesn't change is the case's own colour — the bank stays goldenrod, the moon base stays
neon green — so a screenshot is still recognisably one game or the other whatever your editor
looks like. Every theme clears WCAG AA (4.5:1) on every piece of text, including the bits
where the authentic theme colour needed a nudge to be readable at body size.

Adding a sixth is one block of CSS variables in each file's `<style>`, plus one `<option>`.

## Handing it to someone

Send them the file. That's it — it's self-contained, works offline, opens on a phone, and
nothing they type leaves the page. The one thing a lone case file can't do is the **Home**
button in its top right, which looks for `index.html` beside it, so send the folder if you
want people moving between cases. Nobody needs cluster access to learn this, which is the
whole point: the first time someone runs `scale --replicas=0` shouldn't be in prod at 3am.

A full run is about 20 minutes. Hints are three deep per mission — a nudge, then a bigger
nudge, then the exact command — so nobody gets stuck long enough to quit.

## The report card

Every case ends with a report card, drawn below the handover:

- **missions** completed, and how long the run took
- **commands run**, split into kubectl commands and piped ones
- **hints used out of those available**. Each mission's hints count once, however many times
  you ask. A zero there means you never opened one.
- **answered first try** for the question missions. Only a genuinely wrong answer counts
  against you: "check the evidence first" and typing the answer in the wrong format don't.
- **field manual**: how many of the sidebar commands you actually used
- a strip with one square per mission: solid for no hints, half for some, hollow if the hint
  gave the answer away, and a red dot for a wrong first answer
- a rating stamp, based on hints and first-try answers (in the Ledger, a wrong accusation
  also costs the top rating)

There's a name field on the card, so a screenshot works as a record of who did the training and
when. Type `results` to bring the card back after `clear`, or partway through a run to see an
"in progress" version. **Copy summary** puts a plain-text version on the clipboard for
chat. The name and each case's personal best are remembered in the browser. None of it
leaves the page.

## Adding a case

`index.html` is a launcher. One object in the `CASES` array at the bottom of that file adds
a card:

```js
{
  file:'kubectl-your-case.html',
  key:'yourcase',            // same as CARD.key in the case: shows "Completed" + best run on the card
  difficulty:2,              // 1–3 bars on the card
  status:'ready',            // or 'wip': listed, with the link switched off
  accent:'#39ff14',          // the case's signature colour, used across its card
  accentLight:'#14710a',     // the same colour, darkened so it reads on the Light theme
  name:'Your Case',
  sub:'where · when · what kind of story',
  plot:'The hook, in two sentences.',
  missions:'18', time:'15–20 min', level:'Beginner',
  skills:['get pods -n','describe pod','logs --previous']
}
```

For the game itself, copy an existing case file and replace its world. The engine is the
same in both and it is sectioned with comment banners: `apps` (the container images),
`state` (namespaces, deployments, what's already broken before the player arrives),
`chapters` (the missions: a goal, three hints, an `ok()` that recognises the right command,
an optional `nudge()` for near-misses, and a `done()` that tells the next bit of story),
`CARD` (the report card's title, rating names and optional extra stat),
`story` (opening, cause, closing handover) and `logs` (what each app prints). Everything
below those — the parser, `describe`, tab completion, pipes, `watch` — is generic and
doesn't need touching.

For a piped command, `ok()` sees what the player actually ended up looking at: `e.out` is
the output after every `| grep`, and `e.greps` lists the patterns. Missions that are about
finding one line can check that the line is on screen, not just that the right command ran.
The RobCo file has the fullest engine (ConfigMaps, ReplicaSet history, `rollout undo`/`status`,
`logs -l`, `awk`, `cut`, grep context flags), so copy that one for anything proficient-level.

Timings are worked out from the player's own clock, so "this started 97 minutes ago" is
always true, and every answer about time is checked against it.

## A note on what's in here

Nothing from any real employer, cluster, runbook or person. The banks, stations, hostnames,
registries, images, account numbers and people are invented. The only thing borrowed from
real life is which kubectl commands are worth teaching first.

The one exception is borrowed from fiction: RobCo Industries, Protectrons, Nuka-Cola,
General Atomics and Mr. Handy come from Bethesda's *Fallout* series, and that case is an
unofficial fan tribute with no connection to Bethesda. In canon, Mr. Handy is a General Atomics
robot, which is why RobCo's fleet here is Protectrons. The calendar in that case always says
Friday 22 October 2077, whatever your clock says, and the handover books the follow-up for
Saturday morning. Fans will know how that goes.
