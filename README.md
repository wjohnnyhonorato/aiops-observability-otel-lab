# Ambiente de Observabilidade com OpenTelemetry Astronomy Shop

Este ambiente é um **laboratório local de microsserviços e observabilidade**.

Ele usa o projeto oficial **OpenTelemetry Astronomy Shop**, uma loja virtual fictícia criada para demonstrar como uma aplicação distribuída gera:

- traces;
- métricas;
- logs.

Tudo roda no próprio computador usando **Docker**.

---

## 1. O que é o Astronomy Shop

O Astronomy Shop é uma aplicação de e-commerce fictícia formada por vários microsserviços.

Exemplos:

```text
frontend
checkout
payment
cart
shipping
recommendation
product-catalog
email
```

Uma ação do usuário pode passar por vários serviços:

```text
Usuário
  ↓
frontend
  ↓
checkout
  ├── payment
  ├── shipping
  └── email
```

Isso cria um ambiente parecido com uma aplicação real baseada em microsserviços.

---

## 2. Para que usar este ambiente

O ambiente pode ser usado para estudar e testar:

- observabilidade;
- OpenTelemetry;
- distributed tracing;
- APM;
- métricas;
- logs;
- detecção de anomalias;
- análise de causa raiz (RCA);
- propagação de falhas;
- agentes de IA para investigação de incidentes.

A principal vantagem é que o ambiente é **controlado**.

É possível gerar tráfego, provocar falhas e verificar se um algoritmo conseguiu detectar e localizar o problema.

---

## 3. Onde entra o Docker

O Astronomy Shop possui vários componentes.

Em vez de instalar manualmente Python, Java, Node.js, PostgreSQL, Redis, Prometheus, Jaeger e outras dependências, o Docker executa cada componente em um ambiente isolado chamado **container**.

Exemplo:

```text
Windows
  ↓
Docker
  ├── container frontend
  ├── container checkout
  ├── container payment
  ├── container PostgreSQL
  ├── container OpenTelemetry Collector
  ├── container Jaeger
  ├── container Prometheus
  └── container Grafana
```

Em geral, cada microsserviço roda em seu próprio container.

---

## 4. Docker Compose

O **Docker Compose** coordena os containers do projeto.

Ele lê arquivos de configuração e sabe:

- quais containers criar;
- quais imagens usar;
- quais portas abrir;
- como os serviços se comunicam;
- quais dependências existem entre eles.

Resumo:

```text
Image     = pacote/molde do programa
Container = uma instância desse pacote rodando
Compose   = configuração que sobe e conecta vários containers
```

---

## 5. Pré-requisitos

No Windows:

- Git;
- Docker Desktop;
- VS Code, opcional;
- aproximadamente 4 GB ou mais de memória disponível.

Verifique a instalação:

```powershell
docker --version
docker compose version
```

---

## 6. Baixar o projeto

```powershell
git clone https://github.com/open-telemetry/opentelemetry-demo.git
cd opentelemetry-demo
```

---

## 7. Iniciar o ambiente

Dentro da pasta `opentelemetry-demo`:

```powershell
docker compose --env-file .env --env-file .env.override -f compose.yaml -f compose.observability.yaml -f compose.extras.yaml up --force-recreate --remove-orphans --detach
```

O Docker irá:

1. baixar as imagens necessárias;
2. criar os containers;
3. criar a rede entre eles;
4. iniciar os microsserviços;
5. iniciar as ferramentas de observabilidade.

Na primeira execução isso pode levar alguns minutos.

---

## 8. Verificar se está funcionando

Para ver os containers do projeto:

```powershell
docker compose ps
```

Exemplo:

```text
frontend        Up (healthy)
checkout        Up (healthy)
payment         Up (healthy)
jaeger          Up
prometheus      Up
otel-collector  Up
```

`Up` significa que o processo está rodando.

`healthy` significa que o serviço também passou no teste de saúde configurado.

Se quiser consultar especificamente serviços dos arquivos adicionais, use os mesmos arquivos usados no `up`:

```powershell
docker compose --env-file .env --env-file .env.override -f compose.yaml -f compose.observability.yaml -f compose.extras.yaml ps
```

---

## 9. Abrir a loja

No navegador:

```text
http://localhost:8080
```

Essa é a aplicação Astronomy Shop.

O ambiente também possui um `load-generator`, que simula usuários usando a loja automaticamente.

Por isso traces, métricas e logs são gerados mesmo sem uso manual.

---

## 10. Como a observabilidade funciona

Fluxo simplificado:

```text
microsserviços
     ↓
OpenTelemetry
     ↓
OpenTelemetry Collector
     ↓
traces / métricas / logs
     ↓
ferramentas de observabilidade
```

O **OpenTelemetry Collector** recebe a telemetria dos serviços e encaminha os dados para outras ferramentas.

---

# 11. Ferramentas do ambiente

