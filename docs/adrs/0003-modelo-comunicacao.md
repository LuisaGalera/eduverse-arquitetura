# ADR 0003: Modelo de Comunicação

## Status

Aceita

## Contexto

O EduVerse é uma plataforma de aprendizado adaptativo que interage com diversos componentes internos (aplicação web, API Gateway, Motor de IA, processador de dados) e sistemas externos (LMS Moodle, PostgreSQL). A escolha do modelo de comunicação entre esses componentes é fundamental para garantir o desempenho, a escalabilidade, a resiliência e a manutenibilidade do sistema. É necessário avaliar os trade-offs entre comunicação síncrona e assíncrona para otimizar a interação entre os serviços e atender aos Requisitos Não Funcionais (RNFs) do EduVerse.

## Decisão

Decidiu-se pela adoção de um modelo de comunicação **híbrido**, utilizando **comunicação síncrona** para interações em tempo real que exigem resposta imediata e **comunicação assíncrona** para processos que podem ser executados em segundo plano, não exigem resposta imediata ou envolvem sistemas externos com latência variável.

### Justificativa

1.  **Comunicação Síncrona (API Gateway - Aplicação Web - Motor de IA)**: Para interações como a exibição de recomendações personalizadas na interface do usuário, a comunicação síncrona é essencial. O usuário espera uma resposta imediata ao navegar pela plataforma. A API Gateway atua como um ponto de entrada unificado, roteando requisições síncronas da aplicação web para os serviços internos, como o Motor de IA. Isso garante uma experiência de usuário fluida e responsiva, atendendo ao RNF de Desempenho [1].
2.  **Comunicação Assíncrona (Processador de Dados - LMS Moodle - PostgreSQL)**: Para processos como a sincronização de dados de matrículas, notas históricas e progresso do aluno com o LMS Moodle e o PostgreSQL, a comunicação assíncrona é mais adequada. Essas operações podem ser demoradas e não exigem uma resposta imediata ao usuário. A utilização de filas de mensagens (ex: Apache Kafka) permite que o processador de dados envie eventos para os sistemas externos sem bloquear a execução de outras tarefas. Isso melhora a escalabilidade, a resiliência e a manutenibilidade do sistema, pois falhas temporárias em um sistema externo não afetam a disponibilidade do EduVerse [2].
3.  **Desacoplamento e Resiliência**: A comunicação assíncrona promove um maior desacoplamento entre os serviços, tornando o sistema mais resiliente a falhas. Se um serviço consumidor estiver temporariamente indisponível, as mensagens podem ser armazenadas na fila e processadas posteriormente, evitando a perda de dados e a propagação de falhas em cascata [2].

### Alternativas Consideradas e Rejeitadas

*   **Apenas Comunicação Síncrona**: A adoção exclusiva de comunicação síncrona para todas as interações foi rejeitada. Embora mais simples de implementar inicialmente, essa abordagem introduziria latência desnecessária para operações de longa duração e tornaria o sistema mais suscetível a falhas em cascata, especialmente ao interagir com sistemas externos. O desempenho e a escalabilidade seriam severamente comprometidos durante picos de tráfego [1].
*   **Apenas Comunicação Assíncrona**: A utilização exclusiva de comunicação assíncrona para todas as interações foi rejeitada devido à necessidade de respostas imediatas para a experiência do usuário. Converter todas as interações em assíncronas introduziria complexidade adicional para gerenciar o estado e a notificação de conclusão para o usuário, o que não seria ideal para a interface interativa do EduVerse.

## Consequências

*   **Positivas**:
    *   **Desempenho Otimizado**: Respostas rápidas para interações em tempo real e processamento eficiente em segundo plano para operações de longa duração.
    *   **Escalabilidade Aprimorada**: Capacidade de lidar com grandes volumes de requisições e eventos, com a comunicação assíncrona atuando como um buffer para picos de carga.
    *   **Resiliência Elevada**: Maior tolerância a falhas em serviços dependentes e sistemas externos, com a capacidade de reprocessar mensagens e evitar a perda de dados.
    *   **Desacoplamento de Serviços**: Redução da dependência entre os componentes, facilitando a evolução e a manutenção individual de cada serviço.
*   **Negativas**:
    *   **Complexidade Aumentada**: A implementação de um modelo híbrido exige um gerenciamento mais complexo de diferentes padrões de comunicação e a introdução de infraestrutura adicional (ex: filas de mensagens).
    *   **Rastreamento e Depuração**: A depuração de fluxos de trabalho assíncronos pode ser mais desafiadora devido à natureza distribuída e não bloqueante das operações.
    *   **Consistência de Dados**: A garantia de consistência de dados em sistemas assíncronos pode exigir a implementação de padrões como *eventual consistency*, o que adiciona complexidade ao design do sistema.

## Referências

[1] Richards, M., & Ford, N. (2020). *Fundamentals of Software Architecture: An Engineering Approach*. O'Reilly Media.
[2] Hohpe, G., & Woolf, B. (2003). *Enterprise Integration Patterns: Designing, Building, and Deploying Messaging Solutions*. Addison-Wesley Professional.
