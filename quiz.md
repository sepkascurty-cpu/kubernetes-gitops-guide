# 🧠 GitOps Quiz

Five questions. Answers are collapsed — no peeking until you've committed to one.

---

### 1. What is the core difference between push-based and pull-based deployments?

<details>
<summary><b>Answer</b></summary>

In <b>push-based</b> deployments, the CI pipeline holds cluster credentials and runs <code>kubectl apply</code> <i>into</i> the cluster. In <b>pull-based</b> (GitOps) deployments, an agent <i>inside</i> the cluster watches Git and pulls the desired state itself. The CI pipeline never touches the cluster — which eliminates credential sprawl and makes every change traceable to a commit.

</details>

---

### 2. At 2 AM someone runs `kubectl scale deploy/demo-web --replicas=1` to "fix" a noisy-neighbor issue. It's 9 AM and the deployment is back at 3 replicas. What happened?

<details>
<summary><b>Answer</b></summary>

The GitOps agent's reconciliation loop detected <b>drift</b>: the actual state (1 replica) no longer matched the desired state in Git (3 replicas). Because the Application has <code>selfHeal: true</code>, the agent automatically reverted the manual change. The correct fix was a commit changing the manifest, not a <code>kubectl</code> command.

</details>

---

### 3. A developer deletes the Service manifest from the Git repo and merges the PR. What does `prune: true` do, and what would happen without it?

<details>
<summary><b>Answer</b></summary>

With <code>prune: true</code>, the agent deletes the Service from the cluster because it no longer exists in the desired state. <b>Without</b> prune, the Service would become an orphan — still running in the cluster, invisible to Git, and a source of silent drift. Prune keeps Git and the cluster strictly 1:1.

</details>

---

### 4. How do you roll back a bad deployment in a GitOps workflow?

<details>
<summary><b>Answer</b></summary>

<code>git revert</code> the offending commit and push. The agent detects the revert as a new desired state and syncs the cluster back. No re-running old pipelines, no manual <code>kubectl rollout undo</code> — the rollback is itself a reviewed, auditable commit.

</details>

---

### 5. Why shouldn't application secrets (API keys, DB passwords) be committed to the Git repo, and what are two GitOps-friendly alternatives?

<details>
<summary><b>Answer</b></summary>

Git repos are widely readable (clones, PRs, logs) — committing secrets leaks them permanently into history. Two GitOps-friendly alternatives: <b>Sealed Secrets</b> (encrypt secrets into the repo; only the in-cluster controller can decrypt) or <b>External Secrets Operator</b> (store a <i>reference</i> in Git; the operator fetches the real value from a vault like AWS Secrets Manager at sync time). SOPS with age/PGP keys is a third option.

</details>

---

Scored 5/5? You're ready to delete your pipeline's kubeconfig. 🔥
