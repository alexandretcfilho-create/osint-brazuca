# Skills externas (OSINT / CTI / ASM)

Coletânea de **skills de OSINT, CTI e mapeamento de superfície de ataque** desenvolvidas por
terceiros para uso com o Claude, espelhadas aqui para ampliar as capacidades de busca e análise
do projeto. Cada subpasta é uma cópia fiel do repositório de origem (sincronizada no commit
indicado), **preservando o `LICENSE` e a autoria original**.

> [!IMPORTANT]
> Este conteúdo é de **terceiros** e redistribuído sob as respectivas licenças (todas MIT; o
> `claude-osint` adiciona CC BY 4.0 para o conteúdo textual). Os créditos e direitos pertencem
> aos autores originais. Não foram importados os workflows de CI (`.github/`), hooks nem
> metadados de git dos projetos de origem — apenas o conteúdo das skills, documentação, scripts
> e licenças.

> [!WARNING]
> Mesmas regras do repositório principal: uso **estritamente legal**, sobre fontes públicas,
> respeitando a **LGPD (Lei nº 13.709/2018)**, o **Marco Civil (Lei nº 12.965/2014)** e demais
> normas aplicáveis. Reconhecimento **passivo**; nada aqui autoriza acesso não autorizado a
> sistemas de terceiros. A responsabilidade pelo uso é exclusiva do usuário.

## Procedência

| Pasta | Origem | Autor | Licença | Commit sincronizado |
|---|---|---|---|---|
| [`claude-osint/`](./claude-osint/) | [elementalsouls/Claude-OSINT](https://github.com/elementalsouls/Claude-OSINT) | Sachin Sharma | MIT + CC BY 4.0 (conteúdo) | `13d9204` |
| [`osint-skill/`](./osint-skill/) | [smixs/osint-skill](https://github.com/smixs/osint-skill) | Serge Shima | MIT | `94f382e` |
| [`cti-expert/`](./cti-expert/) | [7onez/cti-expert](https://github.com/7onez/cti-expert) | Hieu Ngo (chongluadao.vn) | MIT | `e05f986` |
| [`osint-md-for-claude/`](./osint-md-for-claude/) | [rostikcermak-pixel/OSINT.md-for-Claude-skill](https://github.com/rostikcermak-pixel/OSINT.md-for-Claude-skill) | WaffleNinja | MIT | `4bd4404` |
| [`osint-researcher/`](./osint-researcher/) | [MrBridgeHQ/osint-researcher-claude](https://github.com/MrBridgeHQ/osint-researcher-claude) | Mr Bridge | MIT | `e194262` |

Sincronizado em **2026-10-09**.

## O que cada skill cobre

- **`claude-osint/`** — Arsenal de reconhecimento externo e *attack surface management* (ASM) para
  engajamentos autorizados: metodologia OSINT em 6 estágios, mapeamento de superfície
  organizacional, recon de provedor de identidade, exposição em nuvem/SaaS, segurança de e-mail
  (SPF/DMARC/DKIM), quantificação de risco (FAIR) e monitoramento contínuo. Inclui catálogos de
  *dorks*, regex de segredos e scripts auxiliares (stdlib). Foco **passivo/discovery**.
- **`osint-skill/`** — Pipeline de investigação OSINT com *dossier*, perfis de plataformas,
  extração de conteúdo e integração com APIs comerciais de coleta (Exa, Tavily, Jina, Apify,
  Bright Data, Perplexity) via scripts.
- **`cti-expert/`** — *Cyber Threat Intelligence* voltado a **desmascarar o operador por trás de
  infraestrutura de crime/golpe** (phishing, scam, fraude): WebPivot (favicon hash, IDs de
  rastreamento, carteiras cripto, WHOIS), fingerprint de *kit* de phishing, detecção de
  *cloaking*, investigação em blockchain e OSINT de pagamentos. Orientado a investigação
  defensiva e atribuição.
- **`osint-md-for-claude/`** — Skill compacta de OSINT (`SKILL.md`), distribuída originalmente
  como pacote `.skill` (mantido e também extraído aqui).
- **`osint-researcher/`** — Skill de pesquisa OSINT com referências de disciplinas de coleta,
  metodologia, *threat intel*, ferramental MCP e um guia de ética/legalidade/OPSEC.

## Como atualizar (re-sincronizar)

Cada pasta é um *snapshot*. Para atualizar, basta clonar novamente o repositório de origem no
commit desejado e substituir o conteúdo da subpasta, mantendo este índice e o `LICENSE` de cada
projeto.
