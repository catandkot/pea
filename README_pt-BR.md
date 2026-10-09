<div align="center">

[한국어](README.md) · [简体中文](README_zh.md) · [繁體中文](README_zh-TW.md) · [日本語](README_ja.md) · [Türkçe](README_tr.md) · [Русский](README_ru.md) · [Tiếng Việt](README_vi.md) · [ไทย](README_th.md) · [Deutsch](README_de.md) · [Español](README_es.md) · [Français](README_fr.md) · [Українська](README_uk.md) · [Polski](README_pl.md) · **Português (Brasil)** · [العربية](README_ar.md) · [فارسی](README_fa.md)

# 🫛 PEA

### Agente de fazenda inteligente — lê, interpreta e age sobre os dados do campo de cultivo.

**Os sensores já estão falando. O PEA faz com que essas mensagens sejam compreendidas.**

[![licença](https://img.shields.io/badge/license-MIT-f4c430?style=flat-square)](LICENSE)
[![situação](https://img.shields.io/badge/status-concept-5c4ee5?style=flat-square)](#status-do-projeto)

[Início rápido](#início-rápido) · [O que ele faz](#o-que-ele-faz) · [Como funciona](#como-funciona) · [Princípios de segurança](#princípios-de-segurança) · [Roadmap](#roadmap)

</div>

---

O PEA (ervilha) é um **agente de fazenda inteligente** para estufas e cultivos em campo aberto. Ele reúne leituras de sensores como umidade do solo, temperatura e umidade do ar, radiação solar, CO2 e EC/pH da solução drenada para explicar *o que está acontecendo agora* e controla atuadores como irrigação, ventilação, sombreamento e iluminação suplementar **dentro de limites de segurança**.

Toda decisão vem acompanhada de evidências. O PEA relata separadamente os **valores observados**, os **valores inferidos** e os **valores ainda desconhecidos**. "O solo está seco" vem do histórico dos sensores, "há alto risco de míldio à tarde" é uma inferência, e "se a válvula de irrigação realmente abriu" permanece desconhecido até que o medidor de vazão confirme.

O trabalho acontece inteiramente dentro da fazenda. Os únicos dados que saem para fora são os itens que o usuário escolheu explicitamente.

## Início rápido

> **Os comandos e os nomes de pacote abaixo ainda não estão definidos.** No momento não existe um pacote `pea` publicado no npm. Quando o lançamento acontecer, atualizaremos este documento com os comandos reais.

```bash
# 1) install (package not published yet)
npx pea-agents setup

# 2) readiness check — gateway, sensors and actuators
pea doctor

# 3) ask about today's greenhouse
pea analyze ./farm.toml
```

Depois de conectar o agente, basta perguntar assim.

```text
A irrigação da manhã na estufa 3 ficou 12% acima da meta. Qual foi a causa?
Como devo mudar o cronograma de irrigação desta tarde para atingir 25% de drenagem?
Mostre quantas horas durou a umidade foliar com base na temperatura mínima da noite passada.
```

## O que ele faz

| Área | O que o PEA faz | Dados utilizados |
| --- | --- | --- |
| Ambiente de cultivo | Tendência de temperatura e umidade na estufa, VPD e radiação solar acumulada, com destaque para faixas anômalas | Sensores de temperatura e umidade, piranômetro |
| Irrigação e nutrição | Cálculo do volume de irrigação com base na radiação solar acumulada e alerta de desvio em relação à meta de drenagem | Sensores de umidade do solo e de peso, EC/pH da solução drenada |
| Ventilação e climatização | Controle para manter a meta de VPD e sugestão de estágios para janelas de ventilação e exaustores | Feedback de abertura das janelas, clima externo |
| Alerta de pragas e doenças | Cálculo do risco de doenças fúngicas a partir da duração da umidade foliar combinada com temperatura e umidade | Histórico de temperatura e umidade, sensores de umidade foliar |
| Diário de cultivo | Registro automático do histórico de irrigação, ambiente e operações, e apoio na entrada de dados de acompanhamento do cultivo | Registros de operações, fotos do campo |
| Previsão de colheita | Estimativa da época de colheita com base nos graus-dia acumulados e comparação entre ciclos de cultivo | Histórico de temperatura, parâmetros da variedade |

Os itens e limites da tabela são **valores que precisam ser recalibrados para cada cultura, variedade e região**. O PEA apenas sugere valores padrão; ele não os impõe como referência daquela fazenda.

## Como funciona

Quando o agente faz uma pergunta ao PEA via MCP, o PEA coleta os dados no gateway e devolve o resultado junto com as evidências. Com esse resultado, o agente explica a situação ou propõe uma ação de controle. O mesmo fluxo de trabalho pode ser usado igualmente pela CLI no terminal.

```text
Agente → MCP → PEA → Gateway → Sensores
                        ← observações + inferências + desconhecidos
```

- **Local primeiro** — a coleta e as decisões são processadas dentro da fazenda.
- **Evidência primeiro** — junto com a conclusão, registramos o valor original, o horário e a origem.
- **O desconhecido como desconhecido** — períodos sem sensor ou com comunicação interrompida são declarados, e não preenchidos com estimativas.
- **Mesmo resultado** — a CLI e o MCP compartilham o mesmo fluxo de trabalho e o mesmo contrato de resultado.

## Integrações suportadas

| Integração | Forma | Situação |
| --- | --- | --- |
| Gateway MQTT | Coleta de tópicos de sensores por assinatura no broker | Planejado |
| Modbus RTU / TCP | Polling dos registradores do controlador | Planejado |
| LoRaWAN | Sensores de campo aberto por meio do gateway | Planejado |
| Dados meteorológicos | API meteorológica regional, temperatura, umidade e precipitação externas | Planejado |
| Sistemas de manejo de cultivo existentes | Importação de arquivos (CSV/JSON) para mesclar o histórico | Planejado |

## Princípios de segurança

Acionar um atuador é algo irreversível. Por isso o PEA segue estas regras.

1. **Aprovação humana por padrão** — ações reais como irrigação, ventilação e iluminação suplementar primeiro exibem o plano e só então recebem aprovação.
2. **Limite rígido** — os tetos configurados na fazenda (volume máximo de irrigação por evento, faixa de temperatura permitida, entre outros) não podem ser ultrapassados por decisão do agente.
3. **Fail-safe** — em caso de perda de comunicação ou queda de energia, o sistema assume o lado seguro (interromper a irrigação, abrir a ventilação, etc.).
4. **Aplicação não é automatizada** — a pulverização de defensivos e fertilizantes não é executada pelo PEA; ele apenas sugere o momento e as evidências.
5. **Registro completo** — todo controle deixa registrados o alvo, o horário, as evidências e o resultado da execução.

## Roadmap

- **M0 — Conceito e documentação** *(atual)*: definição do produto, princípios de segurança e desenho do contrato de resultado
- **M1 — Coleta**: integração com o gateway, armazenamento do histórico dos sensores e consulta das tendências ambientais
- **M2 — Recomendações**: consultor de irrigação e ventilação, risco de pragas e doenças e automação do diário de cultivo
- **M3 — Controle com aprovação**: controle automático sob aprovação humana e fail-safe
- **M4 — Operação**: previsão de colheita, comparação entre várias fazendas e relatórios de ciclo de cultivo

## Status do projeto

Este documento descreve o **conceito de produto do PEA**. O código presente no checkout atual é uma ferramenta de engenharia reversa (`rea-agents`) e ainda não implementa os recursos de coleta e controle descritos acima. Os nomes de comandos, a lista de integrações e os limites serão definidos junto com a implementação. Os READMEs traduzidos ainda se baseiam na documentação anterior e serão ajustados gradualmente conforme o rebranding.

## Contribuição

Issues e pull requests são bem-vindos. Observações obtidas em campo, relatos de falhas e valores de calibração por cultura são especialmente úteis.

## Licença

[MIT](LICENSE)
