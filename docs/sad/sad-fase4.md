# Documento de Arquitetura de Software (SAD) - Fase 4: EduVerse

## 1. Visão Executiva do Sistema

O EduVerse é uma plataforma de aprendizado adaptativo que visa revolucionar a educação através da personalização. Utilizando inteligência artificial e análise de dados, a plataforma adapta o conteúdo e a metodologia de ensino às necessidades individuais de cada aluno. O sistema foi projetado para coexistir com sistemas de Gerenciamento de Aprendizado (LMS) existentes, como o Moodle, sem a necessidade de substituí-los, garantindo uma transição suave e aproveitando a infraestrutura já estabelecida pelas instituições de ensino. Na Fase 4, o EduVerse incorpora decisões arquiteturais robustas para garantir alta disponibilidade, escalabilidade e manutenibilidade, focando em uma experiência de usuário fluida e segura.

## 2. Requisitos Não Funcionais (RNFs) Priorizados na Fase 4

Com base na evolução do projeto, os seguintes RNFs foram priorizados e suas soluções arquiteturais foram consolidadas:

*   **Desempenho**: O sistema deve suportar até 5.000 usuários simultâneos com tempo de resposta inferior a 2 segundos. Isso é crucial para a experiência do usuário e a eficácia do aprendizado online. Estratégias como caching distribuído (Redis) e API Gateway foram implementadas para mitigar latências e otimizar o fluxo de dados.
*   **Escalabilidade**: A plataforma deve ser capaz de lidar com picos de tráfego (aumento de 150% durante semanas de provas) sem degradação de serviço. A arquitetura de microsserviços e a implantação em nuvem (PaaS/Serverless) com escalabilidade horizontal automática são as bases para atender a este requisito.
*   **Segurança**: Implementação de mecanismos de autenticação robustos, bloqueio de contas após múltiplas tentativas de login falhas e proteção contra vulnerabilidades comuns. A segurança dos dados do aluno e a integridade do sistema são prioridades máximas.
*   **Usabilidade**: A interface do usuário é otimizada para dispositivos móveis de baixo custo, garantindo que o conteúdo recomendado seja acessível em no máximo 5 cliques. O foco é na simplicidade e na intuição para alunos do ensino médio.
*   **Manutenibilidade**: A arquitetura hexagonal isola o núcleo de negócios de dependências externas, facilitando a evolução e a manutenção do sistema. A modularidade permite que professores atualizem conteúdos e criem trilhas de aprendizado de forma autônoma, sem intervenção da equipe de TI.

## 3. Estilo Arquitetural Escolhido: Arquitetura Hexagonal

A Arquitetura Hexagonal (também conhecida como Ports and Adapters) foi adotada para o EduVerse. Esta escolha foi motivada pela necessidade de isolar o **Motor de IA** e as **Regras de Negócio** do sistema de quaisquer dependências de infraestrutura ou sistemas externos (como o LMS Moodle e o PostgreSQL). [1]

### 3.1. Justificativa e Trade-offs

*   **Independência e Coexistência**: A arquitetura permite que o EduVerse coexista com o LMS institucional (Moodle) sem que a lógica de recomendação dependa da estrutura de dados do sistema legado. Isso facilita atualizações independentes e a gestão de conteúdo pelos professores. No entanto, introduz uma complexidade inicial de implementação devido à necessidade de múltiplas interfaces e camadas de tradução (ACL) para cada sistema externo integrado.
*   **Isolamento para Teste**: As regras de negócio e entidades, isoladas no centro do hexágono, podem ser testadas rigorosamente sem a necessidade de um banco de dados ou conexão ativa com a internet, garantindo um sistema robusto e confiável. O contrapartida é um aumento significativo no número de arquivos de código, o que pode dificultar o entendimento rápido do fluxo de dados por novos desenvolvedores.
*   **Performance**: A arquitetura permite a integração modular de adaptadores de Cache (Redis) e API Gateways, essenciais para suportar 5.000 usuários simultâneos e manter o tempo de resposta abaixo de 2 segundos. Embora a passagem de dados por diversas camadas possa gerar um pequeno *overhead* de processamento, este é mitigado pelas estratégias de cache implementadas.

## 4. Estratégia de Cloud e Implantação (Fase 4)

A estratégia de implantação do EduVerse na Fase 4 é baseada em um modelo **PaaS (Platform as a Service)** e **Serverless**, visando otimizar custos, escalabilidade e manutenibilidade. A escolha de provedores de nuvem como AWS, Azure ou GCP será definida com base em critérios de custo-benefício, serviços oferecidos e familiaridade da equipe. [2]

