# Documentação de Infraestrutura - CloudHealth

## Metadados do Documento
- Documento: Infraestrutura CloudHealth
- Versão: 1.0
- Status: Rascunho
- Responsável (owner):
- Aprovador:
- Última atualização: 2026-03-28
- Próxima revisão:
- Público-alvo: Times de engenharia, operações e segurança
- Classificação da informação: Interna

## Premissas, Lacunas e Riscos (preenchimento obrigatório)
- Premissas: As informações refletem o estado atual do arquivo `src/contexto-infraestrutura.yaml`. Componentes listados estão ativos em cada ambiente.
- Lacunas de informação: Configurações de rede/VPC, política de backup, RPO/RTO, dimensionamento de recursos e custos não estão documentados no contexto fonte.
- Riscos identificados: Ausência de ambiente de staging entre dev e produção pode aumentar o risco de regressões em produção.


## 1. Visão Geral
- Objetivo da infraestrutura: Suportar a aplicação CloudHealth nos ambientes de desenvolvimento e produção.
- Escopo da documentação: Ambientes, componentes, segurança e observabilidade conforme definidos em `src/contexto-infraestrutura.yaml`.
- Ambientes cobertos: Desenvolvimento, Produção

## 2. Topologia de Ambientes
| Ambiente | Região | Finalidade | Criticidade | Responsável |
|---|---|---|---|---|
| Desenvolvimento | us-east-1 | Desenvolvimento e testes | Baixa | |
| Produção | us-east-1 | Carga de trabalho real | Alta | |

## 3. Componentes de Infraestrutura
| Componente | Função | Ambiente(s) | Observações |
|---|---|---|---|
| app-service | Serviço de aplicação principal | Desenvolvimento, Produção | |
| banco-postgresql | Banco de dados relacional | Desenvolvimento, Produção | |
| cache-redis | Cache em memória | Produção apenas | Não presente no ambiente de desenvolvimento |

## 4. Rede e Conectividade
- Segmentação de rede: Não documentada — a preencher.
- Entrada/saída de tráfego: Não documentada — a preencher.
- Dependências externas: SSO corporativo (autenticação), cofre centralizado de segredos.

## 5. Segurança
- Controle de acesso: Autenticação via SSO corporativo.
- Gestão de segredos: Cofre centralizado — segredos não são armazenados em variáveis de ambiente ou repositório.
- Criptografia em trânsito e em repouso: Não documentada — a preencher.
- Requisitos de compliance relevantes: A preencher.

## 6. Observabilidade e Monitoramento
- Ferramentas de observabilidade: Agregador central de logs, dashboard operacional de métricas.
- Métricas críticas: A preencher com base no dashboard operacional.
- Alertas e responsáveis: A preencher.
- Estratégia de logs: Logs centralizados via agregador central.

## 7. Backup, Recuperação e Continuidade
- Política de backup: Não documentada — a preencher.
- Estratégia de restauração: Não documentada — a preencher.
- RPO/RTO desejados: Não documentados — a preencher.
- Plano de contingência: Não documentado — a preencher.

## 8. Capacidade e Custos
- Perfil de consumo atual: Não documentado — a preencher.
- Pontos de escalabilidade: Cache Redis presente apenas em produção; avaliar necessidade de replicação do banco.
- Oportunidades de otimização de custo: Ambos os ambientes na mesma região (us-east-1) — avaliar uso de instâncias reservadas ou Savings Plans.

## 9. Operação e Rotinas
- Rotinas operacionais: A preencher.
- Janelas de manutenção: A preencher.
- Gestão de incidentes: A preencher.

## 10. Riscos e Plano de Evolução
- Riscos técnicos atuais: Ausência de cache Redis no ambiente de desenvolvimento pode mascarar comportamentos dependentes de cache em produção.
- Gaps de documentação: Rede/VPC, backup, RPO/RTO, dimensionamento e custos ainda não documentados.
- Evoluções recomendadas: Adicionar ambiente de staging; avaliar replicação multi-região para produção.


## Anexos e Referências
- Diagramas de topologia e rede: A anexar.
- Inventário/planilha de componentes: `src/contexto-infraestrutura.yaml`
- Políticas de backup/segurança relacionadas: A anexar.
- Links de PRs/issues relacionados:

## Checklist de Qualidade (pré-entrega)
- [x] Ambientes e topologia descritos com clareza.
- [x] Segurança, observabilidade e continuidade cobertas.
- [ ] Backup, recuperação e RPO/RTO registrados.
- [ ] Capacidade, custos e riscos mapeados.
- [x] Premissas, lacunas e riscos preenchidos.
