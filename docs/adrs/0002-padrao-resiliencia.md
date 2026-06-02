# ADR 0002: Padrões de Resiliência

## Status

Aceita

## Contexto

O EduVerse é um sistema distribuído que integra diversos componentes, incluindo uma aplicação web, uma API Gateway, um Motor de IA, um processador de dados e sistemas externos como o LMS Moodle e um banco de dados PostgreSQL. A comunicação entre esses componentes e sistemas externos é suscetível a falhas de rede, latência e indisponibilidade temporária. Para garantir a alta disponibilidade e a confiabilidade do sistema, especialmente durante picos de tráfego, é essencial implementar padrões de resiliência que mitiguem o impacto dessas falhas e evitem a propagação de erros em cascata.

## Decisão

Decidiu-se implementar os padrões de resiliência **Circuit Breaker** e **Bulkhead** na comunicação entre os componentes críticos do EduVerse, com foco especial na interação entre a API Gateway e o Motor de IA, e na comunicação com sistemas externos (LMS Moodle e PostgreSQL).

### Justificativa

1.  **Circuit Breaker**: O padrão Circuit Breaker atua como um disjuntor elétrico, monitorando as chamadas para um serviço dependente. Se o número de falhas exceder um limite predefinido, o circuito "abre", impedindo que novas requisições sejam enviadas ao serviço defeituoso. Isso evita que o sistema sobrecarregue um serviço já em falha e permite que ele se recupere. Durante o período de circuito aberto, um *fallback* pode ser acionado, fornecendo uma resposta alternativa (ex: recomendações genéricas em vez de personalizadas) ou uma mensagem de erro amigável ao usuário. Isso melhora a experiência do usuário e a estabilidade geral do sistema [1].
2.  **Bulkhead**: O padrão Bulkhead isola os recursos do sistema em "compartimentos" (como os compartimentos estanques de um navio). Se um componente falhar ou for sobrecarregado, a falha fica contida em seu compartimento, não afetando a disponibilidade de outros componentes. Por exemplo, *thread pools* separados podem ser alocados para chamadas ao Motor de IA e para chamadas ao banco de dados. Se o Motor de IA ficar lento, apenas o *thread pool* dedicado a ele será esgotado, enquanto as chamadas ao banco de dados continuarão a ser processadas normalmente [1].
3.  **Proteção contra Falhas em Cascata**: A combinação de Circuit Breaker e Bulkhead é fundamental para evitar que uma falha em um componente não crítico (ou temporariamente indisponível) cause a queda de todo o sistema. Isso é especialmente importante em arquiteturas de microsserviços, onde as dependências entre serviços são complexas e as falhas podem se propagar rapidamente [2].

### Alternativas Consideradas e Rejeitadas

*   **Retry Simples (sem Circuit Breaker)**: Implementar apenas um mecanismo de *retry* (tentar novamente) em caso de falha foi rejeitado como solução única. Embora útil para falhas transitórias de rede, o *retry* simples pode agravar o problema se o serviço dependente estiver sobrecarregado ou indisponível por um período prolongado, levando a um esgotamento de recursos e falhas em cascata. O *retry* deve ser combinado com um Circuit Breaker e um *backoff* exponencial para ser eficaz e seguro [1].
*   **Ignorar Falhas (Fail Fast sem Fallback)**: Retornar um erro imediatamente ao usuário em caso de falha de um serviço dependente foi considerado inaceitável para a experiência do usuário do EduVerse. A plataforma deve buscar fornecer uma funcionalidade degradada (ex: recomendações em cache) em vez de uma falha completa, sempre que possível.

## Consequências

*   **Positivas**:
    *   Aumento significativo da resiliência e disponibilidade do sistema.
    *   Prevenção de falhas em cascata e esgotamento de recursos.
    *   Melhoria na experiência do usuário, com respostas mais rápidas e *fallbacks* adequados em caso de falhas parciais.
    *   Facilitação da recuperação de serviços dependentes após períodos de instabilidade.
*   **Negativas**:
    *   Aumento da complexidade na implementação e configuração da comunicação entre serviços.
    *   Necessidade de definir e ajustar cuidadosamente os limiares de falha e os tempos de *timeout* para os Circuit Breakers.
    *   Esforço adicional para projetar e implementar lógicas de *fallback* adequadas para cada cenário de falha.

## Referências

[1] Nygard, M. T. (2018). *Release It!: Design and Deploy Production-Ready Software*. The Pragmatic Bookshelf.
[2] Richards, M., & Ford, N. (2020). *Fundamentals of Software Architecture: An Engineering Approach*. O'Reilly Media.
