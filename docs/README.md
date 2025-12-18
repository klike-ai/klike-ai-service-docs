# Klike AI Services Documentation

Documentação oficial do Klike AI Services construída com MkDocs Material.

##  Visualizar Localmente

```bash
# Instalar dependências
pip install -r docs/requirements.txt

# Servir documentação localmente
mkdocs serve

# Acessar: http://127.0.0.1:8000
```

##  Build

```bash
# Gerar site estático
mkdocs build

# Arquivos gerados em: site/
```

##  Deploy no GitHub Pages

```bash
# Deploy automático
mkdocs gh-deploy

# Acessível em: https://klike-ai.github.io/klike-ai-services
```

##  Estrutura

```
docs/
├── index.md                    # Homepage
├── getting-started/
│   ├── installation.md         # Instalação
│   ├── configuration.md        # Configuração
│   └── quickstart.md          # Início rápido
├── api/
│   ├── overview.md            # Visão geral da API
│   ├── video-analysis.md      # Endpoints de análise
│   ├── s3-integration.md      # Integração S3
│   ├── background-tasks.md    # Background tasks
│   └── calculator.md          # Calculadora
├── architecture/
│   ├── overview.md            # Visão geral da arquitetura
│   ├── data-flow.md           # Fluxo de dados
│   ├── services.md            # Serviços
│   └── integrations.md        # Integrações externas
└── development/
    └── project-structure.md   # Estrutura do projeto
    
```

##  Customização

Edite `mkdocs.yml` para alterar:
- Cores e tema
- Navegação
- Plugins
- Extensões Markdown

##  Recursos

- [MkDocs](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- [PyMdown Extensions](https://facelessuser.github.io/pymdown-extensions/)
