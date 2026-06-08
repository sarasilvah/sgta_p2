# SGTA — Sistema de Gerenciamento de Tarefas Acadêmicas

API REST desenvolvida em Django para gerenciar tarefas acadêmicas e os usuários responsáveis por elas. O sistema permite cadastrar tarefas com status, prioridade e prazo de entrega, além de filtrar e buscar tarefas de diversas formas.

---

## Tecnologias utilizadas

- Python 3.x
- Django 6.0.3
- Django REST Framework 3.17.0
- SQLite (banco de dados local para desenvolvimento)

---

## Estrutura do projeto

```
sgta/
├── backend/
│   ├── config/          # Configurações do projeto Django (settings, urls, wsgi)
│   ├── tarefas/         # App de tarefas (model, views, urls)
│   ├── usuarios/        # App de usuários (model, views, urls)
│   ├── manage.py
│   └── requirements.txt
└── docker-compose.yml
```

---

## Como executar localmente

**1. Acesse a pasta do backend:**
```bash
cd backend
```

**2. Crie e ative o ambiente virtual:**
```bash
python -m venv venv

# Windows
venv\Scripts\activate

# Linux/Mac
source venv/bin/activate
```

**3. Instale as dependências:**
```bash
pip install -r requirements.txt
```

**4. Aplique as migrations (cria o banco de dados):**
```bash
python manage.py migrate
```

**5. Inicie o servidor:**
```bash
python manage.py runserver
```

A API estará disponível em `http://127.0.0.1:8000/`.

---

## Modelos

### Usuário

| Campo         | Tipo       | Descrição                        |
|---------------|------------|----------------------------------|
| `id`          | inteiro    | Identificador único (automático) |
| `nome`        | texto      | Nome completo do usuário         |
| `email`       | e-mail     | E-mail único do usuário          |
| `ativo`       | booleano   | Se o usuário está ativo          |
| `data_criacao`| data/hora  | Preenchido automaticamente       |

### Tarefa

| Campo                  | Tipo      | Descrição                                                    |
|------------------------|-----------|--------------------------------------------------------------|
| `id`                   | inteiro   | Identificador único (automático)                             |
| `titulo`               | texto     | Título da tarefa                                             |
| `descricao`            | texto     | Descrição detalhada                                          |
| `status`               | escolha   | `ABERTA`, `EM_ANDAMENTO`, `CONCLUIDA` ou `CANCELADA`         |
| `prioridade`           | escolha   | `URGENTE` ou `NAO_URGENTE`                                   |
| `data_criacao`         | data/hora | Preenchida automaticamente                                   |
| `data_entrega`         | data      | Prazo de entrega da tarefa                                   |
| `usuario_responsavel`  | FK        | Usuário responsável (pode ser nulo)                          |

---

## Endpoints

### Tarefas

| Método | URL | Descrição |
|--------|-----|-----------|
| GET | `/tarefas/` | Lista todas as tarefas |
| GET | `/tarefas/<id>/` | Retorna uma tarefa pelo ID |
| GET | `/tarefas/status/<status>/` | Filtra tarefas por status |
| GET | `/tarefas/prioridade/<prioridade>/` | Filtra tarefas por prioridade |
| GET | `/tarefas/filtro/<status>/<prioridade>/` | Filtra por status e prioridade combinados |
| GET | `/tarefas/atrasadas/` | Lista tarefas com prazo vencido e não concluídas |
| GET | `/tarefas/busca/<termo>/` | Busca tarefas pelo título (parcial, sem distinção de maiúsculas) |

**Valores válidos para `status`:** `ABERTA`, `EM_ANDAMENTO`, `CONCLUIDA`, `CANCELADA`

**Valores válidos para `prioridade`:** `URGENTE`, `NAO_URGENTE`

**Exemplo de resposta — `/tarefas/`:**
```json
[
  {
    "id": 1,
    "titulo": "Prova de Algoritmos",
    "descricao": "Revisão dos conteúdos do semestre",
    "status": "ABERTA",
    "prioridade": "URGENTE",
    "data_criacao": "2026-03-23T14:08:26.752Z",
    "data_entrega": "2026-04-30",
    "usuario_responsavel": "Alvaro"
  }
]
```

**Exemplo de resposta — tarefa não encontrada:**
```json
{
  "erro": "Tarefa não encontrada."
}
```

---

### Usuários

| Método | URL | Descrição |
|--------|-----|-----------|
| GET | `/usuarios/` | Lista todos os usuários |
| GET | `/usuarios/<id>/` | Retorna um usuário pelo ID |

**Exemplo de resposta — `/usuarios/`:**
```json
[
  {
    "id": 1,
    "nome": "Alvaro",
    "email": "alvaro@email.com",
    "ativo": true,
    "data_criacao": "2026-03-20T10:00:00.000Z"
  }
]
```

---

## Observações

- O banco de dados padrão é **SQLite**, armazenado no arquivo `backend/db.sqlite3`. É ideal para desenvolvimento e testes locais.
- O campo `usuario_responsavel` nas respostas de tarefas exibe o **nome** do usuário, não o ID.
- Tarefas marcadas como `CONCLUIDA` não aparecem na listagem de tarefas atrasadas, mesmo que o prazo tenha vencido.

# PROVA BACKEND - SQLITE → POSTGRESQL + DOCKER

# =====================================================

# 1) BAIXAR O PROJETO

# =====================================================

# Clonar projeto do GitHub

git clone URL_DO_REPOSITORIO

