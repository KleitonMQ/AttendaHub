Acessos:
N8N : http://localhost:5678/

C:\Users\Moosh>curl -X PUT http://localhost:8080/webhook -H "token: token" -H "Content-Type: application/json" -d "{\"webhook\":\"https://duvet-junkyard-stumbling.ngrok-free.dev/webhook-test/whatsapp\",\"events\":[\"Message\"],\"active\":true}"
{"code":200,"data":{"active":true,"events":["Message"],"webhook":"https://duvet-junkyard-stumbling.ngrok-free.dev/webhook-test/whatsapp"},"success":true}

C:\Users\Moosh>curl -X GET http://localhost:8080/webhook -H "token: token"
{"code":200,"data":{"subscribe":["Message"],"webhook":"https://duvet-junkyard-stumbling.ngrok-free.dev/webhook-test/whatsapp"},"success":true}

```mermaid
erDiagram

    setor {
        SERIAL id PK
        VARCHAR nome
    }

    funcionario {
        SERIAL id PK
        VARCHAR nome
        VARCHAR telefone
        INT setor_id FK
        VARCHAR status
        INT atendimentos_ativos
        TIMESTAMP criado_em
        TIMESTAMP atualizado_em
    }

    atendimento {
        SERIAL id PK
        VARCHAR protocolo
        VARCHAR telefone_cliente
        VARCHAR nome_cliente
        INT setor_id FK
        INT atendente_id FK
        VARCHAR status
        TIMESTAMP ultima_mensagem
        TIMESTAMP criado_em
        TIMESTAMP atualizado_em
    }

    fila_mensagens {
        BIGSERIAL id PK
        VARCHAR telefone_destino
        TEXT mensagem
        INT atendimento_id FK
        VARCHAR tipo
        VARCHAR status
        TIMESTAMP criado_em
        TIMESTAMP enviado_em
        INT tentativas
        TEXT erro
    }

    fila_ia {
        BIGSERIAL id PK
        INT atendimento_id FK
        VARCHAR telefone_cliente
        TEXT mensagem
        VARCHAR status
        TIMESTAMP criado_em
        TIMESTAMP processado_em
    }


    setor ||--o{ funcionario : possui

    setor ||--o{ atendimento : pertence

    funcionario ||--o{ atendimento : atende

    atendimento ||--o{ fila_mensagens : possui

    atendimento ||--o{ fila_ia : possui
```