## 11.1 Jaeger

### O que faz

O Jaeger é usado para visualizar **distributed traces**.

Uma trace representa a jornada completa de uma requisição.

Um span representa uma operação dentro dessa jornada.

Exemplo:

```text
Trace
└── frontend-web
    └── frontend-proxy
        └── frontend
            └── recommendation
                └── product-catalog
```

### Como abrir

```text
http://localhost:8080/jaeger/ui/
```

### 3 exemplos de uso

1. **Ver por quais serviços uma requisição passou.**

```text
frontend → recommendation → product-catalog
```

2. **Encontrar onde houve aumento de latência.**

```text
frontend             800 ms
recommendation       650 ms
product-catalog      600 ms
```

3. **Investigar erros em uma trace.**

```text
payment
status = ERROR
```

### Campos importantes

```text
trace_id
span_id
parent_span_id
service
operation
duration
status
attributes
```

Ao filtrar `Service = frontend`, o Jaeger procura traces que **contêm spans do frontend**.

Isso não significa que a trace começou no `frontend`.

---

## 11.2 Prometheus

### O que faz

O Prometheus armazena e consulta principalmente **métricas de séries temporais**.

Exemplos:

```text
requisições por segundo
latência
taxa de erros
CPU
memória
```

### Como abrir

```text
http://localhost:9090
```

### 3 exemplos de uso

1. **Consultar quantidade de requisições ao longo do tempo.**

2. **Observar aumento de latência de um serviço.**

```text
payment: 30 ms → 500 ms
```

3. **Acompanhar uso de CPU ou memória.**

```text
CPU: 20% → 90%
```

O Prometheus é mais voltado para consulta e armazenamento de métricas do que para dashboards prontos.

---

## 11.3 Grafana

### O que faz

O Grafana é uma **interface de visualização e exploração**.

Ele pode consultar diferentes fontes de dados e apresentar gráficos, dashboards e consultas.

No laboratório, ele funciona como uma camada visual sobre dados de observabilidade.

### Como abrir

Use o endereço disponibilizado pelo proxy da demo:

```text
http://localhost:8080/grafana/
```

### 3 exemplos de uso

1. **Criar um gráfico de latência por serviço.**

```text
frontend
payment
checkout
```

2. **Montar um dashboard com volume, erros e latência.**

```text
requests/s
error rate
p95 latency
```

3. **Explorar logs ou métricas de uma anomalia em uma mesma interface.**

O Grafana **não é o banco de dados**.

Ele consulta dados armazenados em ferramentas como Prometheus ou OpenSearch.

---

## 11.4 OpenSearch

### O que faz

O OpenSearch é um mecanismo de **armazenamento, indexação e busca de dados**.

Neste ambiente ele é usado principalmente como backend para dados como logs.

Ele não deve ser confundido com o Grafana:

```text
OpenSearch = armazena e consulta dados
Grafana    = visualiza dados
```

### Como abrir

O OpenSearch desta demo não possui uma interface gráfica própria instalada.

A porta externa pode mudar quando os containers são recriados.

Descubra a porta atual:

```powershell
docker ps --filter "name=opensearch"
```

Você verá algo parecido com:

```text
0.0.0.0:64517->9200/tcp
```

Nesse exemplo, abra:

```text
http://localhost:64517
```

Você verá uma resposta em **JSON**. Isso é esperado.

Também pode testar pelo terminal:

```powershell
curl http://localhost:64517
```

### 3 exemplos de uso

1. **Buscar logs de um serviço.**

```text
service = payment
```

2. **Buscar mensagens de erro em determinado período.**

```text
ERROR
timeout
connection refused
```

3. **Relacionar logs de vários serviços durante um incidente.**

```text
checkout
payment
shipping
```

Para uma exploração visual, é normalmente mais confortável usar o Grafana conectado ao OpenSearch.

---

## 11.5 OpenTelemetry Collector

### O que faz

O OpenTelemetry Collector é o componente que **recebe, processa e encaminha telemetria**.

Fluxo simplificado:

```text
microsserviços
      ↓
OpenTelemetry Collector
      ├── traces
      ├── métricas
      └── logs
```

Ele é uma peça de infraestrutura, não uma ferramenta de dashboard.

### Como abrir

O Collector **não possui uma interface visual principal** neste ambiente.

Para verificar se está rodando:

```powershell
docker ps --filter "name=otel-collector"
```

Para ver seus logs:

```powershell
docker logs otel-collector --tail 50
```

### 3 exemplos de uso

1. **Receber traces enviados pelos microsserviços.**

2. **Encaminhar métricas para o backend de métricas.**

3. **Processar ou transformar atributos antes de enviar a telemetria.**

Exemplo conceitual:

```text
service.name = payment
      ↓
Collector
      ↓
backend de observabilidade
```

---

## 11.6 Load Generator

### O que faz

O Load Generator simula usuários utilizando a loja.

