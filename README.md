# Coder Fleet

## Executive summary

Coder Fleet is a status board for a fleet of coding-agent workspaces on <img src="https://cdn.simpleicons.org/coder" width="16" alt=""> [Coder](https://coder.com): it shows which workspaces are running, which agent is busy or idle, and what each one did last, without a command line. Workspaces are plain Coder workspaces that run the <img src="https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/opencode.svg" width="16" alt=""> [OpenCode](https://opencode.ai) coding agent against models served through the <img src="https://github.com/BerriAI.png?size=36" width="16" alt=""> [LiteLLM](https://www.litellm.ai) gateway, and a task is handed to one idle workspace at a time. The board is read only: it reports the fleet and never changes it.

- **Fleet** lists one row per workspace with a green or red mark, its build, whether its agent is connected, busy or idle, its health, when it was last used and its version, with a summary count above the table.
- **Activity** shows each agent's state (working, idle, stuck, error or offline) in plain words, its current task, its last action and its recent tool calls, with secrets and internal names removed before anything is shown.
- **Logs** shows recent log lines for the Coder service, the board itself and the dispatch service from <img src="https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/victorialogs.svg" width="16" alt=""> [VictoriaLogs](https://victoriametrics.com/products/victorialogs/), redacted the same way.

## In action

**Check the fleet, then open an agent's recent work.** The page refreshes itself with no refresh button. (Recorded on demo data.)

![Coder Fleet showing the workspace fleet, agent activity and logs](assets/coder-fleet-demo.gif)

## Architecture

Coder Fleet is one web page and a small API running together on a self-hosted <img src="https://cdn.simpleicons.org/kubernetes" width="16" alt=""> [Kubernetes](https://kubernetes.io) cluster. The API holds no Coder credential: it asks the cluster's tool gateway, which talks to Coder and to the dispatch service.

- **Data pipeline.** The API reads workspaces and agent activity through the <img src="https://cdn.simpleicons.org/modelcontextprotocol" width="16" alt=""> [Model Context Protocol](https://modelcontextprotocol.io) gateway and reads logs from <img src="https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/victorialogs.svg" width="16" alt=""> [VictoriaLogs](https://victoriametrics.com/products/victorialogs/). A <img src="https://cdn.simpleicons.org/redis" width="16" alt=""> [Redis](https://redis.io) cache keeps several open tabs on one shared copy.
- **AI.** The agent workspaces run <img src="https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/opencode.svg" width="16" alt=""> [OpenCode](https://opencode.ai) and call open-weight models through <img src="https://github.com/BerriAI.png?size=36" width="16" alt=""> [LiteLLM](https://www.litellm.ai).
- **Web UI.** A <img src="https://cdn.simpleicons.org/react" width="16" alt=""> [React](https://vite.dev) page behind <img src="https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/caddy.svg" width="16" alt=""> [Caddy](https://caddyserver.com), served with a <img src="https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/fastapi.svg" width="16" alt=""> [FastAPI](https://fastapi.tiangolo.com) backend in the same pod. Code arrives from the git repository through <img src="https://cdn.simpleicons.org/git" width="16" alt=""> [git-sync](https://github.com/kubernetes/git-sync), so a push updates the page without rebuilding an image.
- **Deploy and infrastructure.** <img src="https://cdn.jsdelivr.net/gh/homarr-labs/dashboard-icons/svg/argo-cd.svg" width="16" alt=""> [Argo CD](https://argo-cd.readthedocs.io) keeps the page and the workspace count (a replica setting in one YAML file) in step with git, and it deploys the <img src="https://cdn.simpleicons.org/modelcontextprotocol" width="16" alt=""> [Model Context Protocol](https://modelcontextprotocol.io) tools the agents use.

![Coder Fleet architecture: operator browser, status page and API, tool gateway, Coder agent workspaces and delivery](assets/coder-fleet.png)

## License

PolyForm Noncommercial 1.0.0 with a required notice from Haya Tech LLC. Noncommercial use is free; any commercial use, including resale, hosting for others, or a paid product, needs a written license from Haya Tech LLC first. See [LICENSE](LICENSE).
