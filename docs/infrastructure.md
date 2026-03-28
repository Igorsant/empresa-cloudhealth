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
- Política de backup:
 Banco de dados PostgreSQL:
 Backups automáticos diários (full backup)
 Backups incrementais a cada 6 horas
 Retenção de backups por 7 dias em desenvolvimento e 30 dias em produção
 Cache Redis:
 Snapshot diário (RDB) apenas em produção
 Não considerado crítico para recuperação completa (dados voláteis)
 Aplicação (app-service):
 Artefatos versionados em repositório (CI/CD)
 Não requer backup direto
- Estratégia de restauração: PostgreSQL:
 Restauração a partir do último backup completo + incrementais
 Procedimento validado via restore em ambiente isolado
 Redis:
 Restauração via snapshot mais recente (quando aplicável)
 Aplicação:
 Reimplantação via pipeline CI/CD
- RPO/RTO desejados: 
 Produção: até 6 horas de perda de dados
 Desenvolvimento: até 24 horas
- Plano de contingência: Falha no banco:
 Restaurar backup mais recente
 Redirecionar aplicação para nova instância
 Falha na aplicação:
 Re-deploy automático via pipeline
 Falha de infraestrutura regional:
 Ainda não suportado (single region: us-east-1)
 Recomendação futura: replicação multi-região

## 8. Capacidade e Custos
- Perfil de consumo atual: 
 Ambientes compartilham a mesma região (us-east-1)
 Produção possui maior consumo devido a:
 Uso de cache Redis
 Carga real de usuários
 Desenvolvimento com carga reduzida e menor uso de recursos
- Pontos de escalabilidade: 
 app-service:
 Escalável horizontalmente (adicionar instâncias)
 banco-postgresql:
 Escalabilidade vertical (CPU/RAM)
 Possibilidade futura de réplicas de leitura
 cache-redis (produção):
 Redução de carga no banco
 Pode ser escalado conforme volume de requisições
- Oportunidades de otimização de custo: 
 Uso de:
 Reserved Instances ou Savings Plans para produção
 Redução de custo em desenvolvimento:
 Desligamento fora do horário comercial
 Uso de instâncias menores
 Redis apenas em produção evita custo desnecessário
 Avaliar uso de autoscaling para evitar superdimensionamento

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
