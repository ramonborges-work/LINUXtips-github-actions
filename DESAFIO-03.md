# Desafio 03 – Containers e Segurança (GHCR)

## Requisitos do pipeline

- Deve existir um novo workflow do GitHub Actions para o Nível 3 (containers e segurança).
- O disparo deve acontecer quando um Pull Request para a branch `desafio-nivel-3` for fechado com merge (evento `pull_request`, `types: [closed]` + condição `github.event.pull_request.merged == true`).
- O workflow deve:
  - Fazer login no GitHub Container Registry (GHCR) utilizando `GITHUB_TOKEN`.
  - Montar a tag da imagem sempre com o SHA do commit, e com `owner`, `IMAGE_NAME` e `registry` em minúsculas.
  - Buildar a imagem Docker para permitir o scan de vulnerabilidades.
  - Executar lint do `Dockerfile` com Hadolint, salvando o resultado em `lint-report.txt` e falhando se forem encontrados os problemas DL3006 ou DL3008.
  - Executar o scan de vulnerabilidades com Trivy na imagem construída.
    - O resultado deve ser salvo em `trivy-report.txt` e publicado como artefato.
    - O workflow deve falhar caso o Trivy encontre vulnerabilidades CRITICAL.
  - Publicar no GHCR somente se todas as validações passarem.
  - Gerar o artefato `level-3-certificate.md` (não alterar a seção do certificado, assim como nos desafios anteriores).

### Importante: Proteção de branch

Antes de executar, configure a proteção da branch `desafio-nivel-3` como mostramos na live de quarta. Isso garante a qualidade do ciclo de revisão e evita merges diretos na `desafio-nivel-3` sem validações. Veja a demonstração aqui: [AO VIVO - Descomplicando Github Actions - Resolvendo Desafio](https://www.youtube.com/watch?v=VihvfGx58IY).

## Variável obrigatória

- Crie uma variável de repositório chamada `IMAGE_NAME` (em: Settings > Secrets and variables > Actions > Variables) com o nome da aplicação a ser usada no nome da imagem (ex.: `desafio3-linuxtips-gha`).
- Essa variável é obrigatória e será utilizada para compor o nome final da imagem no GHCR.

## Actions obrigatórias

- `docker/login-action@v3`
- `docker/build-push-action@v6`
- `aquasecurity/trivy-action@0.28.0`

### Política de lint (Hadolint)

Para nós, os checks do Hadolint devem focar nos itens que mais impactam segurança e reprodutibilidade. Portanto, este pipeline deve falhar apenas quando forem detectadas as regras abaixo:

- DL3006: uso consistente e fixação do gerenciador de pacotes/base da imagem (garante builds reprodutíveis);
- DL3008: instalação de pacotes sem pin de versões (evita deriva de dependências e janelas de vulnerabilidade).

Por política da empresa e conformidade de supply chain, consideramos DL3006 e DL3008 bloqueadores.

## Regras de publicação da imagem

- Registry: `ghcr.io`
- Tag obrigatória: o SHA do commit (`${{ github.sha }}`)
- Nome completo (exemplo): `ghcr.io/<owner>/<IMAGE_NAME>:<sha>`
- O nome completo deve ser convertido para minúsculas para evitar erros no push.

## Esqueleto mínimo do workflow

No arquivo `.github/workflows/03-build-containers.yml`, deixe apenas o esqueleto mínimo dentro do job `build-scan-and-push`. Defina os nomes dos steps como dicas, mas sem implementação. 

## Critérios de aceite

- [ ] Workflow Nível 3 dispara somente após PR mergeado na `desafio-nivel-3`.
- [ ] `Dockerfile` analisado com Hadolint; gerar artefato `lint-report.txt` e falhar em DL3006/DL3008.
- [ ] Imagem construída e escaneada com Trivy (CRITICAL = 0) com artefato `trivy-report.txt`.
- [ ] Push realizado no GHCR apenas se todas as verificações passarem.
- [ ] Tag da imagem é exatamente o SHA do commit e nome em minúsculas.
- [ ] Uso das actions nas versões exigidas.