*   **Modelo de Serviço**: A maioria dos componentes será implantada como serviços Serverless (Funções Lambda, Azure Functions, Google Cloud Functions) para o Motor de IA e processamento de dados, e PaaS para a API Gateway e a aplicação web (ex: AWS Elastic Beanstalk, Azure App Service, Google App Engine). Isso permite que a equipe se concentre no desenvolvimento da lógica de negócios, delegando a gestão da infraestrutura ao provedor de nuvem.
*   **Escalabilidade**: A escalabilidade horizontal será a principal estratégia, com a capacidade de adicionar automaticamente mais instâncias de serviços (funções, contêineres) em resposta ao aumento da demanda. Ferramentas de auto-scaling dos provedores de nuvem serão configuradas para gerenciar picos de tráfego, como os observados durante as semanas de provas. A escalabilidade vertical será utilizada apenas para componentes específicos que exigem maior poder de processamento em uma única instância.
*   **Monitoramento**: Soluções de monitoramento nativas da nuvem (ex: AWS CloudWatch, Azure Monitor, Google Cloud Monitoring) serão utilizadas para coletar métricas de desempenho, logs e rastreamento distribuído. Isso permitirá a identificação proativa de gargalos e falhas, garantindo a saúde e a disponibilidade do sistema.

## 5. Análise de Fragilidade e Mitigação (Fase 4)

### 5.1. Ponto Frágil: Falha no Motor de IA

O **Motor de IA** é o coração do EduVerse, responsável pela personalização do aprendizado. Uma falha neste componente pode comprometer a funcionalidade central da plataforma, resultando em recomendações incorretas ou inexistentes, impactando diretamente a experiência do usuário e a eficácia do sistema.

### 5.2. Mitigação: Padrões de Resiliência

Para mitigar o risco de falha no Motor de IA, serão aplicados os seguintes padrões de resiliência:

*   **Circuit Breaker**: Implementado na comunicação entre a API Gateway e o Motor de IA. Se o Motor de IA começar a apresentar falhas ou latência excessiva, o Circuit Breaker abrirá o circuito, impedindo que novas requisições cheguem ao serviço defeituoso e permitindo que ele se recupere. Durante o período de circuito aberto, um *fallback* será acionado, oferecendo recomendações genéricas ou baseadas em um cache de últimas recomendações válidas. [3]
*   **Bulkhead**: O tráfego para o Motor de IA será isolado em *thread pools* ou instâncias de serviço separadas. Isso garante que uma falha ou sobrecarga em uma parte do sistema não afete a disponibilidade de outras partes. Por exemplo, requisições de alta prioridade (ex: carregamento inicial de recomendações) podem ter um *bulkhead* separado de requisições de baixa prioridade (ex: atualização de modelos de IA em segundo plano).
*   **Retry with Exponential Backoff**: As requisições falhas para o Motor de IA serão automaticamente retentadas com um atraso crescente entre as tentativas. Isso evita sobrecarregar o serviço já em recuperação e aumenta a chance de sucesso da requisição.

## 6. Parecer Técnico Final

A arquitetura do EduVerse, em sua Fase 4, representa uma escolha estratégica e robusta para atender aos requisitos de negócio e não funcionais. A adoção da Arquitetura Hexagonal garante um núcleo de negócios isolado e testável, facilitando a manutenibilidade e a evolução futura. A estratégia de implantação em nuvem (PaaS/Serverless) com foco em escalabilidade horizontal e monitoramento proativo assegura que a plataforma possa lidar com a demanda crescente e manter um alto desempenho. A aplicação de padrões de resiliência como Circuit Breaker e Bulkhead protege o sistema contra falhas críticas, especialmente no Motor de IA, garantindo a continuidade do serviço e a confiança do usuário. O valor agregado reside na capacidade de oferecer uma experiência de aprendizado personalizada e ininterrupta, com uma arquitetura flexível e preparada para o futuro.

## 7. Referências

[1] Richards, M., & Ford, N. (2020). *Fundamentals of Software Architecture: An Engineering Approach*. O'Reilly Media.
[2] Pressman, R. S. (2021). *Engenharia de Software: Uma Abordagem Profissional*. McGraw Hill.
[3] Nygard, M. T. (2018). *Release It!: Design and Deploy Production-Ready Software*. The Pragmatic Bookshelf.
