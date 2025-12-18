#  Integração S3

Upload direto do frontend para AWS S3, ideal para vídeos grandes.

---

##  Por que usar S3?

### Vantagens

✅ **Upload mais rápido** - Cliente → S3 direto (sem passar pela API)  
✅ **Sem limite de tamanho** - Vídeos até 5GB  
✅ **Menor carga no servidor** - API apenas processa  
✅ **Resumable uploads** - Retomar upload em caso de falha  
✅ **CDN integration** - Distribuição global via CloudFront  

### Fluxo Tradicional vs S3

=== "Tradicional (Lento)"
    ```mermaid
    sequenceDiagram
        Frontend->>API: Upload 500MB
        Note over API: Recebe todo o vídeo
        API->>Gemini: Processa
        Gemini-->>API: Resultado
        API-->>Frontend: Response
    ```

=== "S3 (Rápido)"
    ```mermaid
    sequenceDiagram
        Frontend->>S3: Upload direto
        S3-->>Frontend: file_id
        Frontend->>API: analyze-from-s3
        API->>S3: Download (interno)
        API->>Gemini: Processa
        Gemini-->>API: Resultado
        API-->>Frontend: Response
    ```

---

##  Endpoints

### `POST /video/analyze-from-s3`

Analisa vídeo já armazenado no S3.

#### Request

```json
{
  "file_id": "2025_INSPER_CRIATIVO_6_V3",
  "audience": ["millennials"],
  "platform_type": ["meta"],
  "language": "pt"
}
```

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `file_id` | String | ✅ Sim | ID do vídeo no S3 (sem extensão) |
| `audience` | Array | ❌ Não | Lista de audiências |
| `platform_type` | Array | ❌ Não | Lista de plataformas |
| `language` | String | ❌ Não | `pt` ou `en` |

#### Response

```json
{
  "task_id": "a1b2c3d4-e5f6-7890",
  "status": "accepted",
  "message": "Análise do vídeo S3 iniciada...",
  "check_status_url": "/video/task/a1b2c3d4-e5f6-7890",
  "file_id": "2025_INSPER_CRIATIVO_6_V3"
}
```

---

### `GET /video/list-s3-videos`

Lista todos os vídeos disponíveis no bucket S3.

#### Request

```bash
curl "http://localhost:8000/video/list-s3-videos"
```

#### Response

```json
{
  "total_videos": 5,
  "file_ids": [
    "2025_INSPER_CRIATIVO_6_V3",
    "2025_NIKE_SUMMER_CAMPAIGN",
    "2025_COCA_COLA_HOLIDAY"
  ],
  "videos_details": [
    {
      "file_id": "2025_INSPER_CRIATIVO_6_V3",
      "key": "mp4/2025_INSPER_CRIATIVO_6_V3.mp4",
      "size": 52428800,
      "size_mb": 50.0,
      "last_modified": "2025-12-08T16:30:00",
      "filename": "2025_INSPER_CRIATIVO_6_V3.mp4"
    }
  ],
  "message": "5 vídeo(s) disponível(is) no S3"
}
```

---

### `GET /video/check-s3-video/{file_id}`

Verifica se um vídeo específico existe no S3.

#### Request

```bash
curl "http://localhost:8000/video/check-s3-video/2025_INSPER_CRIATIVO_6_V3"
```

#### Response - Encontrado

```json
{
  "file_id": "2025_INSPER_CRIATIVO_6_V3",
  "exists": true,
  "status": {
    "mp4": true,
    "mp3": false,
    "output": false,
    "metadata": false
  },
  "message": "Vídeo encontrado e pronto para processamento",
  "s3_path": "s3://test-bucket-video-klikeai/mp4/2025_INSPER_CRIATIVO_6_V3.mp4"
}
```

#### Response - Não Encontrado

**Status: 404 Not Found**

```json
{
  "detail": "Vídeo 2025_INSPER_CRIATIVO_6_V3.mp4 não encontrado no S3. Faça upload na pasta mp4/ do bucket."
}
```

