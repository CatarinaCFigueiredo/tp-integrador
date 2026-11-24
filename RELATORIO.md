# Relatorio de evidencias - Trabalho Pratico Integrador 01-06

Nome: Catarina Da Costa Figueiredo   Data: 24/11/2026
URL do repositorio Git: ____________________________

## E01 - Ambiente (1 valor)
Comandos e versoes observadas:
- `docker --version`: Docker version 29.9.3, build 5457199
- `docker compose version`: Docker Compose version v5.7.0
- `git --version`: git version 2.53.0
- `docker info`: Daemon Docker ativo (Docker Desktop / WSL2 Linux, Server Version 29.9.3).
Identificação dos quatro componentes principais do projeto:
- API: `app.py` (com dependências em `requirements.txt`)
- Testes: `tests/test_app.py` (com dependências em `requirements-dev.txt`)
- Compose: `compose.yaml` (e overlay `incidente.yaml`)
- Pipeline: `Jenkinsfile`

Conclusão: O ambiente local dispõe de Docker, Docker Compose V2 e Git funcionais com o daemon Docker ativo, e os quatro componentes base do projeto foram corretamente identificados na raiz do kit.

## E02 - Dockerfile e container (3 valores)
Tag da imagem; docker ps; HTTP /health; utilizador efetivo do container:
- **Tag da imagem:** `natixis-support:1.0`
- **Mapeamento de portas (host -> container):** `127.0.0.1:18080->8000/tcp` (`docker ps`: container `tp-api` em estado `Up`)
- **HTTP `/health`:** `HTTP/1.1 200 OK` com corpo `{"status":"ok","service":"support-api","version":"1.0"}`
- **Utilizador efetivo (`docker exec tp-api id`):** `uid=1000(appuser) gid=1000(appuser) groups=1000(appuser)`
- **Linha relevante dos logs (`docker logs --tail 15 tp-api`):** `INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)` e `INFO:     172.17.0.1:53912 - "GET /health HTTP/1.1" 200 OK`

Conclusão: A imagem Docker foi construída otimizando a cache de dependências com `.dockerignore` configurado, e o container executa a API de forma segura sob o utilizador não privilegiado `appuser` com resposta HTTP 200 válida.

## E03 - Compose, DNS, healthcheck e volume (4 valores)
Resultado de docker compose config; health; teste checker -> api; contador antes/depois da recriacao:
- **Validação do Compose (`docker compose config --quiet`):** Sem erros de sintaxe (saída vazia com código 0).
- **Estado do serviço (`docker compose ps`):** `natixis-integrador-api-1` em estado `Up ... (healthy)`.
- **Teste DNS e dependência (`checker -> api`):** `docker compose --profile diagnostico run --rm checker -fsS http://api:8000/health` aguardou pelo estado `healthy` da `api` e devolveu `{"status":"ok","service":"support-api-integrador","version":"1.0"}`.
- **Contador antes da recriação:** `{"visits":0}` inicial, incrementado para `{"visits":1}` via `POST /visits`.
- **Contador após `--force-recreate`:** `{"visits":1}` (estado `healthy` restabelecido).

Conclusão: O contador não foi perdido porque o ficheiro `visitas.txt` é gravado na diretoria `/data` associada ao volume Docker nomeado (`app-data`), cujo ciclo de vida e dados persistem de forma independente da recriação do container.
## E04 - Incidente Docker (3 valores)
Sintoma, comandos de investigacao, causa-raiz, alteracao realizada, prova da recuperacao e contador preservado:
- **Sintoma:** O container `natixis-integrador-api-1` ficou em estado `(unhealthy)` após a aplicação do overlay `incidente.yaml`, tornando a API inacessível na porta `18080`.
- **Comandos de diagnóstico:** `docker compose -f compose.yaml -f incidente.yaml ps`, `docker compose -f compose.yaml -f incidente.yaml logs --tail 30 api` e `docker compose -f compose.yaml -f incidente.yaml config`.
- **Causa-raiz:** O ficheiro `incidente.yaml` alterou o comando de arranque do Uvicorn para escutar na porta `8001` (`--port 8001`), criando uma incompatibilidade com o `healthcheck` interno (`http://127.0.0.1:8000/health`) e com o mapeamento de portas (`127.0.0.1:18080->8000`), que esperam o serviço na porta `8000`.
- **Correção:** Recriação do serviço utilizando apenas a configuração base funcional (`docker compose up -d --force-recreate`), mantendo o volume `app-data` intacto.
- **Validação:** O comando `docker compose ps` confirmou o regresso ao estado `Up ... (healthy)`, o endpoint `GET /health` devolveu `{"status":"ok","service":"support-api-integrador","version":"1.0"}` e o endpoint `GET /visits` devolveu `{"visits":1}`, comprovando a preservação dos dados no volume.

## E05 - Git (3 valores)
Branch, hashes de commits, merge e tag v1.0.0:

## E06 - Jenkins SUCCESS (4 valores)
URL/numero do build, etapas e identificador da imagem construida:

## E07 - Jenkins FAILURE e recuperacao (2 valores)
Build FAILURE, etapa que falhou e excerto relevante do log; build SUCCESS apos correcao:

## Verificacao final
[ ] Dockerfile [ ] compose.yaml [ ] app.py [ ] tests/ [ ] Jenkinsfile
[ ] Historico Git [ ] Tag v1.0.0 [ ] Evidencias E01-E07
