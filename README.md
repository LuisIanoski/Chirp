# Chirp - Sistema de Monitoramento Ambiental utilizando Visão Computacional

Aplicação **Fullstack** para monitoramento em tempo real com detecção de objetos usando YOLO, Django e WebSockets. Monitore múltiplas câmeras IP simultaneamente através de uma interface web intuitiva com sistema de gerenciamento de risco.

## Eventos

- Apresentado em:
  - **SIPEX 2025** - Irati
  - **Paraná Faz Ciência** 2025 - Guarapuava
  - **XSEPIN** - Foz Do Iguaçu

## Tecnologias

- **Backend:**
  - Python 3.12+
  - Django 5.2.1
  - Django REST Framework
  - Django Channels (WebSockets)
  - Gunicorn

- **Frontend:**
  - HTML5
  - CSS3
  - JavaScript

- **Visão Computacional:**
  - OpenCV
  - YOLO (ultralytics)
  - PyTorch
  - NumPy

- **Banco de Dados:**
  - MySQL 8.0+

- **Utilitários:**
  - python-dotenv
  - mysqlclient

## Pré-requisitos

- Python 3.12 ou superior
- MySQL 8.0 ou superior
- Ambiente virtual Python (venv)
- Git
- Câmeras IP com acesso por RTSP ou HTTP
- Protótipo para medição do nível do rio.

## Instalação

### 1. Clone o repositório:
```bash
git clone https://github.com/LuisIanoski/Chirp.git
cd Chirp
```

### 2. Configure o ambiente virtual:
```bash
# Windows
python -m venv venv
venv\Scripts\activate

# Linux/Mac
python -m venv venv
source venv/bin/activate
```

### 3. Instale as dependências:
```bash
pip install -r requirements.txt
```

### 4. Configure as variáveis de ambiente:
```bash
# Copie o arquivo de exemplo
cp .env.example .env

# Edite o arquivo .env com suas configurações
```

Variáveis necessárias no `.env`:
```
DEBUG=False
SECRET_KEY=sua-chave-secreta-aqui
DB_NAME=chirp_db
DB_USER=seu_usuario_mysql
DB_PASSWORD=sua_senha_mysql
DB_HOST=localhost
DB_PORT=3306
ALLOWED_HOSTS=localhost,127.0.0.1
```

### 5. Execute as migrações do banco de dados:
```bash
python manage.py makemigrations
python manage.py migrate
```

### 6. Crie um superusuário para acesso admin:
```bash
python manage.py createsuperuser
```

### 7. Inicie o servidor Django:
```bash
# Desenvolvimento
python manage.py runserver

# Produção (com Gunicorn)
gunicorn extracaoimg.wsgi:application --bind 0.0.0.0:8000
```

O servidor estará disponível em `http://localhost:8000`

## Estrutura do Projeto

```
Chirp/
├── camera/                    # App Django - Gerenciamento de câmeras e detecção
│   ├── models.py             # Modelo de câmeras
│   ├── views.py              # ViewSets e endpoints de câmeras
│   ├── serializers.py        # Serialização de dados
│   ├── urls.py               # Rotas da API de câmeras
│   ├── object_detector.py    # Lógica de detecção YOLO
│   └── services.py           # Serviços de processamento de vídeo
│
├── monitoramento/            # App Django - Lógica de monitoramento e streaming
│   ├── models.py             # Modelo de detecções
│   ├── views.py              # Endpoints de detecções e streaming
│   ├── serializers.py        # Serialização de detecções
│   ├── urls.py               # Rotas de monitoramento
│   ├── camera.py             # Classe para gerenciamento de streams
│   ├── video_extractor.py    # Extração de frames de vídeo
│   └── json_converter.py     # Conversão de dados para JSON
│
├── dashboard/                # App Django - Interface Web e gerenciamento de risco
│   ├── models.py             # Modelo Dashboard com níveis de risco
│   ├── views.py              # Views da interface web e API de dashboards
│   ├── serializers.py        # Serialização de dados de dashboard
│   ├── urls.py               # Rotas de dashboards
│   └── admin.py              # Configuração do painel admin
│
├── extracaoimg/              # Configurações do projeto Django
│   ├── settings.py           # Configurações gerais
│   ├── urls.py               # Roteamento principal
│   ├── wsgi.py               # Configuração WSGI para produção
│   └── asgi.py               # Configuração ASGI para WebSockets
│
├── templates/                # Templates HTML frontend
│   ├── dashboard.html        # Interface principal de monitoramento
│   ├── dashboard_management.html  # Gerenciamento de dashboards e câmeras
│   └── risco.html            # Página de gerenciamento de níveis de risco
│
├── static/                   # Arquivos estáticos (CSS, JS, imagens)
├── manage.py                 # Gerenciador Django
├── requirements.txt          # Dependências do projeto
└── .env.example             # Arquivo de exemplo de variáveis de ambiente
```

