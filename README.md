# Achemed Local

Aplicacao Laravel 11 configurada para rodar localmente com Docker Compose.

O arquivo principal para uso local e o `docker-composes-local.yaml`.

## Requisitos

- Docker
- Docker Compose

## Primeiro uso

Se ainda nao existir um arquivo `.env`, crie a partir do exemplo:

```bash
cp .env.example .env
```

Garanta que o banco SQLite exista:

```bash
touch database/database.sqlite
```

Suba os containers:

```bash
docker compose -f docker-composes-local.yaml up -d --build
```

Gere a chave da aplicacao, se ainda nao tiver sido gerada:

```bash
docker compose -f docker-composes-local.yaml exec app php artisan key:generate
```

A aplicacao ficara disponivel em:

```text
http://localhost:8000
```

## Containers

Os servicos principais sao:

- `app`: servidor Laravel em `localhost:8000`
- `queue`: worker de filas com `php artisan queue:work`
- `scheduler`: agendador com `php artisan schedule:work`
- `vite`: servidor de assets, usado apenas com o profile `assets`

## Comandos principais

Subir os containers em segundo plano:

```bash
docker compose -f docker-composes-local.yaml up -d
```

Subir reconstruindo as imagens:

```bash
docker compose -f docker-composes-local.yaml up -d --build
```

Subir exibindo os logs no terminal:

```bash
docker compose -f docker-composes-local.yaml up
```

Parar os containers:

```bash
docker compose -f docker-composes-local.yaml stop
```

Parar e remover containers e rede:

```bash
docker compose -f docker-composes-local.yaml down
```

Ver status dos servicos:

```bash
docker compose -f docker-composes-local.yaml ps
```

Ver logs de todos os servicos:

```bash
docker compose -f docker-composes-local.yaml logs -f
```

Ver logs de um servico especifico:

```bash
docker compose -f docker-composes-local.yaml logs -f app
```

## Executar comandos dentro do container

Abrir um shell no container da aplicacao:

```bash
docker compose -f docker-composes-local.yaml exec app sh
```

Rodar comandos Artisan:

```bash
docker compose -f docker-composes-local.yaml exec app php artisan migrate
docker compose -f docker-composes-local.yaml exec app php artisan migrate:fresh --seed
docker compose -f docker-composes-local.yaml exec app php artisan cache:clear
docker compose -f docker-composes-local.yaml exec app php artisan route:list
```

Rodar o Tinker:

```bash
docker compose -f docker-composes-local.yaml exec app php artisan tinker
```

Instalar dependencias PHP:

```bash
docker compose -f docker-composes-local.yaml exec app composer install
```

Atualizar dependencias PHP:

```bash
docker compose -f docker-composes-local.yaml exec app composer update
```

Rodar testes:

```bash
docker compose -f docker-composes-local.yaml exec app php artisan test
```

## Assets com Vite

O servico `vite` usa o profile `assets`, entao ele so sobe quando solicitado.

Subir a aplicacao com Vite:

```bash
docker compose -f docker-composes-local.yaml --profile assets up -d
```

Ver logs do Vite:

```bash
docker compose -f docker-composes-local.yaml logs -f vite
```

Executar comandos NPM no container do Vite:

```bash
docker compose -f docker-composes-local.yaml --profile assets exec vite npm install
docker compose -f docker-composes-local.yaml --profile assets exec vite npm run build
```

## Filas e scheduler

Reiniciar o worker de filas:

```bash
docker compose -f docker-composes-local.yaml restart queue
```

Reiniciar o scheduler:

```bash
docker compose -f docker-composes-local.yaml restart scheduler
```

Ver logs da fila:

```bash
docker compose -f docker-composes-local.yaml logs -f queue
```

Ver logs do scheduler:

```bash
docker compose -f docker-composes-local.yaml logs -f scheduler
```

## Limpeza

Remover containers, rede e volumes anonimos:

```bash
docker compose -f docker-composes-local.yaml down -v
```

Remover containers e reconstruir tudo do zero:

```bash
docker compose -f docker-composes-local.yaml down -v
docker compose -f docker-composes-local.yaml up -d --build
```
