# Exemplo de Uso do Terraform para Gerenciamento do AWS API Gateway

Este exemplo demonstra como usar o **Terraform** para criar e gerenciar um serviço **AWS API Gateway** integrado a uma função Lambda. O Terraform permite automação, controle e reprodutibilidade ao configurar a infraestrutura.

---

## **Descrição do Cenário**
- Um serviço de API Gateway expõe um endpoint `/hello`.
- O endpoint é integrado a uma função **AWS Lambda**, que processa a solicitação e retorna uma mensagem.
- Inclui configurações de monitoramento no **CloudWatch** e habilitação de **cache** para otimização de desempenho.

---

## **Código Terraform**

### Arquivo: `main.tf`
```hcl
provider "aws" {
  region = "us-east-1" # Define a região AWS onde os recursos serão criados
}

resource "aws_api_gateway_rest_api" "example_api" {
  name        = "example-api" # Nome da API REST
  description = "API Gateway integrado a Lambda" # Descrição da API
}

resource "aws_api_gateway_resource" "hello_resource" {
  rest_api_id = aws_api_gateway_rest_api.example_api.id # Vincula o recurso à API criada acima
  parent_id   = aws_api_gateway_rest_api.example_api.root_resource_id # Define o recurso raiz como pai
  path_part   = "hello" # Define a rota como /hello
}

resource "aws_api_gateway_method" "get_hello" {
  rest_api_id   = aws_api_gateway_rest_api.example_api.id # Vincula o método à API
  resource_id   = aws_api_gateway_resource.hello_resource.id # Vincula o método ao recurso /hello
  http_method   = "GET" # Define o método HTTP como GET
  authorization = "NONE" # Sem autenticação para simplificação
}

resource "aws_lambda_function" "hello_lambda" {
  filename         = "lambda_function.zip" # Caminho para o arquivo compactado com o código da função Lambda
  function_name    = "hello_lambda" # Nome da função Lambda
  role             = aws_iam_role.lambda_exec.arn # Papel IAM com permissões para execução da função
  handler          = "lambda_function.lambda_handler" # Especifica o ponto de entrada da função no código
  runtime          = "python3.9" # Ambiente de execução da função Lambda

  source_code_hash = filebase64sha256("lambda_function.zip") # Garante que alterações no código recompilado sejam detectadas
}

resource "aws_lambda_permission" "api_gateway_permission" {
  statement_id  = "AllowAPIGatewayInvoke" # Identificador da permissão
  action        = "lambda:InvokeFunction" # Permissão para invocar a função Lambda
  function_name = aws_lambda_function.hello_lambda.function_name # Nome da função Lambda
  principal     = "apigateway.amazonaws.com" # API Gateway como principal autorizado
  source_arn    = "${aws_api_gateway_rest_api.example_api.execution_arn}/*/*" # Restringe a permissão ao API Gateway especificado
}

resource "aws_api_gateway_integration" "lambda_integration" {
  rest_api_id = aws_api_gateway_rest_api.example_api.id # Vincula a integração à API
  resource_id = aws_api_gateway_resource.hello_resource.id # Vincula a integração ao recurso /hello
  http_method = aws_api_gateway_method.get_hello.http_method # Vincula ao método GET
  integration_http_method = "POST" # Define como o API Gateway se comunica com o backend (POST para AWS_PROXY)
  type        = "AWS_PROXY" # Tipo de integração: passa o evento diretamente para o Lambda
  uri         = aws_lambda_function.hello_lambda.invoke_arn # ARN para invocar a função Lambda
}

resource "aws_api_gateway_deployment" "example_deployment" {
  rest_api_id = aws_api_gateway_rest_api.example_api.id # API que será implantada
  stage_name  = "prod" # Nome do estágio da API (ex.: produção)

  depends_on = [aws_api_gateway_integration.lambda_integration] # Garante que a integração esteja configurada antes da implantação
}

resource "aws_api_gateway_stage" "example_stage" {
  rest_api_id          = aws_api_gateway_rest_api.example_api.id # API vinculada ao estágio
  stage_name           = "prod" # Nome do estágio
  deployment_id        = aws_api_gateway_deployment.example_deployment.id # ID da implantação
  cache_enabled        = true # Habilita cache para melhorar o desempenho
  cache_cluster_size   = "0.5" # Define o tamanho do cache
  variables = {
    env = "production" # Variável de ambiente configurada para o estágio
  }
}

resource "aws_cloudwatch_log_group" "api_logs" {
  name              = "/aws/api-gateway/example-api" # Nome do grupo de logs no CloudWatch
  retention_in_days = 7 # Tempo de retenção dos logs
}

resource "aws_iam_role" "lambda_exec" {
  name = "lambda_exec_role" # Nome do papel IAM

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": "lambda.amazonaws.com" # Serviço que pode assumir este papel
      },
      "Effect": "Allow",
      "Sid": ""
    }
  ]
}
EOF
}

resource "aws_iam_role_policy_attachment" "lambda_policy_attachment" {
  role       = aws_iam_role.lambda_exec.name # Papel ao qual a política será anexada
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole" # Política básica para execução do Lambda
}
```

---

## **Explicação do Código**

1. **Criação da API Gateway:**
   - `aws_api_gateway_rest_api`: Define uma API REST chamada `example-api`.
   - `aws_api_gateway_resource`: Cria um recurso `/hello` como parte do endpoint.

2. **Integração com Lambda:**
   - `aws_lambda_function`: Define uma função Lambda que será chamada quando o endpoint for acessado.
   - `aws_api_gateway_integration`: Conecta a API Gateway à função Lambda usando o modo **AWS_PROXY**.

3. **Permissões:**
   - `aws_lambda_permission`: Permite que o API Gateway invoque a função Lambda.

4. **Cache e Desempenho:**
   - `aws_api_gateway_stage`: Habilita o cache com `cache_enabled` e define o tamanho do cluster de cache.

5. **Monitoramento:**
   - `aws_cloudwatch_log_group`: Configura logs do API Gateway para análise de tráfego e erros.

6. **Controle de Acesso:**
   - `aws_iam_role`: Cria um papel IAM com permissões básicas para execução do Lambda.

---

## **Benefícios de Usar Terraform**

1. **Automação e Reprodutibilidade:**
   - Criação, atualização ou destruição do ambiente completo com um único arquivo.

2. **Gerenciamento de Configuração:**
   - Histórico de mudanças para acompanhar alterações.

3. **Facilidade de Escalabilidade:**
   - Adição de endpoints e estágios de forma padronizada.

4. **Integração com CICD:**
   - Automação em pipelines para diferentes ambientes (dev, staging, prod).

5. **Monitoramento Integrado:**
   - Configuração automática de logs e métricas.

---

Este exemplo ilustra como o **Terraform** pode simplificar e otimizar a criação e o gerenciamento de um serviço **AWS API Gateway** com integração Lambda, garantindo eficiência e controle total sobre a infraestrutura.
