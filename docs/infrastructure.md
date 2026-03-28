# Documentação de Infraestrutura - CloudHealth

## Metadados do Documento
- Documento: Infraestrutura CloudHealth
- Versão: 1.0
- Status: Rascunho
- Responsável (owner): Engenheiros Devops
- Aprovador: Arquiteto de soluções
- Última atualização: 2026-03-28
- Próxima revisão: 2027-03-29
- Público-alvo: Times de engenharia, operações e segurança
- Classificação da informação: Interna

## Premissas, Lacunas e Riscos (preenchimento obrigatório)
- Premissas: 
  - Componentes listados estão ativos em cada ambiente
  - Utiliza ambiente da AWS
- Lacunas de informação:
  - Topologia detalhada de rede e VPC não documentada (sub-redes, Security Groups, NACLs, NAT Gateways)
  - Janelas de manutenção não definidas formalmente
  - Classificação formal de severidade de incidentes ainda não implementada
  - Diagramas de topologia e rede pendentes de elaboração
- Riscos identificados: 
  - Ausência de ambiente de staging entre dev e produção pode aumentar o risco de regressões em produção.
  - Ausência de cache no ambiente de desenvolvimento, podendo causar divergência no desenvolvimento da aplicação.


## 1. Visão Geral
- Objetivo da infraestrutura: Suportar a aplicação CloudHealth nos ambientes de desenvolvimento e produção com alta confiabilidade e alta disponibilidade.
- Escopo da documentação: Ambientes, componentes, segurança e observabilidade conforme definidos em `src/contexto-infraestrutura.yaml`.
- Ambientes cobertos: Desenvolvimento, Produção

## 2. Topologia de Ambientes
| Ambiente | Região | Finalidade | Criticidade | Responsável |
|---|---|---|---|---|
| Desenvolvimento | us-east-1 | Desenvolvimento e testes | Baixa | DevOps e Desenvolvimento|
| Produção | us-east-1 | Carga de trabalho real | Alta | Data Protection Officer (DPO) e DevSecOps |

## 3. Componentes de Infraestrutura
| Componente | Função | Ambiente(s) | Observações |
|---|---|---|---|
| app-service | Serviço de aplicação principal (ex.: Node.js)| Desenvolvimento, Produção | Presente em todos os ambientes |
| banco-postgresql | Armazenamento de dados | Desenvolvimento, Produção | Backups automáticos diários |
| cache-redis | Cache em memória | Produção apenas | Não presente no ambiente de desenvolvimento |
| métricas | Monitoramento | Produção apenas (Grafana) | Alertas |
| agregador-central | centraizador de logs | Desenvolvimento, Produção | Ponto único de retenção e busca (ex.: CloudWatch) |

## 4. Rede e Conectividade
- Segmentação de rede: ambientes separados logicamente (desenvolvimento e produção).
- Entrada/saída de tráfego: entrada via requisições de usuários (HTTP/HTTPS).
- Dependências externas: SSO corporativo (autenticação), cofre centralizado de segredos.

## 5. Segurança
- Controle de acesso: Autenticação via SSO corporativo.
- Gestão de segredos: AWS Secrets — segredos não são armazenados em variáveis de ambiente ou repositório.
- Criptografia em trânsito e em repouso: Tráfego forçado via TLS 1.2+ e criptografia ativada via KMS para discos, banco de dados e backups.
- Requisitos de compliance relevantes: Adequação à LGPD para dados sensíveis e conformidade com HIPAA para dados de saúde.

## 6. Observabilidade e Monitoramento
- Ferramentas de observabilidade: Agregador central de logs, dashboard operacional de métricas.
- Métricas críticas: Taxa de erros, latência (p90/p99) e throughput do app-service, além de saturação do banco-postgresql e cache-redis.
- Alertas e responsáveis: P1 (Queda/Crítico) notifica o SRE/Plantão; P2 (Degradação) notifica a squad de desenvolvimento responsável.
- Estratégia de logs: Logs centralizados via agregador central com formatação em JSON e mascaramento de dados pessoais (PII).

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
- Rotinas operacionais:
    - Monitoramento contínuo da infraestrutura por meio de dashboards operacionais
    - Análise periódica de logs centralizados para identificação de falhas e anomalias
    - Verificação da disponibilidade dos serviços críticos
