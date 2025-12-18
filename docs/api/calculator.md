# Calculadora de Orçamento

A API inclui uma calculadora inteligente para estimar custos de produção de vídeos publicitários baseados em diversos parâmetros.

## Visão Geral

A calculadora considera:

- **Duração do vídeo**
- **Qualidade de produção** (básica, profissional, premium)
- **Tipo de vídeo** (UGC, animação, filmagem, etc.)
- **Recursos necessários** (locações, atores, edição, etc.)
- **Região geográfica**

## Endpoint

```
POST /calculate-budget
```

## Autenticação

Não requer autenticação (endpoint público).

##  Requisição

### Parâmetros
| Parâmetro          | Tipo     | Descrição                                      | Obrigatório |
|--------------------|----------|------------------------------------------------|-------------|
| `monthly_revenue_goal` | `float`  | Meta de receita mensal desejada (em USD)       | ✅ Sim      |
| `avg_ticket_price`     | `float`  | Preço médio por venda (em USD)                  | ✅ Sim      |
|     `score`               | `int`    | Score de qualidade desejado (0-100)            | ✅ Sim     |


### Exemplo de Requisição

```python
import requests
data = {
    "monthly_revenue_goal": 10000.0,
    "avg_ticket_price": 50.0,
    "score": 85
}
response = requests.post(
    "http://localhost:8000/calculate-budget",
    json=data
)
budget_estimate = response.json()
print(budget_estimate)
```
## Resposta
### Exemplo de Resposta

```json
{
  "budget_range": {
    "min_budget": 5600,
    "max_budget": 9400
  }
}
```


### Descrição dos Campos

| Campo            | Tipo     | Descrição                                      |
|------------------|----------|------------------------------------------------|
| `min_budget`     | `float`  | Orçamento mínimo estimado (em USD)             |
| `max_budget`     | `float`  | Orçamento máximo estimado (em USD)             |




##  Próximos Passos

- [Análise de Vídeos](video-analysis.md) - Analise vídeos existentes
- [Upload para S3](s3-integration.md) - Envie vídeos diretamente para S3
- [Tarefas em Background](background-tasks.md) - Gerencie análises assíncronas
