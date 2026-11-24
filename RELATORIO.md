# Relatorio de evidencias - Trabalho Pratico Integrador 01-06

Nome: Catarina Da Costa Figueiredo   Data: 24/11/2026
URL do repositorio Git: https://github.com/CatarinaCFigueiredo/tp-integrador.git

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
- **Branch de correção:** `fix/health-info`
- **Hashes dos dois commits principais:**
  - `ae6cf14` (`feat: ambiente integrador funcional` - commit inicial na `main`)
  - `1afed63` (`fix: acrescentar identificacao no health` - adição de `"team": "support"` em `app.py` e validação em `tests/test_app.py`)
- **Confirmação do merge:** Integração concluída na branch `main` via fast-forward (`Updating ae6cf14..1afed63`), com `.gitignore` a excluir `.venv` e `.pytest_cache`.
- **Tag anotada e publicação:** Tag `v1.0.0` (`Release integrador 1.0.0`) criada sobre o commit `1afed63` e publicada no remoto `origin` (`https://github.com/CatarinaCFigueiredo/tp-integrador.git`).

Conclusão: O fluxo de versionamento com branch dedicada, atualização de código e testes, integração na branch principal e etiquetagem de release foi executado e publicado de forma rastreável.
## E06 - Jenkins SUCCESS (4 valores)
URL/numero do build, etapas e identificador da imagem construida:
- **Job e número do build:** `natixis-tp-integrador` — Build `#3` (`Finished: SUCCESS`, duração: 43s)
- **URL do job/build:** `http://127.0.0.1:8080/job/natixis-tp-integrador/3/`
- **Agente de execução:** `ci-python-docker`
- **Commit validado:** `46848ad` (`fix: remover comentario invalido no sh do Jenkinsfile`)
- **Resumo das etapas:**
  - `Checkout`: Código obtido com sucesso a partir da branch `main` (`https://github.com/CatarinaCFigueiredo/tp-integrador.git`).
  - `Tests`: Ambiente virtual `.venv` criado, dependências instaladas e `pytest` executado com sucesso (`2 passed, 1 warning in 0.55s`).
  - `Docker Build`: Imagem `natixis-support:ci-3` construída a partir do `Dockerfile` e validada via `docker image inspect`, devolvendo o ID `sha256:6d9e956c5f86498d6e5b66bf52ae9fb85fcbe9059540f13ed389cb5c5b9ca39d`.

Conclusão: A pipeline CI/CD automatizou com êxito a verificação dos testes unitários e a construção da imagem Docker num agente dedicado.

## E07 - Jenkins FAILURE e recuperacao (2 valores)
Build FAILURE, etapa que falhou e excerto relevante do log; build SUCCESS apos correcao:
- **Build de falha controlada:** Build `#4` (`Finished: FAILURE`) — commit `9c74a3a` (`test: simular falha controlada na assercao de status`).
- **Etapa que falhou:** `Tests` (a etapa seguinte `Docker Build` foi automaticamente ignorada com `Stage "Docker Build" skipped due to earlier failure(s)`).
- **Excerto relevante do log (`Console Output` do Build `#4`):**
  ```text
  >       assert response.json()["status"] == "offline"
  E       AssertionError: assert 'ok' == 'offline'
  E         
  E         - offline
  E         + ok

  tests/test_app.py:10: AssertionError
  FAILED tests/test_app.py::test_health - AssertionError: assert 'ok' == 'offline'

## Verificacao final
[x] Dockerfile [x] compose.yaml [x] app.py [x] tests/ [x] Jenkinsfile
[x] Historico Git [x] Tag v1.0.0 [x] Evidencias E01-E07