## Interface Web

### Páginas Disponíveis:

#### 1. **Dashboard Principal** (`/dashboards/<dashboard_id>/`)
- Visualização de todas as câmeras vinculadas
- Stream em tempo real de vídeos
- Status de cada câmera (ativo/inativo/erro)
- Indicador visual de nível de risco do dashboard
- Adição/remoção de câmeras (máximo 3 câmeras por dashboard)

#### 2. **Gerenciamento de Dashboard** (`/dashboards/<dashboard_id>/management/`)
- Criar e editar dashboards
- Adicionar câmeras via link RTSP/HTTP
- Gerenciar configurações de monitoramento
- Visualizar histórico de alterações

#### 3. **Gerenciamento de Risco** (`/dashboards/<dashboard_id>/risco/`)
- Visualizar nível atual de risco: **Baixo** (Verde), **Médio** (Amarelo), **Alto** (Vermelho), **Crítico** (Vermelho Escuro)
- Alterar nível de risco conforme necessário
- Descrição detalhada de cada nível de risco
- Histórico de mudanças de risco

## Autenticação e Permissões

- **Operações GET:** Públicas (sem autenticação)
- **Operações POST, PUT, DELETE:** Requerem autenticação via token ou sessão
- **Painel Admin:** Disponível em `/admin`

## Endpoints da API

### Câmeras

| Método | Endpoint | Descrição | Auth |
|--------|----------|-----------|------|
| GET | `/api/cameras/` | Lista todas as câmeras | Não |
| POST | `/api/cameras/` | Adiciona nova câmera | Sim |
| GET | `/api/cameras/{id}/` | Detalhes de uma câmera | Não |
| PUT | `/api/cameras/{id}/` | Atualiza câmera | Sim |
| DELETE | `/api/cameras/{id}/` | Remove câmera | Sim |
| GET | `/api/cameras/{id}/stream/` | Stream ao vivo (MJPEG) | Não |

### Dashboards

| Método | Endpoint | Descrição | Auth |
|--------|----------|-----------|------|
| GET | `/api/dashboards/` | Lista todos os dashboards | Não |
| POST | `/api/dashboards/` | Cria novo dashboard | Não |
| GET | `/api/dashboards/{id}/` | Detalhes do dashboard | Não |
| PUT | `/api/dashboards/{id}/` | Atualiza dashboard | Não |
| DELETE | `/api/dashboards/{id}/` | Remove dashboard | Não |
| PATCH | `/api/dashboards/{id}/update-risk/` | Atualiza nível de risco | Não |
| GET | `/api/dashboards/{id}/risk-history/` | Histórico de risco | Não |
| GET | `/api/dashboards/risk-levels/` | Lista níveis de risco | Não |

### Detecções

| Método | Endpoint | Descrição | Auth |
|--------|----------|-----------|------|
| GET | `/api/detections/` | Lista todas as detecções | Não |
| GET | `/api/detections/{id}/` | Detalhes de uma detecção | Não |

## Modelos de Dados

### Camera
```json
{
    "camera_id": "string (único)",
    "camera_link": "string (URL RTSP/HTTP)",
    "camera_status": "string (active/inactive/error)",
    "camera_loc": "string (localização física)",
    "created_at": "datetime",
    "updated_at": "datetime"
}
```

### Dashboard
```json
{
    "dashboard_id": "string (único)",
    "name": "string",
    "risco": "string (baixo/medio/alto/critico)",
    "cameras": ["array de IDs de câmeras"],
    "created_at": "datetime",
    "updated_at": "datetime"
}
```