---

## Estrutura do Bucket

O bucket S3 segue esta organização:

```
test-bucket-video-klikeai/
├── mp4/                     # Vídeos originais (obrigatório)
│   ├── 2025_INSPER_CRIATIVO_6_V3.mp4
│   ├── 2025_NIKE_SUMMER.mp4
│   └── ...
├── mp3/                     # Áudios extraídos (futuro)
│   └── 2025_INSPER_CRIATIVO_6_V3.mp3
├── outputs/                 # Transcrições (futuro)
│   └── 2025_INSPER_CRIATIVO_6_V3.txt
└── metadata/                # Metadados JSON (futuro)
    └── 2025_INSPER_CRIATIVO_6_V3.json
```

---

##  Implementação Frontend

### Upload Direto para S3

#### 1. Obter Presigned URL (Backend)

```python
from app.utils.s3 import S3Handler

@router.post("/video/get-upload-url")
async def get_upload_url(file_id: str):
    """
    Gera URL presigned para upload direto do frontend
    """
    s3_handler = S3Handler()
    
    # Gerar URL presigned para PUT
    url = s3_handler.s3_client.generate_presigned_url(
        'put_object',
        Params={
            'Bucket': s3_handler.bucket_name,
            'Key': f'mp4/{file_id}.mp4',
            'ContentType': 'video/mp4'
        },
        ExpiresIn=3600  # 1 hora
    )
    
    return {
        "upload_url": url,
        "file_id": file_id,
        "expires_in": 3600
    }
```

#### 2. Upload do Frontend (JavaScript)

```javascript
// 1. Gerar file_id único
const generateFileId = () => {
  const timestamp = new Date().toISOString().split('T')[0].replace(/-/g, '_');
  const random = Math.random().toString(36).substring(2, 8).toUpperCase();
  return `${timestamp}_VIDEO_${random}`;
};

// 2. Obter presigned URL
const getUploadUrl = async (fileId) => {
  const response = await fetch('/video/get-upload-url', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ file_id: fileId })
  });
  return await response.json();
};

// 3. Upload direto para S3
const uploadToS3 = async (file, uploadUrl) => {
  const response = await fetch(uploadUrl, {
    method: 'PUT',
    body: file,
    headers: {
      'Content-Type': 'video/mp4'
    }
  });
  
  if (!response.ok) {
    throw new Error('Falha no upload para S3');
  }
};

// 4. Iniciar análise
const analyzeFromS3 = async (fileId, options) => {
  const response = await fetch('/video/analyze-from-s3', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      file_id: fileId,
      ...options
    })
  });
  return await response.json();
};

// Fluxo completo
const processVideo = async (videoFile) => {
  try {
    // 1. Gerar ID
    const fileId = generateFileId();
    console.log('File ID:', fileId);
    
    // 2. Obter URL
    const { upload_url } = await getUploadUrl(fileId);
    console.log('Upload URL obtida');
    
    // 3. Upload
    await uploadToS3(videoFile, upload_url);
    console.log(' Upload concluído!');
    
    // 4. Análise
    const { task_id } = await analyzeFromS3(fileId, {
      audience: ['millennials'],
      platform_type: ['meta'],
      language: 'pt'
    });
    console.log(' Análise iniciada:', task_id);
    
    return task_id;
  } catch (error) {
    console.error(' Erro:', error);
    throw error;
  }
};
```

#### 3. Upload com Progress (React)