# Entrar na pasta

cd nome_do_projeto

# =====================================================

# 2) CRIAR ARQUIVOS DO DOCKER

# =====================================================

# RAIZ DO PROJETO

AQUIII docker-compose.yml
services:

  db:
  
    image: postgres:16

    environment:
      POSTGRES_DB: ${DB_NAME}           
      POSTGRES_USER: ${DB_USER}        
      POSTGRES_PASSWORD: ${DB_PASSWORD} 

    volumes:
      - postgres_data:/var/lib/postgresql/data

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d ${DB_NAME}"]
      interval: 5s  
      timeout: 5s   
      retries: 5    


  web:

    build: ./backend

    volumes:
      - ./backend:/app

  
    # Acesso em: http://localhost:8000
    ports:
      - "8000:8000"

    env_file:
      - .env

  
    depends_on:
      db:
        condition: service_healthy

volumes:
  postgres_data:

AQUIII.env
SECRET_KEY=troque-por-uma-chave-segura
DEBUG=True

DB_NAME=DB_SGTA
DB_USER=DB_ADMIN
DB_PASSWORD= 1234
DB_HOST= db
DB_PORT=5432

.env.example

# DENTRO DE backend/

AQUIII
Dockerfile


FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1

ENV PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN sed -i 's/\r//' entrypoint.sh && chmod +x entrypoint.sh

EXPOSE 8000

ENTRYPOINT ["sh", "entrypoint.sh"]

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

AQUIII entrypoint.sh
#!/bin/sh

echo "Verificando o banco de dados..."

echo "Aplicando as migrações no banco de dados..."
python manage.py migrate

echo "Iniciando o servidor Python..."

exec "$@"

# =====================================================

# 3) CONFIGURAR O .env

# =====================================================

SECRET_KEY=troque-por-uma-chave-segura
DEBUG=True

DB_NAME=DB_SGTA
DB_USER=DB_ADMIN
DB_PASSWORD= 1234
DB_HOST= db
DB_PORT=5432


# =====================================================

# 4) ALTERAR O settings.py

# =====================================================

# Trocar SQLite por PostgreSQL

DATABASES = {
'default': {
'ENGINE': 'django.db.backends.postgresql',
'NAME': config('DB_NAME'),
'USER': config('DB_USER'),
'PASSWORD': config('DB_PASSWORD'),
'HOST': config('DB_HOST'),
'PORT': config('DB_PORT'),
}
}

# =====================================================

# 5) ATUALIZAR requirements.txt

# =====================================================

# Adicionar:

psycopg
psycopg-binary
python-decouple

# =====================================================

# 6) INICIAR O DOCKER

# =====================================================

# Abrir Docker Desktop

# Na raiz do projeto:

docker compose up --build

# O comando:

# - cria a imagem

# - cria o PostgreSQL

# - cria o Django

# - instala dependências

# - aplica migrations

# - inicia o servidor

# =====================================================

# 7) VERIFICAR CONTAINERS

# =====================================================

docker compose ps

# Deve aparecer:

web
db

# =====================================================

# 8) CRIAR SUPERUSUÁRIO

# =====================================================

docker compose exec web python manage.py createsuperuser

# =====================================================

# 9) TESTAR O SISTEMA

# =====================================================

http://localhost:8000/admin

# Fazer login

# =====================================================

# 10) COMPARTILHAR NO DOCKER HUB

# =====================================================

# Login

docker login

# Criar tag

docker tag nome-imagem:latest usuario/nome-imagem:v1

# Enviar

docker push usuario/nome-imagem:v1

# =====================================================

# 11) ENVIAR ALTERAÇÕES PARA O GITHUB

# =====================================================

# Verificar alterações

git status

# Adicionar arquivos

git add .

# Criar commit

git commit -m "Migracao SQLite para PostgreSQL com Docker"

# Ver commits

git log --oneline

# Enviar para GitHub

git push origin main

# ou

git push origin master

# =====================================================

# 12) SE DER ERRO 403 NO PUSH

# =====================================================

# Exemplo:

Permission denied
403

# Significa:

# O Git está funcionando

# O repositório existe

# O commit foi criado

# O usuário autenticado não possui permissão

# =====================================================

# 13) VERIFICAR REPOSITÓRIO REMOTO

# =====================================================

git remote -v

# Exemplo:

origin https://github.com/usuario/projeto.git

# =====================================================

# 14) ENVIAR PARA OUTRO REPOSITÓRIO

# =====================================================

# Ver remotes atuais

git remote -v

# Remover o origin atual

git remote remove origin

# Adicionar novo repositório

git remote add origin https://github.com/SEU_USUARIO/NOVO_REPOSITORIO.git

# Conferir

git remote -v

# Enviar

git push -u origin main

# =====================================================

# 15) ALTERNATIVA MAIS SEGURA

# =====================================================

# Manter o origin original

# e adicionar outro remote

git remote add meu-repo https://github.com/SEU_USUARIO/NOVO_REPOSITORIO.git

# Enviar para o novo repositório

git push meu-repo main

# =====================================================

# RESUMO DE PROVA

# =====================================================

# 1. git clone

# 2. criar Dockerfile

# 3. criar docker-compose.yml

# 4. criar .env

# 5. configurar PostgreSQL

# 6. docker compose up --build

# 7. createsuperuser

# 8. testar localhost:8000/admin

# 9. git add .

# 10. git commit -m "..."

# 11. git push

# 12. docker push