### Detection
```json
{
    "detection_id": "int (automático)",
    "camera_id": "string",
    "detection_date": "date (DD/MM/YYYY)",
    "detection_time": "time (HH:MM:SS)",
    "detections": [
        {
            "class": "string (objeto detectado)",
            "confidence": "float (0-1)",
            "bbox": [x1, y1, x2, y2]
        }
    ]
}
```

## Configurações Importantes

### Configurações de Detecção (em `camera/object_detector.py`):
- **Confiança mínima:** 0.45 (ajustável)
- **Modelo YOLO:** YOLO11s (pode ser alterado para YOLOv8, YOLOv10, etc.)
- **Intervalo entre frames:** 15 segundos (ajustável em `camera/services.py`)

### Configurações de WebSocket (em `extracaoimg/settings.py`):
- Habilitado via Django Channels
- Suporta múltiplas conexões simultâneas
- Broadcast de detecções em tempo real

### Configurações de Produção:
```bash
# Usar Gunicorn com múltiplos workers
gunicorn extracaoimg.wsgi:application --workers 4 --bind 0.0.0.0:8000

# Para WebSockets em produção, usar Daphne
daphne -b 0.0.0.0 -p 8000 extracaoimg.asgi:application
```

## Níveis de Risco

| Nível | Cor | Descrição | Ação |
|-------|-----|-----------|------|
| **Baixo** | 🟢 Verde | Situação normal, monitoramento de rotina | Monitorar |
| **Médio** | 🟡 Amarelo | Atenção necessária, monitoramento aumentado | Verificar |
| **Alto** | 🔴 Vermelho | Situação de alerta, ação imediata recomendada | Agir |
| **Crítico** | 🔴 Vermelho Escuro | Perigo iminente, ação urgente necessária | Intervir Urgente |

## Exemplos de Uso

### Criar um Dashboard
```bash
curl -X POST http://localhost:8000/api/dashboards/ \
  -H "Content-Type: application/json" \
  -d '{\n    "dashboard_id": "dash_001",\n    "name": "Monitoramento Entrada",\n    "risco": "baixo"\n  }'
```

### Adicionar Câmera a um Dashboard
```bash
curl -X POST http://localhost:8000/api/dashboards/dash_001/cameras/ \
  -H "Content-Type: application/json" \
  -d '{\n    "camera_id": "cam_001",\n    "camera_link": "rtsp://192.168.1.100:554/stream",\n    "camera_loc": "Entrada Principal",\n    "camera_status": "active"\n  }'
```

### Alterar Nível de Risco
```bash
curl -X PATCH http://localhost:8000/api/dashboards/dash_001/update-risk/ \
  -H "Content-Type: application/json" \
  -d '{"risco": "alto"}'
```

### Obter Stream de Câmera
```bash
# Acessar em um navegador ou VLC
http://localhost:8000/api/cameras/cam_001/stream/
```

## Troubleshooting

### Problema: Conexão com MySQL falha
- Verificar se o MySQL está rodando
- Validar credenciais em `.env`
- Confirmar se o banco de dados foi criado: `CREATE DATABASE chirp_db;`

### Problema: Câmeras não conectam
- Verificar se as URLs RTSP/HTTP estão corretas
- Confirmar acesso à câmera na rede
- Verificar firewall e permissões de acesso

### Problema: Detecção lenta
- Reduzir tamanho do input para YOLO
- Usar modelo mais leve (YOLOv8n em vez de v11s)
- Aumentar intervalo entre frames

### Problema: WebSocket desconecta
- Verificar logs do Channels
- Aumentar timeout de conexão
- Usar Daphne em produção em vez de Gunicorn

## Dependências Principais

Todas as dependências estão listadas em `requirements.txt`:
- `django==5.2.1` - Framework web
- `djangorestframework` - API REST
- `channels` - WebSockets
- `opencv-python` - Processamento de vídeo
- `ultralytics` - YOLO
- `torch` - PyTorch para IA
- `mysqlclient` - Driver MySQL
- `gunicorn` - Servidor WSGI
- `python-dotenv` - Variáveis de ambiente

## Licença

Este projeto está licenciado sob a Licença MIT - veja o arquivo [LICENSE](LICENSE) para detalhes.


## Suporte

Para dúvidas ou sugestões, contate: [Luís Henrique](luish.ianoski@gmail.com).

---

**Desenvolvido com <3 usando Django e YOLO**