- Janelas de manutenção:
    - Ainda não definidas formalmente
    - Recomenda-se definição de janelas fora do horário de pico para minimizar impacto aos usuários
- Gestão de incidentes: 
    - Incidentes são identificados por meio de alertas gerados pelo sistema de monitoramento
    - A equipe DevOps é responsável pela análise e resolução dos incidentes
    -  O diagnóstico é realizado com base em logs e métricas coletadas
    - Recomenda-se adoção futura de classificação de incidentes por severidade. Ex: Crítico, alto, médio, etc...

## 10. Riscos e Plano de Evolução

Nesta seção, consolidamos os riscos técnicos identificados na arquitetura atual, as lacunas de documentação que precisam ser preenchidas nas próximas iterações e o plano de evolução estratégica para a infraestrutura da CloudHealth, considerando a criticidade dos dados clínicos e agendas médicas.

### 10.1. Riscos Técnicos e Operacionais Atuais

| Risco / Vulnerabilidade | Impacto (Negócio e Sistema) | Estratégia de Mitigação |
| :--- | :--- | :--- |
| **Falta de Paridade entre Ambientes** | A ausência de cache Redis no ambiente de Desenvolvimento pode mascarar comportamentos dependentes de cache, resultando em falhas ou gargalos inesperados ao fazer deploy em Produção. | Provisionar uma instância de Redis (com dimensionamento reduzido) no ambiente de DEV/Local para garantir fidelidade com a Produção. |
| **Riscos de Continuidade e SPOF** | Sem o mapeamento claro de redundância, falhas em instâncias únicas podem derrubar o sistema de agendamento médico. | Revisar a arquitetura para garantir que serviços críticos e bancos de dados operem no mínimo em Multi-AZ (múltiplas zonas de disponibilidade). |
| **Exposição de Dados Sensíveis** | Por ser uma HealthTech, configurações incorretas de rede podem expor dados clínicos, gerando multas e quebra de conformidade (LGPD). | Aplicar criptografia em trânsito e repouso por padrão, e revisar rigorosamente o acesso aos bancos de dados. |

### 10.2. Lacunas de Documentação (Gaps)

As seguintes áreas ainda não foram mapeadas na versão atual desta documentação e devem ser priorizadas nos próximos ciclos:
* **Topologia de Rede e VPC:** Falta o detalhamento de sub-redes (públicas e privadas), tabelas de roteamento, NAT Gateways e regras de firewall (Security Groups / NACLs).
* **Dimensionamento e Custos (Capacity & FinOps):** O sizing atual dos recursos (CPU, RAM, Storage) e o baseline de custos da infraestrutura na nuvem ainda não foram documentados.

### 10.3. Plano de Evolução e Melhorias Recomendadas

Para sustentar o crescimento seguro da plataforma CloudHealth e facilitar a gestão de capacidade e conformidade, recomendamos as seguintes evoluções arquiteturais:
1. **Criação de Ambiente de Staging (Homologação):** Implementar um ambiente isolado, porém idêntico à Produção, para realização de testes de carga, validação de QA e simulação de deploys. 
2. **Alta Disponibilidade Geográfica:** Avaliar a implementação de replicação multi-região (Multi-Region) para o ambiente de Produção, garantindo a disponibilidade do serviço de ponta a ponta mesmo em caso de indisponibilidade regional do provedor de nuvem.
3. **Maturidade em Observabilidade:** Implementar uma stack centralizada de monitoramento e logs para melhorar o tempo de detecção e resposta a incidentes.
4. **Infraestrutura como Código (IaC):** Iniciar a migração do provisionamento manual para IaC (ex: Terraform), garantindo rastreabilidade e prevenindo *configuration drift* (mudanças não documentadas diretamente no portal da nuvem).

## Anexos e Referências
- Diagramas de topologia e rede: A anexar.
- Inventário/planilha de componentes: `src/contexto-infraestrutura.yaml`
- Políticas de backup/segurança relacionadas: A anexar.
- Links de PRs/issues relacionados:

## Checklist de Qualidade (pré-entrega)
- [x] Ambientes e topologia descritos com clareza.
- [x] Segurança, observabilidade e continuidade cobertas.
- [x] Backup, recuperação e RPO/RTO registrados.
- [x] Capacidade, custos e riscos mapeados.
- [x] Premissas, lacunas e riscos preenchidos.
