# GreenChit — Trade-offs and Design Review

## Setup
**Option A**: Azure App Service monolith — the GreenChit API (claim management, receipt handling, CSV export, audit logging) and the Service Bus notification consumer all run as a single ASP.NET Core application on one App Service instance. One codebase, one deployment, one process.
**Option B**: Azure Container Apps split — the GreenChit API runs as one container and the Service Bus notification consumer runs as a separate container. Both sit in the same Container Apps environment, sharing networking and managed identity, but each deploys, restarts, and scales independently.
- Quality attributes scored 1 (worst) to 5 (best). Scores are pre-set; justifications are mine.

## Trade-off Table

| Quality attribute | Option A: App Service monolith | Option B: Container Apps split | Why |
|---|---|---|---|
| Time-to-first-deploy | **5** | **2** | App Service is a fully managed PaaS platform that lets developers deploy web applications without managing containers. You push code, it runs. Container Apps requires the team to write a Dockerfile, set up Azure Container Registry, and build a CI/CD image pipeline before the first deploy. For a team new to containers, this adds days to initial setup. |
| Cost (low spend) | **5** | **2** | App Service B1 Linux tier costs a flat ~$55/month .For a low-traffic internal tool with ~100 users, App Service's flat rate is simpler. Container Apps' consumption billing can be unpredictable during development and testing when developers trigger frequent rebuilds and restarts. |
| Operability for 10-person team | **4** | **3** | App Service is frequently preferred by developers due to its simplicity — it removes underlying complexity through abstraction, letting developers focus on building applications. Container Apps adds container concepts like images, registries, and revisions. Microsoft's own documentation on Container Apps notes it requires understanding of containerization and orchestration concepts. For a small team, that is extra knowledge App Service does not require. |
| Independent deploy | **1** | **5** | In the App Service monolith, the API and notification consumer are one deployment unit. Changing notification logic means redeploying the entire API, risking downtime for an unrelated change. Container Apps lets each container deploy independently with separate revision histories. The team can update the notification consumer without touching the API. For a system with a 99.9% business-hours SLA handling financial claims, this separation reduces deployment risk. |
| Future scaling | **2** | **5** | App Service scales the entire application as one unit — you cannot scale the notification consumer independently of the API. Container Apps scales each container separately using autoscaling, supporting triggers based on HTTP requests, TCP connections, CPU, memory, and custom metrics like Service Bus queue depth. GreenChit does not need this today at ~100 users, but Container Apps handles growth without re-architecture. App Service would require splitting the codebase later — a costly rewrite. |
| Authn/authz consistency | **4** | **3** | Both services provide built-in Easy Auth for Entra ID authentication with minimal code . However, Easy Auth is implemented differently across the two services — the authentication headers vary, and .NET applications on Container Apps do not recognise Easy Auth by default, requiring custom middleware to map claims correctly. App Service's Easy Auth integration with .NET is more mature and works out of the box, giving it a slight edge for auth consistency. |
| **Total** | **21** | **20** | |

## Results Summary

| Metric | Target | Achieved |
|---|---|---|
| Quality attributes scored | 6 | 6 |
| Cells with a written justification | 12 | 12 |
| Decision-affecting attributes identified | 2–3 | 2 |

## Decision and rationale

We choose **Option A: App Service monolith**. It scores 21 to 20, and the attributes it wins on are the ones that matter most for GreenChit.

The two attributes that drove this decision are **cost** and **operability**:

- **Cost (5 vs 2):** GreenChit is an internal tool for ~100 staff. At this scale, App Service at a flat monthly rate is hard to beat. Container Apps introduces registry costs, ingress fees, and per-second billing complexity that are not justified for a low-traffic internal tool. Every dollar spent on infrastructure for GreenChit is a dollar not spent on something that serves clients.

- **Operability for a 10-person team (4 vs 3):** The development team is small. App Service is deploy-and-forget — push code, it runs. No Dockerfiles, no image builds, no registry management, no revision tracking. The team spends time building features, not managing containers.

- **Time-to-first-deploy (5 vs 2)**: Container Apps requires writing Dockerfiles, configuring Azure Container Registry, and building an image pipeline before anything can run. App Service requires none of this. For a design being handed off to a development team, a simpler first deployment means faster time to a working system.


---

## Design review feedback (received from another pair)


<!-- The reviewing pair identified that the B1 App Service tier chosen in ADR-0002 does not support staging. For a financial claims system with a 99.9% business-hours SLA, deploying directly to production without a staging environment is a risk , live users are affected immediately if something breaks. They recommended upgrading to the S1 Standard tier (~$74/month), which includes 5 deployment slots, allowing the team to test changes in staging before swapping to production. 
However, the option of deploying outside business hours exists. The SLA only covers 08:00–19:00 Mon–Fri. If the team always deploys at 7pm Friday, they have the weekend to catch issues before anyone uses the system Monday morning. -->