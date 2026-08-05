Acessos:
N8N : http://localhost:5678/

```mermaid
erDiagram

    SETOR {
        int id PK
        varchar nome
    }

    FUNCIONARIO {
        int id PK
        varchar nome
        varchar telefone UK
        int setor_id FK
        varchar status
        int atendimentos_ativos
        timestamp criado_em
        timestamp atualizado_em
    }

    ATENDIMENTO {
        int id PK
        varchar protocolo UK
        varchar telefone_cliente UK
        varchar nome_cliente
        int setor_id FK
        int atendente_id FK
        varchar etapa
        varchar status
        timestamp ultima_mensagem
        timestamp criado_em
        timestamp atualizado_em
    }

    SETOR ||--o{ FUNCIONARIO : possui

    SETOR ||--o{ ATENDIMENTO : atende

    FUNCIONARIO ||--o{ ATENDIMENTO : realiza
```

