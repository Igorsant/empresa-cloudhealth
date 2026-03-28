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

Com base no levantamento atual (`src/contexto-infraestrutura.yaml`), consolidamos os riscos técnicos imediatos, as lacunas de documentação (gaps) que precisam ser preenchidas nas próximas revisões e o plano de evolução para garantir a segurança e estabilidade da plataforma CloudHealth.

### 10.1. Riscos Técnicos e Operacionais Atuais

| Risco / Vulnerabilidade | Impacto (Negócio e Sistema) | Estratégia de Mitigação |
| :--- | :--- | :--- |
| **Ausência de Ambiente de Staging** | A falta de um ambiente intermediário entre Desenvolvimento e Produção aumenta drasticamente o risco de regressões, bugs e indisponibilidade afetando a operação real das agendas médicas. | Criar um ambiente de Staging/Homologação idêntico à Produção para validação de QA e testes de carga antes dos deploys oficiais. |
| **Falta de Paridade no Cache (Redis)** | O `cache-redis` existe apenas em Produção. Desenvolvedores não conseguem prever ou testar comportamentos dependentes de cache no ambiente de Desenvolvimento. | Provisionar uma instância de Redis no ambiente de Desenvolvimento para garantir fidelidade de comportamento com a Produção. |
| **Único Ponto de Falha (SPOF) e Continuidade** | Ambos os ambientes estão na mesma região (`us-east-1`) e não há documentação sobre replicação do `banco-postgresql` ou contingência. Uma falha regional paralisa o sistema. | Avaliar a necessidade de replicação do banco de dados (Multi-AZ) e definir urgentemente um plano de contingência e recuperação. |

### 10.2. Lacunas de Documentação (Gaps)

Para as próximas revisões deste documento (após 2026-03-28), as seguintes áreas em branco devem ser investigadas e preenchidas com as áreas responsáveis:
* **Rede e Segurança:** Detalhar segmentação de rede/VPC, regras de entrada/saída, criptografia (em trânsito e repouso) e listar os requisitos de compliance obrigatórios (ex: LGPD/HIPAA para dados clínicos).
* **Continuidade e Recuperação (DR):** Definir e documentar a política de backup, estratégia de restauração e as métricas de RPO (Recovery Point Objective) e RTO (Recovery Time Objective).
* **Operação e Observabilidade:** Mapear quais são as métricas críticas do dashboard operacional, configurar alertas, definir responsáveis e documentar janelas de manutenção e gestão de incidentes.
* **Capacidade:** Levantar o perfil de consumo atual e o dimensionamento exato dos recursos.

### 10.3. Plano de Evolução e Melhorias Recomendadas

Para sustentar o crescimento e garantir a governança da infraestrutura, recomendamos os seguintes passos estratégicos:
1. **Implementação de FinOps (Otimização de Custos):** Como os ambientes rodam consolidados na região `us-east-1`, iniciar um estudo para adoção de instâncias reservadas ou *Savings Plans*, reduzindo o custo contínuo de computação.
2. **Evolução da Topologia de Ambientes:** Efetivar a criação do ambiente de Staging (Homologação) mitigando o risco de regressões apontado na matriz de riscos.
3. **Mapeamento de Compliance:** Priorizar o preenchimento da seção de Segurança, garantindo que a infraestrutura atenda aos requisitos legais de uma HealthTech antes de escalar novos serviços clínicos.

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