Ele cria tráfego automaticamente.

Assim, o sistema continua gerando traces, métricas e logs mesmo sem ninguém clicar manualmente.

### Como abrir

```text
http://localhost:8080/loadgen/
```

### 3 exemplos de uso

1. **Gerar tráfego normal continuamente.**

2. **Criar volume suficiente para observar métricas.**

3. **Manter carga durante um experimento de falha.**

Exemplo:

```text
carga normal
    ↓
ativar falha
    ↓
continuar gerando requisições
    ↓
observar impacto
```

---

## 11.7 Feature Flags

### O que faz

As Feature Flags permitem alterar comportamentos do sistema e ativar **falhas controladas**.

Isso é especialmente útil para testes de anomalia e RCA.

### Como abrir

```text
http://localhost:8080/feature
```

### 3 exemplos de uso

1. **Provocar erro em um serviço.**

2. **Criar aumento de CPU ou outro comportamento anormal disponível na demo.**

3. **Comparar um período normal com um período de falha conhecida.**

Exemplo:

```text
NORMAL
  ↓
ativar feature flag de falha
  ↓
ANÔMALO
  ↓
detecção
  ↓
RCA
```

Como a falha foi ativada manualmente, sua causa é conhecida.

Isso fornece um **ground truth** para testar algoritmos.

---

# 12. Resumo rápido das ferramentas

| Ferramenta | Principal função | Como acessar |
|---|---|---|
| Astronomy Shop | Aplicação que gera o tráfego | `http://localhost:8080` |
| Jaeger | Visualizar traces | `http://localhost:8080/jaeger/ui/` |
| Prometheus | Consultar métricas | `http://localhost:9090` |
| Grafana | Dashboards e exploração visual | `http://localhost:8080/grafana/` |
| OpenSearch | Armazenar e buscar dados/logs | porta dinâmica → `docker ps --filter "name=opensearch"` |
| OTEL Collector | Receber e encaminhar telemetria | sem UI; usar `docker logs` |
| Load Generator | Simular usuários | `http://localhost:8080/loadgen/` |
| Feature Flags | Ativar falhas controladas | `http://localhost:8080/feature` |

---

## 13. Exemplo de uso para RCA

Um experimento simples:

```text
Astronomy Shop
      ↓
tráfego normal
      ↓
coletar traces e métricas
      ↓
injetar falha
      ↓
observar mudança de latência e erros
      ↓
executar algoritmo de RCA
      ↓
comparar resultado com a falha conhecida
```

Exemplo:

```text
falha conhecida:
payment

resultado do RCA:
1. payment
2. checkout
3. frontend
```

Como existe um ground truth, podem ser calculadas métricas como:

```text
Hit@1
Hit@3
MRR
tempo até detecção
```

---

## 14. Parar e iniciar novamente

Como o ambiente foi iniciado usando vários arquivos Compose, é mais seguro reutilizar os mesmos arquivos.

### Parar

```powershell
docker compose --env-file .env --env-file .env.override -f compose.yaml -f compose.observability.yaml -f compose.extras.yaml stop
```

### Iniciar novamente

```powershell
docker compose --env-file .env --env-file .env.override -f compose.yaml -f compose.observability.yaml -f compose.extras.yaml start
```

### Remover os containers do ambiente

```powershell
docker compose --env-file .env --env-file .env.override -f compose.yaml -f compose.observability.yaml -f compose.extras.yaml down
```

O comando `down` remove os containers e a rede criada pelo Compose.

As imagens baixadas não são automaticamente apagadas.

---

## 15. Comandos úteis

```powershell
# Ver versão do Docker
docker --version

# Ver versão do Docker Compose
docker compose version

# Ver todos os containers em execução
docker ps

# Ver containers do projeto
docker compose ps

# Ver apenas o Jaeger
docker ps --filter "name=jaeger"

# Ver apenas o Grafana
docker ps --filter "name=grafana"

# Ver apenas o OpenSearch
docker ps --filter "name=opensearch"

# Ver apenas o OpenTelemetry Collector
docker ps --filter "name=otel-collector"

# Ver logs de um container
docker logs NOME_DO_CONTAINER --tail 50
```

---

## 16. Visão geral

```text
                       Windows
                          │
                    Docker Desktop
                          │
                    Docker Compose
                          │
          ┌───────────────┴────────────────┐
          │                                │
   Astronomy Shop                    Observabilidade
          │                                │
    microsserviços                    OpenTelemetry
          │                                │
          │                         OTEL Collector
          │                                │
          │               ┌────────────────┼───────────────┐
          │               │                │               │
          └──────────→  traces          métricas          logs
                          │                │               │
                       Jaeger         Prometheus       OpenSearch
                                           \             /
                                            \           /
                                               Grafana
```

O objetivo deste ambiente é permitir estudar uma aplicação distribuída realista sem depender de sistemas ou dados de produção.
