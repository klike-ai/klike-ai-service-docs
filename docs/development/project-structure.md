# Estrutura do Projeto

Documentação completa da organização e arquitetura de código do Klike AI Services.

##  Visão Geral

```
klike-ai-services/
├── app/                          # Código principal da aplicação
│   ├── __init__.py
│   ├── app.py                    # FastAPI application entry point
│   ├── dependencies.py           # Injeção de dependências
│   ├── audio/                    # Módulo de processamento de áudio
│   ├── managers/                 # Gerenciamento de tasks e workers
│   ├── prompts/                  # Templates de prompts para IA
│   ├── routers/                  # Endpoints da API
│   ├── schemas/                  # Modelos Pydantic
│   ├── services/                 # Lógica de negócio
│   ├── static/                   # Arquivos estáticos (HTML, CSS, JS)
│   ├── utils/                    # Utilitários e helpers
│   └── validators/               # Validadores customizados
├── data/                         # Dados de desenvolvimento/teste
│   └── mp4/                      # Vídeos de exemplo
├── docs/                         # Documentação MkDocs
│   ├── api/                      # Docs da API
│   ├── architecture/             # Docs de arquitetura
│   ├── development/              # Guias para desenvolvedores
│   |── getting-started/          # Guias de início
│   
├── env/                          # Virtual environment (não versionado)
├── notebooks/                    # Jupyter notebooks para experimentação
├── scripts/                      # Scripts de automação
│   ├── build-docs.ps1
│   ├── deploy-docs.ps1
│   └── serve-docs.ps1
├── .github/                      # GitHub Actions workflows
│   └── workflows/
│       └── deploy-docs.yml
├── .env                          # Variáveis de ambiente (não versionado)
├── .gitignore
├── Dockerfile                    # Container Docker
├── mkdocs.yml                    # Configuração da documentação
├── PUBLISHING_GUIDE.md           # Guia de publicação
├── README.md                     # README principal
└── requirements.txt              # Dependências Python
```

##  Principais Diretórios

### `/app` - Aplicação Principal

```
app/
├── app.py                        # Entry point, configuração FastAPI
├── dependencies.py               # Dependency injection
├── audio/                        
│   └── audio_analyzer.py         # Análise de características de áudio
├── managers/
│   └── task_manager.py           # Gerenciamento de tasks assíncronas
├── prompts/
│   ├── en/                       # Prompts em inglês
│   │   ├── compliance.txt
│   │   ├── detected_issue.txt
│   │   └── audiences/
│   └── pt/                       # Prompts em português
│       ├── compliance.txt
│       ├── detected_issue.txt
│       └── platforms/
├── routers/
│   ├── __init__.py
│   ├── calculator_router.py      # Endpoints da calculadora
│   └── video_router.py           # Endpoints de vídeo
├── schemas/
│   ├── base_schema_video.py      # Modelos de vídeo
│   └── calculator_schema.py      # Modelos da calculadora
├── services/
│   ├── action_generation_service.py
│   ├── background_processor_service.py
│   ├── prompt_builder_service.py
│   ├── recommendation_service.py
│   ├── s3_video_service.py
│   ├── scoring_service.py
│   ├── video_analysis_service.py
│   └── video_upload_service.py
├── static/
│   └── index.html                # Documentação interativa Swagger
├── utils/
│   ├── actions.py
│   ├── actions_extra_content_prompts.py
│   ├── actions_filters.py
│   ├── actions_formatters.py
│   ├── actions_prompts_config.py
│   ├── audio_extractor.py
│   ├── budget_calculator.py
│   ├── s3.py
│   ├── subtitle_processor.py
│   ├── transcribe_audio.py
│   ├── utils.py
│   └── video_metadata_extractor.py
└── validators/
    └── video_validator.py
```

### `/docs` - Documentação

Documentação técnica e guias do usuário usando MkDocs Material.

```
docs/
├── index.md                      # Homepage
├── api/                          # Referência da API
│   ├── overview.md
│   ├── video-analysis.md
│   ├── s3-integration.md
│   ├── background-tasks.md
│   └── calculator.md
├── architecture/                 # Arquitetura do sistema
│   ├── overview.md
│   ├── data-flow.md
│   ├── services.md
│   └── integrations.md
├── development/                  # Para desenvolvedores
│   └── project-structure.md
│   
└── getting-started/              # Início rápido
    ├── installation.md
    ├── configuration.md
    └── quickstart.md

```

### `/notebooks` - Experimentação

Jupyter notebooks para prototipagem e análise exploratória.

```
notebooks/
├── functions_recommendation.ipynb
├── gemini_api_files.ipynb
├── generate_actions_for_issues.ipynb
├── generate_audience_actions.ipynb
├── generate_plataforms.ipynb
├── s3.ipynb
└── teste_params_video.ipynb
```

### `/scripts` - Automação

Scripts PowerShell para tarefas comuns de desenvolvimento.

```
scripts/
├── README.md                     # Documentação dos scripts
├── build-docs.ps1                # Build documentação
├── deploy-docs.ps1               # Deploy documentação
└── serve-docs.ps1                # Serve documentação localmente
```

##  Arquitetura em Camadas

### 1. Presentation Layer (Routers)

