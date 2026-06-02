# ADR 0001: Estratégia de Nuvem e Escalabilidade

## Status

Aceita

## Contexto

O EduVerse é uma plataforma de aprendizado adaptativo que necessita de alta disponibilidade e capacidade de escalar para atender a picos de demanda, como durante semanas de provas, onde o tráfego pode aumentar em 150%. A plataforma busca modernizar o processo de aprendizado de instituições de ensino, integrando-se com sistemas LMS existentes. A escolha da estratégia de nuvem e escalabilidade é crucial para garantir o desempenho, a manutenibilidade e a resiliência do sistema, conforme os Requisitos Não Funcionais (RNFs) priorizados.

## Decisão

Decidiu-se pela adoção de uma estratégia de nuvem híbrida, combinando **PaaS (Platform as a Service)** e **Serverless**, com foco em **escalabilidade horizontal**. Os componentes principais do EduVerse, como o Motor de IA e o processamento de dados, serão implementados utilizando serviços Serverless (ex: AWS Lambda, Azure Functions, Google Cloud Functions). A API Gateway e a aplicação web serão implantadas em plataformas PaaS (ex: AWS Elastic Beanstalk, Azure App Service, Google App Engine).

### Justificativa

1.  **Otimização de Custos e Operação**: A abordagem PaaS/Serverless permite que a equipe de desenvolvimento se concentre na lógica de negócios, delegando a gestão da infraestrutura, provisionamento e manutenção ao provedor de nuvem. Isso reduz custos operacionais e a sobrecarga de gerenciamento, alinhando-se com a necessidade de eficiência e agilidade no desenvolvimento de software [1].
2.  **Escalabilidade Horizontal Automática**: Os serviços Serverless e PaaS oferecem escalabilidade horizontal automática e elástica, o que é fundamental para lidar com os picos de tráfego esperados sem intervenção manual. O sistema poderá adicionar ou remover recursos dinamicamente conforme a demanda, garantindo que o desempenho não seja comprometido durante períodos de alta utilização [2].
3.  **Alta Disponibilidade e Resiliência**: Os provedores de nuvem oferecem infraestrutura redundante e mecanismos de failover integrados, aumentando a disponibilidade e a resiliência do EduVerse. A distribuição de componentes em múltiplas zonas de disponibilidade e regiões (quando aplicável) minimiza o impacto de falhas localizadas.
4.  **Foco no Negócio**: Ao abstrair a complexidade da infraestrutura, a equipe pode dedicar mais tempo ao desenvolvimento de funcionalidades que agregam valor direto aos usuários, como aprimoramento do Motor de IA e da experiência de aprendizado.

### Alternativas Consideradas e Rejeitadas

*   **IaaS (Infrastructure as a Service)**: Embora ofereça maior controle sobre a infraestrutura, a IaaS exigiria um gerenciamento mais intensivo de servidores, sistemas operacionais e redes. Isso aumentaria a complexidade operacional e os custos, desviando o foco da equipe do desenvolvimento do produto. A necessidade de gerenciar manualmente a escalabilidade e a manutenção de VMs foi considerada um *trade-off* inaceitável para os objetivos do EduVerse.
*   **Escalabilidade Vertical**: Aumentar os recursos (CPU, memória) de uma única instância de servidor foi rejeitado como estratégia principal de escalabilidade. Embora mais simples de implementar inicialmente, a escalabilidade vertical possui limites físicos e pode levar a gargalos de desempenho e pontos únicos de falha. A escalabilidade horizontal é mais adequada para sistemas distribuídos que precisam lidar com grandes volumes de usuários simultâneos e picos de demanda imprevisíveis [2].

## Consequências

*   **Positivas**:
    *   Redução significativa dos custos operacionais e de infraestrutura.
    *   Capacidade de escalar automaticamente para atender a picos de demanda, garantindo alta performance e disponibilidade.
    *   Maior agilidade no desenvolvimento e implantação de novas funcionalidades.
    *   Foco da equipe de desenvolvimento nas funcionalidades centrais do negócio.
*   **Negativas**:
    *   Dependência do provedor de nuvem, o que pode gerar *vendor lock-in*.
    *   Curva de aprendizado para a equipe em relação às ferramentas e serviços específicos do provedor de nuvem.
    *   Potencial complexidade no monitoramento e depuração de ambientes distribuídos e serverless.

## Referências

[1] Pressman, R. S. (2021). *Engenharia de Software: Uma Abordagem Profissional*. McGraw Hill.
[2] Richards, M., & Ford, N. (2020). *Fundamentals of Software Architecture: An Engineering Approach*. O'Reilly Media.