```jsx
import { useState } from 'react';

function VideoUploader() {
  const [progress, setProgress] = useState(0);
  const [taskId, setTaskId] = useState(null);

  const handleUpload = async (file) => {
    try {
      // 1. Gerar file_id
      const fileId = generateFileId();
      
      // 2. Obter presigned URL
      const { upload_url } = await getUploadUrl(fileId);
      
      // 3. Upload com progress
      const xhr = new XMLHttpRequest();
      
      xhr.upload.addEventListener('progress', (e) => {
        if (e.lengthComputable) {
          const percentComplete = (e.loaded / e.total) * 100;
          setProgress(Math.round(percentComplete));
        }
      });
      
      xhr.addEventListener('load', async () => {
        if (xhr.status === 200) {
          console.log(' Upload S3 concluído');
          
          // 4. Iniciar análise
          const { task_id } = await analyzeFromS3(fileId, {
            audience: ['millennials'],
            platform_type: ['meta'],
            language: 'pt'
          });
          
          setTaskId(task_id);
        }
      });
      
      xhr.open('PUT', upload_url);
      xhr.setRequestHeader('Content-Type', 'video/mp4');
      xhr.send(file);
      
    } catch (error) {
      console.error('Erro:', error);
    }
  };

  return (
    <div>
      <input 
        type="file" 
        accept="video/mp4" 
        onChange={(e) => handleUpload(e.target.files[0])} 
      />
      {progress > 0 && <progress value={progress} max="100" />}
      {taskId && <p>Task ID: {taskId}</p>}
    </div>
  );
}
```

---

##  Segurança

### Presigned URLs

- ✅ **Expiração:** URLs expiram em 1 hora
- ✅ **Escopo limitado:** Apenas PUT no caminho específico
- ✅ **Sem credenciais:** Frontend não precisa de AWS credentials

### Validação de File ID

```python
import re

def validate_file_id(file_id: str) -> bool:
    """
    Valida formato do file_id
    """
    # Apenas alfanuméricos, underscore e hífen
    pattern = r'^[A-Za-z0-9_-]+$'
    
    if not re.match(pattern, file_id):
        raise ValueError("File ID inválido")
    
    if len(file_id) > 100:
        raise ValueError("File ID muito longo")
    
    return True
```

---

##  Performance

### Comparação de Tempos

| Vídeo | Tradicional | S3 | Economia |
|-------|-------------|-----|----------|
| 50MB | ~30s | ~10s | **66%** |
| 100MB | ~60s | ~15s | **75%** |
| 500MB | ~5min | ~45s | **85%** |

### Processamento Paralelo

O backend usa **ThreadPoolExecutor** para processar em paralelo:

```python
with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
    future_upload = executor.submit(upload_to_gemini_task)
    future_subtitles = executor.submit(process_subtitles_task)
    future_aspect_ratio = executor.submit(process_aspect_ratio_task)
    
    uploaded = future_upload.result()
    subtitles = future_subtitles.result()
    aspect_ratio = future_aspect_ratio.result()
```

---

##  Troubleshooting

### Erro: "Vídeo não encontrado no S3"

**Verificar:**
1. File ID correto (sem espaços ou caracteres especiais)
2. Vídeo está em `mp4/` no bucket
3. Nome do bucket no `.env` está correto

```bash
# Verificar via AWS CLI
aws s3 ls s3://test-bucket-video-klikeai/mp4/
```

### Erro: "Access Denied"

**Solução:**
Verificar IAM policy do usuário:

```json
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "s3:PutObject",
    "s3:ListBucket"
  ],
  "Resource": [
    "arn:aws:s3:::test-bucket-video-klikeai",
    "arn:aws:s3:::test-bucket-video-klikeai/*"
  ]
}
```

### Upload lento para S3

**Otimizações:**
1. Use **multipart upload** para vídeos > 100MB
2. Configure **Transfer Acceleration** no bucket
3. Use região S3 mais próxima do usuário

---

##  Recursos Adicionais

- [AWS S3 Presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html)
- [Multipart Upload](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)
- [Transfer Acceleration](https://docs.aws.amazon.com/AmazonS3/latest/userguide/transfer-acceleration.html)

---

**Veja também:**

- [Análise de Vídeos](video-analysis.md)
- [Background Tasks](background-tasks.md)
- [Calculator API](calculator.md)