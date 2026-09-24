# kubectl Training Ground

```
$ kubectl get pods -n bank-prod
NAME                          READY   STATUS             RESTARTS          AGE
ledger-api-7d4f9b2c1-kx8vq    0/1     CrashLoopBackOff   26 (14s ago)      3d2h
```

Two little terminals that teach you kubectl by breaking something and making you fix it.

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
| **[Moon Base: Dark Side](kubectl-moon-base.html)** | 18 missions · 15–20 min · beginner → low intermediate | You are the only one awake on the far side of the Moon. Life support is crash-looping, the Earth uplink is dead, and the supply window is 40 hours out. Nobody sabotaged anything — an automation did exactly what it was told. |

They teach the same commands from opposite directions. The Ledger is a whodunnit: follow
the trail to a person. Moon Base has no culprit at all — it is a triage problem, where the
cause is one label copied from the wrong manifest, and the ending depends on what you decide
the station can afford to switch off. Do the Ledger first, then Moon Base.

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

Plus the thing people actually get asked on a call: **why did it stop?**

| What you see | What it means |
|---|---|
| `Exit 0 · Completed` | Finished its job normally. Not a problem. |
| `Exit 1 · Error` | The app quit on an error. Read `logs --previous`. |
| `Exit 137 · OOMKilled` | Used more memory than its limit. The kernel killed it. |
| `Killing` / `ScalingReplicaSet` events | Stopped on purpose. Check the SOURCE column: a controller leaves its name there, a person running `kubectl` doesn't. |

## What they deliberately don't cover — yet

These are aimed at people who do basic admin and escalate anything serious. So the games are
read-only apart from `scale`, and the risky commands are answered with an explanation
instead of an outcome:

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

## Adding a case

`index.html` is a launcher. One object in the `CASES` array at the bottom of that file adds
a card:

```js
{
  file:'kubectl-your-case.html',
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
`story` (opening, cause, closing handover) and `logs` (what each app prints). Everything
below those — the parser, `describe`, tab completion, pipes, `watch` — is generic and
doesn't need touching.

Timings are worked out from the player's own clock, so "this started 97 minutes ago" is
always true, and every answer about time is checked against it.

## A note on what's in here

Nothing from any real employer, cluster, runbook or person. The banks, stations, hostnames,
registries, images, account numbers and people are invented. The only thing borrowed from
real life is which kubectl commands are worth teaching first.
