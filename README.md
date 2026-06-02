# EduVerse - Arquitetura do Sistema

## Visão Executiva do Sistema

O EduVerse é uma plataforma de aprendizado adaptativo que utiliza inteligência artificial e análise de dados para personalizar a experiência educacional. Seu objetivo é modernizar o processo de aprendizado de instituições de ensino, integrando-se com seus sistemas LMS existentes (como o Moodle) sem substituí-los. A plataforma identifica lacunas de conhecimento e recomenda conteúdos personalizados através de trilhas de aprendizado, avaliações dinâmicas e feedback instantâneo. Na Fase 4, o sistema está em um estado de maturidade arquitetural, com decisões bem fundamentadas para escalabilidade, resiliência e manutenibilidade, conforme detalhado nos documentos anexos.

## Diagrama C4 de Containers (Mermaid)

```mermaid
C4Container
    title Diagrama de Containers para EduVerse

    System_Ext(moodle, "LMS Moodle", "Sistema de Gerenciamento de Aprendizado existente")
    System_Ext(postgresql, "PostgreSQL", "Banco de Dados Relacional")

    Person(aluno, "Aluno", "Usuário da plataforma EduVerse")
    Person(professor, "Professor", "Usuário da plataforma EduVerse")

    Container_Boundary(eduverse, "Sistema EduVerse") {
        Container(webapp, "Aplicação Web", "React, TypeScript", "Interface do usuário para alunos e professores")
        Container(api, "API Gateway", "Node.js, Express", "API para comunicação com a aplicação web e sistemas externos")
        Container(ia_engine, "Motor de IA", "Python, TensorFlow", "Responsável pela personalização do aprendizado e recomendações")
        Container(data_processor, "Processador de Dados", "Apache Kafka, Spark", "Processamento de dados de uso e desempenho dos alunos")
        Container(database, "Banco de Dados", "MongoDB", "Armazena dados de perfil de aluno, progresso e conteúdo")
        Container(cache, "Cache Distribuído", "Redis", "Armazenamento temporário de dados para alta performance")
    }

    Rel(aluno, webapp, "Acessa via navegador")
    Rel(professor, webapp, "Acessa via navegador")
    Rel(webapp, api, "Faz requisições API")
    Rel(api, ia_engine, "Consulta recomendações")
    Rel(api, data_processor, "Envia eventos de uso")
    Rel(api, database, "Lê e escreve dados")
    Rel(api, cache, "Lê e escreve dados em cache")
    Rel(ia_engine, database, "Lê e escreve dados de modelo")
    Rel(data_processor, database, "Escreve dados processados")
    Rel(data_processor, moodle, "Sincroniza dados com", "Adaptador Moodle")
    Rel(data_processor, postgresql, "Sincroniza dados com", "Adaptador PostgreSQL")
    Rel(moodle, api, "Envia notificações via webhook")
```

## Documentação Arquitetural

### Architecture Decision Records (ADRs)

*   [ADR 0001 - Estratégia de Nuvem e Escalabilidade](./docs/adrs/0001-estrategia-nuvem.md)
*   [ADR 0002 - Padrões de Resiliência](./docs/adrs/0002-padrao-resiliencia.md)
*   [ADR 0003 - Modelo de Comunicação](./docs/adrs/0003-modelo-comunicacao.md)

### Software Architecture Document (SAD)

*   [SAD - Fase 4](./docs/sad/sad-fase4.md)