**Responsabilidade:** Receber requisições HTTP, validar entrada, retornar respostas.

**Arquivos:**

- `routers/video_router.py`
- `routers/calculator_router.py`

**Exemplo:**

```python
# routers/video_router.py
from fastapi import APIRouter, Depends, UploadFile
from app.schemas.base_schema_video import VideoAnalysisRequest
from app.services.video_analysis_service import VideoAnalysisService

router = APIRouter(prefix="/api/v1/videos", tags=["videos"])

@router.post("/analyze")
async def analyze_video(
    request: VideoAnalysisRequest,
    service: VideoAnalysisService = Depends()
):
    """Endpoint para análise de vídeo"""
    result = await service.analyze(request)
    return result
```

### 2. Business Logic Layer (Services)

**Responsabilidade:** Implementar regras de negócio, orquestrar operações.

**Arquivos:**

- `services/video_analysis_service.py`
- `services/scoring_service.py`
- `services/recommendation_service.py`
- `services/prompt_builder_service.py`

**Padrão:**

```python
# services/video_analysis_service.py
class VideoAnalysisService:
    def __init__(
        self,
        s3_service: S3VideoService,
        scoring_service: ScoringService,
        recommendation_service: RecommendationService
    ):
        self.s3 = s3_service
        self.scoring = scoring_service
        self.recommendations = recommendation_service
    
    async def analyze(self, video_id: str) -> AnalysisResult:
        # Orquestrar todo o fluxo de análise
        video = await self.s3.download(video_id)
        transcript = await self.transcribe(video)
        scores = await self.scoring.calculate(transcript)
        recs = await self.recommendations.generate(scores)
        
        return AnalysisResult(
            scores=scores,
            recommendations=recs
        )
```

### 3. Data Access Layer (Utils)

**Responsabilidade:** Interagir com serviços externos, I/O, processamento de dados.

**Arquivos:**

- `utils/s3.py` - Cliente AWS S3
- `utils/transcribe_audio.py` - Integração Whisper
- `utils/audio_extractor.py` - FFmpeg wrapper
- `utils/video_metadata_extractor.py` - Extração de metadados

### 4. Domain Layer (Schemas)

**Responsabilidade:** Definir modelos de dados, validação.

**Arquivos:**

- `schemas/base_schema_video.py`
- `schemas/calculator_schema.py`

**Exemplo:**

```python
# schemas/base_schema_video.py
from pydantic import BaseModel, Field
from typing import Optional, List

class VideoAnalysisRequest(BaseModel):
    video_id: str = Field(..., description="UUID do vídeo")
    platform: str = Field(..., description="Plataforma alvo")
    audience: str = Field(..., description="Audiência alvo")
    
    class Config:
        json_schema_extra = {
            "example": {
                "video_id": "550e8400-e29b-41d4-a716-446655440000",
                "platform": "instagram",
                "audience": "jovens_adultos"
            }
        }
```

##  Padrões de Código

### Dependency Injection

```python
# dependencies.py
from functools import lru_cache
from app.services.video_analysis_service import VideoAnalysisService

@lru_cache()
def get_video_analysis_service() -> VideoAnalysisService:
    return VideoAnalysisService()

# Uso nos routers
@router.post("/analyze")
async def analyze(
    request: VideoAnalysisRequest,
    service: VideoAnalysisService = Depends(get_video_analysis_service)
):
    return await service.analyze(request)
```

### Error Handling

```python
# routers/video_router.py
from fastapi import HTTPException

@router.post("/upload")
async def upload_video(file: UploadFile):
    try:
        result = await service.upload(file)
        return result
    except VideoTooLargeError as e:
        raise HTTPException(
            status_code=413,
            detail={
                "error": "FILE_TOO_LARGE",
                "message": str(e),
                "max_size_mb": 500
            }
        )
    except Exception as e:
        logger.exception("Unexpected error during upload")
        raise HTTPException(
            status_code=500,
            detail={"error": "INTERNAL_ERROR", "message": "Upload failed"}
        )
```

### Logging

```python
import logging

logger = logging.getLogger(__name__)

class VideoAnalysisService:
    async def analyze(self, video_id: str):
        logger.info(f"Starting analysis for video {video_id}")
        
        try:
            result = await self._process(video_id)
            logger.info(f"Analysis completed for {video_id}")
            return result
        except Exception as e:
            logger.error(f"Analysis failed for {video_id}: {e}")
            raise
```

### Configuration

```python
# dependencies.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    google_gemini_api_key: str
    openai_api_key: str
    aws_access_key_id: str
    aws_secret_access_key: str
    s3_bucket_name: str
    supabase_url: str
    supabase_key: str
    
    class Config:
        env_file = ".env"

@lru_cache()
def get_settings() -> Settings:
    return Settings()
```

### Código Python

```python
# Classes: PascalCase
class VideoAnalysisService:
    pass

# Funções/métodos: snake_case
def analyze_video():
    pass

# Constantes: UPPER_SNAKE_CASE
MAX_VIDEO_SIZE_MB = 500

# Variáveis: snake_case
video_id = "abc123"

# Privados: prefixo _
def _internal_method():
    pass
```


##  Recursos Adicionais

- [Arquitetura](../architecture/overview.md) - Visão geral da arquitetura
