# Catálogo de Casos de Uso e Relacionamentos — G03

> Visão geral dos 8 casos de uso do módulo (E3). UC-G03-01 a UC-G03-04 têm especificação textual
> completa em arquivos próprios. UC-G03-05 a UC-G03-08 são resumidos aqui; a especificação textual deles
> não é exigida no M1 e pode ser expandida no M2.
> Diagrama: [`../diagramas/diagrama-casos-de-uso-g03.puml`](../diagramas/diagrama-casos-de-uso-g03.puml)
> (exportação: `.svg` e `.png` na mesma pasta).

## 1. Atores

| Ator | Papel neste módulo |
| --- | --- |
| MedicoRegulador | Registra teleorientação, solicita a pré-admissão e dá ciência de recusa |
| EquipeIntervencao | Preenche a ficha em campo, registra fármaco e recusa |
| EquipeReceptora | Consulta a ficha após a pré-admissão enviada (passagem de plantão digital) |
| Módulo G2 (despacho) | Fornece o contrato C2 que abre a ficha |
| Módulo G4 (leitos) | Recebe o C3 e responde reservado ou negado |
| Módulo G5 (estoque) | Recebe o C4 de consumo de fármaco |
| Módulo G6 (auditoria) | Recebe os eventos sensíveis (C5) e é dono da assinatura digital |

## 2. Catálogo

| ID | Caso de uso | Ator primário | Especificação | Relacionamento | RFs |
| --- | --- | --- | --- | --- | --- |
| UC-G03-01 | Registrar Teleorientação | MedicoRegulador | completa | autônomo | RF-G03-01, RF-G03-02 |
| UC-G03-02 | Preencher Ficha Pré-Hospitalar | EquipeIntervencao | completa | base de três `<<extend>>` | RF-G03-03, RF-G03-04, RF-G03-05, RF-G03-07 |
| UC-G03-03 | Solicitar Pré-Admissão | MedicoRegulador | completa (fluxo crítico) | `<<include>>` UC-G03-05 | RF-G03-09, RF-G03-10, RF-G03-11 |
| UC-G03-04 | Registrar Recusa de Atendimento | EquipeIntervencao | completa | `<<extend>>` UC-G03-02 | RF-G03-12, RF-G03-13 |
| UC-G03-05 | Validar Prioridade | Sistema | resumida | incluído por UC-G03-03 | RF-G03-10 |
| UC-G03-06 | Registrar Consumo de Fármaco | EquipeIntervencao | resumida | `<<extend>>` UC-G03-02 | RF-G03-06 |
| UC-G03-07 | Consultar Ficha Pré-Hospitalar | MedicoRegulador, EquipeIntervencao, EquipeReceptora | resumida | autônomo | RF-G03-14, RF-G03-15 |
| UC-G03-08 | Sincronizar Ficha após Reconexão | Sistema | resumida | `<<extend>>` UC-G03-02 | RF-G03-07, RF-G03-08 |

Cobertura: os 15 RF têm ao menos um caso de uso (conferir na matriz de rastreabilidade).

## 3. Justificativa dos relacionamentos

| Relacionamento | Por que esse tipo |
| --- | --- |
| UC-G03-03 `<<include>>` UC-G03-05 | Obrigatório: o C3 exige prioridade herdada (não recalculada) e campos completos. Toda solicitação passa pela validação; ela está isolada porque também é reaproveitada ao reenviar o pedido (FA-01 do UC-G03-03). |
| UC-G03-04 `<<extend>>` UC-G03-02 | Opcional: só ocorre quando um paciente consciente recusa. A maioria dos atendimentos não tem recusa. |
| UC-G03-06 `<<extend>>` UC-G03-02 | Opcional: só ocorre quando há fármaco administrado. Quando ocorre, o lote passa a ser obrigatório (RN-07). |
| UC-G03-08 `<<extend>>` UC-G03-02 | Opcional: só ocorre quando há registros pendentes por falta de conexão. |

Por que não `<<include>>` nesses três: o preenchimento da ficha é completo sem recusa, sem fármaco e sem
sincronização pendente. Se fossem `<<include>>`, qualquer preenchimento teria de passar por eles.

## 4. Resumos dos casos de uso não detalhados

### UC-G03-05 — Validar Prioridade

- **Objetivo:** garantir que o pedido de pré-admissão carrega a prioridade vigente do chamado (herdada de G1) e que os campos do C3 estão completos.
- **Verificações:** a prioridade existe e é a vigente; há ao menos um registro de sinais vitais; `necessidade` é `SALA_VERMELHA` ou `OBSERVACAO`; `eta_hospital` está preenchido.
- **Exceção principal:** qualquer verificação falha: o envio é bloqueado e o campo faltante é indicado. A prioridade nunca é editável aqui.

### UC-G03-06 — Registrar Consumo de Fármaco

- **Objetivo:** registrar o fármaco administrado e publicar o C4 `consumo_farmaco` para G5.
- **Dados:** `item_id`, `lote`, `quantidade`, `via`, `horario`, `responsavel_id`, `prontuario_id`.
- **Exceção principal:** lote ausente: a gravação é bloqueada e o C4 não sai. A baixa de estoque é de G5.

### UC-G03-07 — Consultar Ficha Pré-Hospitalar

- **Objetivo:** permitir a leitura da ficha por quem precisa dela, com controle de perfil e registro de cada acesso.
- **Regra de acesso:** `EquipeIntervencao` só vê fichas do seu `equipe_id`; `EquipeReceptora`, só com pré-admissão enviada; `MedicoRegulador`, dos chamados que regula.
- **Exceção principal:** acesso fora do perfil: negado e registrado como `PRONTUARIO_ACESSADO` com `resultado = NEGADO`.

### UC-G03-08 — Sincronizar Ficha após Reconexão

- **Objetivo:** enviar os registros pendentes depois que a conexão volta, sem duplicar nem perder.
- **Regras:** ordem pelo horário do fato; reenvio do mesmo `evento_id` não cria cópia (ARQ §6).
- **Exceção principal:** nova queda durante a sincronização: os registros não confirmados continuam pendentes.
