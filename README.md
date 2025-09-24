<!-- # infra-terraform-s3-pipeline


####  Infra -->

graph TD
    subgraph "Ambiente Local"
        A[Developer]
    end

    subgraph "GitHub"
        B[Git Repository]
        C["GitHub Actions Workflow (develop.yml / main.yml)"]
        D["Workflow Reutilizável (terraform.yml)"]
        E[Arquivo de Configuração destroy_config.json]
    end

    subgraph "AWS Account"
        subgraph "Autenticação (IAM)"
            F[IAM Role for GitHub Actions]
        end

        subgraph "Gerenciamento de Estado do Terraform"
            G[S3 Bucket para State: kelberstuchi-us-east-1-terraform-pipeline]
            H[DynamoDB Table para Lock: kelberstuchi-us-east-1-terraform-lock]
        end

        subgraph "Infraestrutura Gerenciada (Terraform)"
            I[S3 Bucket da Aplicação: kelberstuchi-dev/prod-east-1-buildrun-pipeline]
            J[DynamoDB Table da Aplicação: ...-lock]
        end
    end

    %% Fluxo de Ações
    A -->|git push (develop/main)| B
    B -->|Aciona o Workflow| C
    C -->|Chama e passa parâmetros (env, ARN, etc.)| D
    D -->|1. Lê config de destroy| E
    D -->|2. Assume a Role para se autenticar| F
    F -->|Concede permissões temporárias| D
    D -->|3. Executa terraform init| G
    D -->|3. Executa terraform init| H
    G -->|Lê/Escreve terraform.tfstate| D
    H -->|Cria/Verifica o lock do estado| D
    D -->|4. Executa terraform plan/apply| I
    D -->|4. Executa terraform plan/apply| J

    style I fill:#f9f,stroke:#333,stroke-width:2px
    style J fill:#f9f,stroke:#333,stroke-width:2px
