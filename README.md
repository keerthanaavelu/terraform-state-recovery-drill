💡 Why This Exists

State drift and corruption aren't edge cases — they happen in every real environment. Most engineers only read about terraform import and state mv. This repo documents actually breaking things on purpose and recovering from them, using the dev environment from terraform-envs.

🔥 The Drills
#	Scenario	Detection	Fix	Outcome
1	Resource deleted manually in Azure	terraform plan showed 2 resources to recreate	terraform apply	Clean recreation, no dependent disruption
2	Resource created manually in Azure	N/A — resource existed outside state	terraform import	Adopted into management, zero disruption
3	Resource renamed in .tf config	terraform plan showed 1 destroy + 1 create for the same resource	terraform state mv	Avoided unnecessary destroy/recreate

Full command-by-command walkthrough → RUNBOOK.md

🧠 Key Takeaways
Drift is inevitable — the skill is recovering without unnecessary disruption, not avoiding it entirely.
apply is for restoring what should exist. import is for adopting resources that already exist outside Terraform. state mv is for correcting Terraform's bookkeeping when the real resource never changed.
Import commands are case-sensitive on resource ID segments (resourceGroups, not resourcegroups) — a small gotcha that costs real time if you don't know it.
Never apply a plan showing an unexpected destroy — check why first. A destroy/create pair for a resource you just renamed is a state mv fix, not an apply.
🛠️ Tools Used

terraform plan · terraform apply · terraform import · terraform state mv · terraform state list

Environment: terraform-envs · Module: terraform-azure-vnet-module
